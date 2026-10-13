---
layout: post
title: "CAP-Theorem in a Nutshell"
date: 2026-09-27
categories: [backend_engineering]
---

# CAP-Theorem in a Nutshell

Das **CAP-Theorem** beschreibt einen zentralen Trade-off in verteilten Systemen: Sobald eine **Netzwerkpartition** auftritt, kann ein System nicht gleichzeitig **starke Konsistenz** und **hohe Verfügbarkeit** garantieren.

Genau diese Entscheidung prägt viele moderne Datenbanken, Message-Systeme und verteilte Plattformen.

## Was bedeutet CAP?

| Buchstabe | Begriff             | Bedeutung                                                                                                  |
|-----------|---------------------|------------------------------------------------------------------------------------------------------------|
| C         | Consistency         | Jeder Read liefert den neuesten erfolgreichen Write oder einen Fehler.                                     |
| A         | Availability        | Jede Anfrage erhält eine Antwort, auch wenn diese Antwort möglicherweise nicht den neuesten Stand enthält. |
| P         | Partition Tolerance | Das System arbeitet trotz Netzwerkausfällen zwischen Knoten weiter.                                        |

Der wichtigste Punkt ist: In echten verteilten Systemen ist **P keine Option, sondern eine Randbedingung**. Netzwerkprobleme lassen sich nicht vollständig verhindern. Deshalb lautet die praktische Frage nicht *"CA, CP oder AP?"*, sondern eher: **Wie verhält sich das System während einer Partition - eher konsistent oder eher verfügbar?**

## Ein einfaches Beispiel

Stell dir zwei Datenbankknoten vor, **Node A** und **Node B**. Zu Beginn halten beide denselben Wert:

````text
        Network
    ┌───────────────┐
    │               │
┌───▼───┐       ┌───▼───┐
│ Node A│       │ Node B│
│       │       │       │
│ x = 10│       │ x = 10│
└───────┘       └───────┘
````

- **T0:** Beide Knoten speichern `x = 10`.
- **T1:** Eine **Netzwerkpartition** trennt Node A und Node B.
- **T2:** Ein Client schreibt auf **Node A** den neuen Wert `x = 11`.
- **T3:** Ein anderer Client liest `x` über **Node B**, das weiterhin nur `x = 10` kennt.

**Konsequenz:** Das System muss sich nun zwischen **Verfügbarkeit** und **Konsistenz** entscheiden.

## Verhalten in einem AP-orientierten System

Ein **AP-System** bevorzugt in dieser Situation die **Verfügbarkeit**. Node B beantwortet die Anfrage also weiterhin - auch wenn der Wert veraltet sein kann.

Das bedeutet:

- Node B antwortet mit `x = 10`
- die Antwort ist verfügbar
- die Antwort ist möglicherweise **nicht konsistent** mit dem neuesten Write

Dieses Verhalten ist typisch für Systeme, die mit **eventual consistency** arbeiten.

## Verhalten in einem CP-orientierten System

Ein **CP-System** bevorzugt in derselben Situation die **Konsistenz**. Node B darf den alten Wert nicht einfach zurückgeben, weil er nicht sicher weiß, ob inzwischen ein neuer Write existiert.

Das bedeutet:

- Node B liefert **keinen veralteten Wert**
- stattdessen gibt es einen Fehler, ein Timeout oder die Anfrage wird abgelehnt
- die Daten bleiben konsistent, aber die **Verfügbarkeit leidet**

## Was CAP nicht bedeutet

Das CAP-Theorem wird oft missverstanden. Es sagt **nicht**, dass man sich dauerhaft nur zwei von drei Eigenschaften aussuchen kann. Es sagt vielmehr:

> **Wenn eine Partition auftritt, musst du zwischen Konsistenz und Verfügbarkeit entscheiden.**

* Ohne Partition: Node A und Node B können miteinander sprechen. Ein neuer Write kann repliziert werden, und Clients bekommen in der Regel eine korrekte Antwort. Das System wirkt also konsistent und verfügbar zugleich.
* Mit Partition: Die Knoten sind voneinander getrennt. Dann muss das System entscheiden:
  * entweder verfügbar bleiben und möglicherweise veraltete Daten liefern
  * oder konsistent bleiben und dafür Anfragen ablehnen oder blockieren