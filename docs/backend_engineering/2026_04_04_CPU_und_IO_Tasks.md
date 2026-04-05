---
layout: post
title: "CPU und IO Tasks"
date: 2026-04-04
categories: [backend_engineering]
---

# CPU und IO Tasks

Die meisten echten Systeme sind CPU und IO Tasks gebunden, z.B. Request -> DB Call (IO) danach JSON serialisieren (CPU).

Man kann grob diese beide Tasks unterscheiden.
* Die Laufzeit wird durch Rechenleistung begrenzt, d.h. CPU-bound = Rechnen limitiert Performance, z.B. Kryptographie, Bild-/Videoverarbeitung ...
* Die Laufzeit wird durch Warten auf externe Ressourcen bestimmt, d.h. IO-bound = Warten limitiert Performance, z.B. Datenbankabfragen, HTTP-Calls, File I/O ...

Um den Typ der Aufgabe zu bestimmen, beantworte diese Frage:
> Verbringt der Thread mehr Zeit mit Rechnen oder Warten?

