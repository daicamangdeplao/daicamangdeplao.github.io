---
layout: post
title: "Wenn die Downstream-Dependency nicht stabil ist: Adaptive Backpressure im Queue-Worker"
date: 2026-10-13
categories: [backend_engineering]
---

# Wenn die Downstream-Dependency nicht stabil ist

Wenn eine Downstream-Dependency ausfällt oder überlastet ist, führt die Gewohnheit, den Queue-Worker weiterhin mit normaler Geschwindigkeit Nachrichten pollen zu lassen, häufig zu einer nutzlosen Retry-Schleife und schiebt Nachrichten nach Erschöpfung der Versuche in die Dead Letter Queue (DLQ).

Statt den Worker manuell abzuschalten oder den Verarbeitungsfluss hart zu unterbrechen, lässt Canva den Worker seine Concurrency selbst reduzieren, wenn die Fehler zunehmen, und sie wieder erhöhen, wenn sich die Dependency erholt. In einem Artikel hat der Autor Mikalai Barysau von Canva Engineering geteilt, wie er diesen Mechanismus direkt in die Shared Queue Library integriert haben, um die Last lokal auf jeder Instanz automatisch zu regulieren.

## Inhaltsverzeichnis
- [1. Motivation](#1-motivation)
- [2. Lösung – Entwurf des Steuerungsflusses in der Shared Queue Library](#2-lösung--entwurf-des-steuerungsflusses-in-der-shared-queue-library)
- [3. Fazit](#3-fazit)

## 1. Motivation

Ein gewöhnlicher Queue-Worker arbeitet nach dem Prinzip eines **Greedy Workers**: Sobald Ressourcen frei sind, nimmt er mehr Arbeit an. Wenn eine Dependency schwächelt, erschöpft dieses Verhalten schnell die Retry-Versuche und führt dazu, dass Nachrichten in die DLQ gelangen. Die gängigen Techniken stoßen dabei auf bestimmte Hürden:

- **Fixed Rate Limiting** — Es ist schwierig, einen passenden Schwellenwert zu wählen, wenn die Kapazität der Dependency ständig schwankt; ein fester Schwellenwert kann im Normalbetrieb zu streng oder im Störungsfall zu locker sein.
- **Circuit Breaker im Öffnen/Schließen-Stil** — Üblicherweise wird der Traffic unterbrochen, wenn der Breaker öffnet, und nach einer Cooldown-Phase wieder zugelassen. Das Fehlen eines Übergangspuffers zwischen maximalem Durchsatz und vollständigem Stopp erzeugt leicht einen neuen Lastansturm, sobald der Breaker Aufrufe wieder zulässt.
- **Exponential Retry Backoff** — Dehnt lediglich die Verarbeitungszeit für jede einzelne Nachricht, kontrolliert aber nicht den gesamten Traffic, den die Worker-Fleet an das Downstream-System sendet.
- **Client-side Adaptive Throttling** — Ein Team bei Canva hatte eine solche Bibliothek auf Basis des Google SRE Book eingesetzt, doch die Lösung lag außerhalb der Shared Queue Library und war eng an einen festen Algorithmus gebunden.

## 2. Lösung – Entwurf des Steuerungsflusses in der Shared Queue Library

Anders als beim klassischen Backpressure, der das Signal zurück an den Producer übermittelt, lässt diese Lösung den Worker seine Annahmegeschwindigkeit selbst drosseln. Indem der Worker ablehnt, weitere Nachrichten anzunehmen, wird die Last in der Queue gehalten. Die gesamte Entscheidungslogik läuft vollständig lokal auf jeder Instanz, ohne externen Coordinator und ohne zusätzliche Network Calls für den Steuerungsmechanismus.

Der Ablauf besteht aus drei Komponenten:

- **Signals** — Nach jeder verarbeiteten Nachricht hält der Worker das Ergebnis als Erfolg oder Fehlschlag fest.
- **Backpressure Controller** — Ein pluggable Interface, das einen Backoff-Factor von `0.0` (maximale Geschwindigkeit) bis `1.0` (vollständiger Rückzug) pflegt. Der Controller vergleicht die tatsächliche Failure Rate mit dem **Set Point** – der akzeptablen Basis-Fehlerrate, die lokal auf jeder Instanz konfiguriert wird. Solange die Failure Rate unter dem Set Point liegt, greift der Controller nicht ein. Überschreitet die Failure Rate den Set Point, erhöht der Controller schrittweise den Backoff-Factor, um die Concurrency zu drosseln; erholt sich die Dependency und fällt die Failure Rate wieder unter den Set Point, senkt der Controller den Backoff-Factor schrittweise, um die Concurrency automatisch wieder zu erhöhen.
- **Permits** — Vor jedem Pollen einer Nachricht aus der Queue fragt der Worker den Controller nach der Anzahl der Nachrichten, die er gleichzeitig abrufen darf. Der Controller reduziert diese Anzahl der Permits entsprechend dem aktuellen Backoff-Factor.

Das Zusammenspiel lässt sich als Regelkreis skizzieren:

````text
   ┌─────────────┐   Signals (Erfolg/Fehlschlag)    ┌────────────────────────┐
   │   Worker    │ ───────────────────────────────▶ │ Backpressure Controller│
   │             │                                  │  Failure Rate vs.      │
   │  poll(n)    │ ◀─────────────────────────────── │  Set Point → Backoff   │
   └──────┬──────┘        Permits (n reduziert)     └────────────────────────┘
          │
          ▼
   ┌─────────────┐
   │    Queue    │  Last bleibt hier, statt in die DLQ zu laufen
   └─────────────┘
````

## 3. Fazit

Durch die Integration des adaptiven Backpressure direkt in die Shared Queue Library muss nicht länger jedes Team seine eigene Throttling-Lösung bauen. Der Worker reagiert kontinuierlich und lokal auf den tatsächlichen Zustand der Downstream-Dependency: Er drosselt sich sanft bei steigenden Fehlerraten und beschleunigt wieder, sobald sich die Dependency erholt. Das Ergebnis ist ein gleichmäßigerer Übergang zwischen vollem Durchsatz und Stopp, weniger unnötige Retries und deutlich weniger Nachrichten, die in der DLQ landen.

---
**Reference:** Barysau, Mikalai: [Worker Backpressure (Part 1): How we taught our queue workers to slow down](https://www.canva.dev/blog/engineering/worker-backpressure-part-1-how-we-taught-our-queue-workers-to-slow-down/)
