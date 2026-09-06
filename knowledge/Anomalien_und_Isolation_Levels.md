---
layout: post
title: "DB-Anomalien und Isolation Levels — was die Namen verschweigen"
date: 2026-07-25
categories: [backend_engineering]
---

# DB-Anomalien und Isolation Levels — was die Namen verschweigen

Ein Production-Bug bei einer E-Wallet: Am Tagesende fehlen ein paar hunderttausend im Saldo. Der Code sieht harmlos aus — `SELECT` Saldo, prüfen ob genug Geld da ist, `UPDATE` und abziehen. Lokal reproduzierbar? Nein. Exceptions im Log? Keine. In Produktion aber laufen zwei Abhebungen fast gleichzeitig los, lesen beide `500k`, sehen beide „genug Geld", und lassen beide durch. Klassisches **Lost Update**.

## 1. Klassische Anomalien (ANSI SQL 1992)

Wenn mehrere Transaktionen parallel dieselben Daten anfassen, entstehen Anomalien. Der ANSI-SQL-Standard von 1992 nennt drei:

- **Dirty Read** — Transaktion A ändert eine Zeile, committet noch nicht. B liest den uncommitted Wert. A rollt zurück. B hat auf Daten reagiert, die nie wirklich existierten.
- **Non-repeatable Read** — Innerhalb *einer* Transaktion liest B eine Zeile zweimal und bekommt zwei verschiedene Werte, weil dazwischen jemand committet hat.
- **Phantom Read** — Wie Non-repeatable Read, aber auf Ergebnismengen: dieselbe Query liefert beim zweiten Aufruf eine neue Zeile, weil dazwischen ein passendes `INSERT` committet wurde.

## 2. Die vier Isolation Levels

Isolation Level ist im Kern ein Schalter: welche Anomalien akzeptierst du, um Durchsatz zu gewinnen?

| Isolation Level   | Dirty Read | Non-repeatable Read | Phantom Read | Lost Update | Write Skew |
|-------------------|------------|---------------------|--------------|-------------|------------|
| Read Uncommitted  | möglich    | möglich             | möglich      | möglich     | möglich    |
| Read Committed    | verhindert | möglich             | möglich      | möglich     | möglich    |
| Repeatable Read   | verhindert | verhindert          | möglich      | verhindert  | möglich    |
| Serializable      | verhindert | verhindert          | verhindert   | verhindert  | verhindert |

Je höher, desto sicherer — aber auch teurer, weil die Datenbank mehr sperren oder mehr prüfen muss.

## 3. Was der Standard verschweigt

1995 zeigte das Paper *„A Critique of ANSI SQL Isolation Levels"*, dass die klassische Tabelle Lücken hat. Zwei weitere Anomalien sind praktisch besonders relevant:

- **Lost Update** — Genau der E-Wallet-Bug. Zwei Transaktionen lesen denselben Saldo, rechnen darauf, schreiben ihn zurück — das spätere Schreiben überschreibt das frühere. Um Lost Update zu verhindern, braucht man laut Paper mindestens Repeatable Read.
- **Write Skew** — Subtiler. Beispiel: In einer Klinik muss immer mindestens ein Arzt Bereitschaft haben. Anna und Ben sind eingeteilt. Anna beantragt frei, das System liest „2 sind da" — okay. Zeitgleich beantragt Ben frei, seine Transaktion liest ebenfalls „2 sind da" — okay. Beide schreiben in *verschiedene* Zeilen, keine Kollision, beide committen. Ergebnis: niemand mehr im Dienst. Weil beide Transaktionen unterschiedliche Zeilen schreiben, gibt es keinen direkten Konflikt, den die DB erkennen könnte. Write Skew rutscht sogar durch Snapshot Isolation. Um es zu verhindern, muss man theoretisch auf Serializable.

## 4. PostgreSQL im Detail

Klingt einfach: Repeatable Read für Lost Update, Serializable für Write Skew. Aber jede Datenbank implementiert diese Namen anders. PostgreSQL als konkretes Beispiel:

- **Read Uncommitted** existiert nicht wirklich — der Name ist erlaubt, verhält sich aber wie Read Committed. Dirty Reads sind in PostgreSQL prinzipiell unmöglich.
- **Read Committed** (Default) — Snapshot **pro Statement**. Zwei `SELECT`s in derselben Transaktion können unterschiedliche Daten sehen.
- **Repeatable Read** — Snapshot Isolation (SI), **pro Transaktion**. Ein einziger Snapshot vom ersten Statement bis Commit. Blockt zusätzlich zum Standard auch **Phantom Read** und **Lost Update** (die zweite Transaktion wird mit `serialization failure` abgebrochen — kein stilles Überschreiben). **Write Skew** bleibt aber möglich.
- **Serializable** — Umgesetzt via **Serializable Snapshot Isolation (SSI)**. Die DB verfolgt Lese-Schreib-Beziehungen zwischen Transaktionen und bricht eine ab, wenn das Ergebnis nicht äquivalent zu einer seriellen Ausführung wäre. Fängt damit auch Write Skew.

Zum Vergleich: Oracles „Serializable" ist unter der Haube Snapshot Isolation — Write Skew läuft dort durch. **Gleicher Name, unterschiedliches Verhalten.**

### Übersicht: Isolation Level × Anomalie × PostgreSQL

Die folgende Tabelle stellt gegenüber, was der ANSI-Standard (1992 + Erweiterung 1995) *garantiert* — und was PostgreSQL *tatsächlich* liefert. „möglich" heißt: die Anomalie kann auftreten. „verhindert" heißt: die Anomalie kann in diesem Level nicht auftreten.

| Isolation Level                        | Dirty Read | Non-repeatable Read | Phantom Read   | Lost Update                                    | Write Skew     |
|----------------------------------------|------------|---------------------|----------------|------------------------------------------------|----------------|
| Read Uncommitted *(= Read Committed)*  | verhindert | möglich             | möglich        | möglich                                        | möglich        |
| Read Committed *(Default)*             | verhindert | möglich             | möglich        | möglich                                        | möglich        |
| Repeatable Read *(Snapshot Isolation)* | verhindert | verhindert          | **verhindert** | **verhindert** *(via `serialization failure`)* | möglich        |
| Serializable *(SSI)*                   | verhindert | verhindert          | verhindert     | verhindert                                     | **verhindert** |

Zwei Beobachtungen fallen auf:

- PostgreSQL **übererfüllt** den Standard bei Repeatable Read: Phantom Read und Lost Update sind dort schon abgedeckt, obwohl der Standard sie erst bei Serializable verlangt.
- PostgreSQL **erfüllt** Serializable strenger als andere DBs: nur SSI fängt Write Skew — Oracles gleichnamiger Level tut das nicht.

## 5. Spec vs. Implementation

Isolation Level ist eine **Spezifikation**: welche Anomalie soll verhindert werden. Die Spec sagt nicht, *wie*. PostgreSQL setzt sie mit **MVCC** (Multi-Version Concurrency Control) plus Locks um: bei einem `UPDATE` wird die alte Version behalten, eine neue angelegt. Ein Snapshot ist nur die Auswahl, welche Version diese Transaktion sehen darf. Reader blockieren Writer nicht und umgekehrt — daher gewinnt MVCC im Alltag. Erst wenn man mit `SELECT ... FOR UPDATE` Ordnung erzwingt, wird tatsächlich gesperrt.

**Konsequenz:** Nie einem Isolation-Level-Namen vertrauen, weil er sicher klingt. Immer fragen: *Was macht diese Datenbank konkret unter der Haube?*

## 6. Praxis — was man wirklich tut

- **Isolation Level ist nicht global.** Für eine einzelne Transaktion setzt man den Level lokal (`BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE`), ohne den Rest der Anwendung anzufassen. Das Default der DB zu erhöhen hat einen viel größeren Blast Radius.
- **Lost Update** löst man meist besser auf Statement-Ebene, ohne Isolation Level zu ändern:
  - **Conditional Write** — atomar in einem Statement: `UPDATE accounts SET balance = balance - 70 WHERE id = ? AND balance >= 70`, dann `affected rows` prüfen. Die DB sperrt die richtige Zeile automatisch.
  - **Optimistic Locking** — Version-Spalte, `UPDATE ... WHERE id = ? AND version = ?`. Bei `affected rows = 0` retry. JPA `@Version` ist genau das. Geeignet bei wenig Contention.
  - **Pessimistic Locking** — `SELECT ... FOR UPDATE`, die Zeile wird gesperrt. Geeignet für Hot Rows wie Flash-Sale-Bestände, wo Optimistic sich unter Last totretrien würde.

  Die Wahl hängt von der Contention ab, nicht davon, was „edler" klingt.

- **Write Skew** — Bevor man auf Serializable geht (das unter Last zu vielen Aborts und Retry-Schleifen führt), macht man die versteckte Invariante sichtbar: eine repräsentative Zeile per `SELECT ... FOR UPDATE` sperren, sodass sich die zwei Transaktionen doch berühren müssen. Aus einem unsichtbaren Konflikt wird ein sichtbarer, den die DB sperren kann.

## 7. Fazit

Isolation Level ist kein Config-Wert, den man einmal setzt und vergisst. Es ist ein **Vertrag** zwischen dir und der Datenbank: was verbirgt sie für dich, und was musst du selbst absichern? Jede Absenkung des Levels holt Verantwortung zurück in die Anwendung — und die DB erinnert dich nie. Sie liefert einfach still falsche Ergebnisse, meist am umsatzstärksten Tag des Jahres.

Die richtige Frage ist nicht **„welches Isolation Level?"**, sondern:

> Welche Invariante muss in diesem Geschäftsvorgang immer gelten, und auf welchen Daten beruht sie, die ich zwar lese, aber nicht schreibe?

Ist diese Frage beantwortet, folgt Read Committed vs. Serializable, Optimistic vs. Pessimistic fast von selbst.

Die Werkzeuge ändern sich — heute InnoDB, morgen ein Distributed SQL, übermorgen etwas anderes. Das Grundproblem — mehrere Beobachter schauen auf dieselbe, sich ändernde Wahrheit, und jeder glaubt, das Richtige zu sehen — ist älter als Computer und wird uns alle überleben.
