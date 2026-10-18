# Template für wissensorientierte Dokumentation

Dieser Typ beantwortet die Frage:

> "Wie funktioniert etwas?"

---

## Struktur

### Titel

Thema klar benennen

**Beispiel:**
- "Controller vs. RestController in Spring"
- "Wie funktioniert OAuth in SAP Commerce?"

---

### 1. Motivation
💡
Warum sollte man das kennen?

### 2. Erklärung

Begriff oder Konzept vorstellen.

### 3. Beispiel

Am besten mit einem realen Szenario.

### 4. Praktische Anwendung

Praktische Empfehlungen, Anwendungsfälle oder Best Practices.

**Mögliche Varianten:**
- "Wann verwenden?" (bei Technologieauswahl)
- "Wie schützt man sich?" (bei Security-Themen)
- "Best Practices" (bei Implementierungen)
- "Anwendungsfälle" (bei Konzepten)

### 5. Zusammenfassung

Die wichtigsten Punkte in 2 bis 3 Sätzen.

---

## Beispiel

### Controller vs. RestController in Spring

**Warum ist das wichtig?**
Beide Annotationen werden häufig verwendet, erfüllen aber unterschiedliche Aufgaben.

**Controller**
Wird hauptsächlich für MVC-Anwendungen genutzt und liefert Views zurück.

**RestController**
Liefert direkt JSON oder XML zurück und wird typischerweise für APIs verwendet.

**Praktische Anwendung**
Für Weboberflächen → Controller
Für REST APIs → RestController

**Fazit**
Die Wahl hängt davon ab, ob eine View oder eine API bereitgestellt werden soll.
