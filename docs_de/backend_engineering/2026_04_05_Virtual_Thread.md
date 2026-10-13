---
layout: post
title: "Virtual Thread"
date: 2026-04-05
categories: [backend_engineering]
---

# Virtual Thread in Java 21+

> Write synchronous code, get asynchronous scalability.

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

## Vorteile

Angenommen, es gibt die einfache Logik

```java
User user = fetchUser();
Order order = fetchOrder();
```

laufen die Aufrufe **blockierend** und **nacheinander**

1. `fetchUser()`
2. warten bis fertig
3. `fetchOrder()`
4. warten bis fertig

Bei der Verwendung von Virtual Threads kann man trotzdem parallel arbeiten

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {

    Future<User> userFuture = executor.submit(this::fetchUser);
    Future<Order> orderFuture = executor.submit(this::fetchOrder);

    return combine(
        userFuture.get(),
        orderFuture.get()
    );
}
```

Durch ersetzt praktisch

```java
CompletableFuture<User> userFuture = fetchUserAsync();
CompletableFuture<Order> orderFuture = fetchOrderAsync();

CompletableFuture<Result> result =
        userFuture.thenCombine(
                orderFuture,
                (user, order) -> combine(user, order)
        );
```

## Fazit

Virtual Threads sind gut geeignet für
* Web-Server (Thread-per-Request)
* Datenbank Zugriffe
* Microservices mit vielen I/O Requests

Virtual Threads bringen keine Vorteile im Szenario
* CPU-bound Tasks
* Hochoptimierte Low-Level Parallelisierung
