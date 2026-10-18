# Template für lösungsorientierte Dokumentation

Dieser Typ beantwortet die Frage:

> "Ich habe ein Problem. Wie löse ich es?"

---

## Struktur

### Titel

Konkretes Problem nennen

**Beispiel:**
- "Solr-Indexierung schlägt in SAP Commerce fehl"
- "Docker Container startet nach Deployment nicht"

---

### 1. Problem

Was ist passiert?
Welche Symptome gab es?

### 2. Analyse

Wie bist du vorgegangen?
Welche Hypothesen hattest du?

### 3. Ursache

Was war der eigentliche Grund?

### 4. Lösung

Was wurde geändert?

### 5. Lessons Learned

Was würdest du beim nächsten Mal anders machen?

---

## Beispiel

### SAP Commerce: Solr-Indexierung schlägt fehl

**Problem**
Nach dem Deployment konnte kein Produktindex erstellt werden.

**Analyse**
Die Logs zeigten mehrere Fehlermeldungen während des Full Index.

**Ursache**
Eine fehlerhafte Solr-Konfiguration wurde aus einer älteren Umgebung übernommen.

**Lösung**
Die Konfiguration wurde angepasst und der Index neu aufgebaut.

**Lessons Learned**
- Solr-Konfigurationen vor dem Deployment prüfen.
- Logs immer zuerst analysieren.
