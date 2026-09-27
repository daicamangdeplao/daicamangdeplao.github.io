---
layout: post
title: "Virtual Thread"
date: 2026-04-05
categories: [backend_engineering]
---

# Virtual Thread

Klassische Java-Threads (sogenannte Platform Threads) sind direkt an OS-Threads gebunden. Das führt zu: hohem Speicherverbrauch, teurem Kontextwechsel, limitierter Skalierung.

Virtual Threads sind eine Alternative, um diese Probleme zu lösen. Millionen leichtgewichtiger Threads, verwaltet durch die JVM statt vom OS.

## Wie funktioniert das?

Wenn ein Virtual Thread blockiert ist, unmountet die JVM ihn vom Carrier Thread, und der Carrier Thread wird für andere Tasks frei.

```java
public class VirtualThreadExecutorExample {
    public static void main(String[] args) throws Exception {
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < 100_000; i++) {
                int taskId = i;
                executor.submit(() -> {
                    Thread.sleep(1000);
                    System.out.println("Task " + taskId);
                    return null;
                });
            }
        }
    }
}
```

## Wann einsetzen?

Virtual Threads sind gut geeignet für
* Web-Server (Thread-per-Request)
* Datenbank Zugriffe
* Microservices mit vielen I/O Requests

Virtual Threads bringen keine Vorteile im Szenario
* CPU-bound Tasks
* Hochoptimierte Low-Level Parallelisierung
