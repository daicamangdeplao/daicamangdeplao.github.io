---
layout: post
title: "Debugging-Strategie"
date: 2026-09-06
categories: [mental_engineering]
---

# Debugging-Strategie

Debugging ist keine Frage von Intuition oder Glück, sondern ein systematisches Vorgehen: vom beobachtbaren Symptom über eine überprüfbare Hypothese bis hin zur verifizierten Lösung. Wer diese Reihenfolge einhält, behebt die Ursache statt nur die Erscheinung.

## 1. Mit dem Symptom beginnen

Am Anfang steht immer die Beobachtung — nicht die Vermutung. Das Symptom wird so präzise wie möglich beschrieben:

- **Was ist es?** — Wie äußert sich der Fehler konkret? Falsches Ergebnis, Absturz, Timeout, fehlende Daten? Fehlermeldungen, Logs und Stacktraces gehören wörtlich festgehalten.
- **Wann tritt es auf?** — Erst seit einem bestimmten Release? Nur unter Last? Nur zu bestimmten Zeitpunkten? Reproduzierbar oder sporadisch?
- **Unter welchen Bedingungen tritt es auf?** — Welche Umgebung, welche Eingabedaten, welcher Benutzer, welche Konfiguration? Genauso wichtig: Unter welchen Bedingungen tritt es **nicht** auf?

Die Antworten grenzen den Suchraum ein. Das Gegenbeispiel ist dabei oft aufschlussreicher als das Beispiel.

## 2. Eine Hypothese formulieren

Aus dem beobachteten Muster entsteht eine konkrete, widerlegbare Annahme über die Ursache. Eine gute Hypothese benennt einen bestimmten Mechanismus an einer bestimmten Stelle — nicht „irgendwo in der Datenbankschicht", sondern „die Transaktion wird vor dem Commit des Nebenläufers gelesen".

Sind mehrere Hypothesen plausibel, werden sie danach priorisiert, wie schnell und wie eindeutig sie sich prüfen lassen.

## 3. Die Hypothese durch Tests überprüfen — nachstellen

Die Annahme wird nicht durch Nachdenken bestätigt, sondern durch das Nachstellen des Fehlers:

- **Reproduzieren** — Den Fehler zuverlässig und möglichst minimal auslösbar machen. Ein Fehler, der sich nicht reproduzieren lässt, lässt sich auch nicht als behoben nachweisen.
- **Automatisieren** — Die Reproduktion in einen fehlschlagenden Test überführen. Dieser Test dokumentiert das Problem und schützt später vor Regressionen.
- **Isolieren** — Variablen einzeln verändern und den Suchraum halbieren (Bisektion über Code, Daten, Konfiguration oder Commit-Historie), bis nur noch die Ursache übrig bleibt.

Bestätigt der Test die Hypothese nicht, wird sie verworfen — nicht angepasst, bis sie irgendwie passt. Der Weg führt zurück zu Schritt 2.

## 4. Die Lösung einsetzen

Erst wenn die Ursache belegt ist, wird korrigiert:

- Die **Ursache** beheben, nicht das Symptom kaschieren.
- Der zuvor fehlschlagende Test muss nun erfolgreich durchlaufen, alle übrigen Tests weiterhin ebenfalls.
- Prüfen, ob dieselbe Ursache **an anderen Stellen** dieselbe Klasse von Fehlern erzeugt.
- Ergebnis und Erkenntnis festhalten: Was war die Ursache, warum ist sie entstanden, wie wird sie künftig verhindert?

---

Der Kern der Strategie liegt in der Disziplin der Reihenfolge:

* **Beobachten vor Vermuten** — das Symptom bestimmt die Richtung, nicht der erste Verdacht;
* **Nachstellen vor Reparieren** — was nicht reproduzierbar ist, ist nicht verifizierbar;
* **Ursache vor Symptom** — eine unterdrückte Erscheinung kehrt an anderer Stelle zurück.
