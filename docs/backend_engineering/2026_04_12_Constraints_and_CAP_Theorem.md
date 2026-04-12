---
layout: post
title: "Constraints and CAP Theorem"
date: 2026-04-12
categories: [backend_engineering]
---

# Constraints and CAP Theorem

Constraints (Beschränkungen) sind die Rahmenbedingungen, innerhalb derer dein System funktionieren muss, z.B. Budget, Latenzanforderung (z.B. 200 ms), Datenkonsistenz, usw. Die Constraints werden unterteilt in Kategorien: (1) Skalierung & Last, (2) Latenz & Performance, (3) Konsistenz & Verfügbarkeit, (4) Zuverlässigkeit & Fehlertoleranz. Der Constraint ist allgemein und gegeben. Es hängt von Business-Anforderungen ab. 

CAP-Theorem ist eine technische, unveränderliche Systemgrenze und gilt immer für verteilte Systeme.

In verteilten Systemen kannst du nicht alle drei Eigenschaften gleichzeitig garantieren — du kannst immer nur **zwei der drei** erfüllen:

* **Consistency (Konsistenz)** — Jede Leseanfrage erhält die aktuellste Version der Daten oder einen Fehler.
* **Availability (Verfügbarkeit)** — Jede Anfrage erhält eine Antwort (kein Fehler), aber nicht zwingend die aktuellsten Daten.
* **Partition Tolerance (Partitionstoleranz)** — Das System funktioniert weiter, auch wenn Nachrichten zwischen Knoten verloren gehen oder verzögert werden.

In der Praxis ist **Partition Tolerance keine echte Wahl** — Netzwerkausfälle passieren immer in verteilten Systemen. Daher lautet der eigentliche Trade-off: **CP vs. AP** — entweder opferst du Konsistenz (AP) oder Verfügbarkeit (CP), wenn eine Netzwerkpartition auftritt.

