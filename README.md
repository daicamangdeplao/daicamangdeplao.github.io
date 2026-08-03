# daicamangdeplao

https://docs.github.com/en/pages/quickstart

## Projektaufbau

Dieses Projekt besteht aus zwei großen Teilen:

* **Public Posts** – Beiträge, die öffentlich angezeigt werden.
* **Private Posts** – Beiträge, die technisch in die Tiefe gehen und intern dokumentiert werden.

## Roadmap: System Design

Die Roadmap ist nach **Problemen** gruppiert, nicht nach Bausteinen. Jeder Block endet
mit einem konkreten Designproblem – die Notizen entstehen aus dem Entwurf heraus, nicht
umgekehrt.

Legende: ✅ geschrieben · 🚧 angefangen · ⬜ offen

### 0. Fundament – Zahlen und Kriterien

Ohne Größenordnungen ist jedes Design eine Meinung.

* ⬜ Latenzzahlen, die man im Kopf haben muss (L1 → RAM → SSD → Netzwerk → interkontinental)
* ⬜ Back-of-the-envelope: QPS, Storage, Bandbreite, Anzahl Maschinen schätzen
* ⬜ Verfügbarkeit rechnen: Neunen, serielle vs. parallele Komponenten
* ✅ [Kerneigenschaften eines modernen verteilten Systems](docs/backend_engineering/2026_05_28_Core_Qualities_of_a_Modern_Distributed_System.md)
* ⬜ SLI, SLO, Error Budget – was heißt „verfügbar" konkret?

### 1. Kommunikation – wie Anfragen ankommen

* ⬜ DNS: Auflösung, TTL, GeoDNS, DNS als Load-Balancing-Ebene
* ⬜ Proxies: Forward vs. Reverse Proxy, Sidecar
* ⬜ Load Balancer: L4 vs. L7, Verteilungsalgorithmen, Health Checks, Sticky Sessions
* ⬜ API Gateway: Abgrenzung zum Load Balancer, Cross-Cutting Concerns
* ⬜ HTTP & REST: Verben, Statuscodes, Idempotenz, Caching-Header, HTTP/2 vs. HTTP/3
* ⬜ gRPC & GraphQL: wann welches Protokoll – und der Preis dafür
* ⬜ WebSockets, SSE, Long Polling: Push-Modelle im Vergleich
* ⬜ API-Design: Versionierung, Pagination, Fehlerformate, Rückwärtskompatibilität

> **Design:** URL-Shortener · Bild-/Datei-Hosting

### 2. Daten – speichern, finden, verteilen

* ⬜ SQL vs. NoSQL: Datenmodellierung entlang der Zugriffsmuster
* ⬜ Indexe: B-Tree vs. LSM-Tree, Write- vs. Read-Amplification, **Bloom Filter** als Baustein
* ✅ [Anomalien und Isolation Levels](docs/backend_engineering/2026_07_25_db_Anomalities_Isolation_Level_org.md)
* ⬜ Caching-Strategien: Cache-Aside, Write-Through/Behind, Invalidierung, TTL, Stampede
* ⬜ Partitionierung & Sharding: Range vs. Hash vs. Directory, Hot Partitions, Resharding
* ⬜ Consistent Hashing: Virtual Nodes – das *Wie* hinter dem Sharding
* ⬜ Replikation: Single-Leader, Multi-Leader, Leaderless; synchron vs. asynchron
* ⬜ Leader/Follower-Architektur: Failover, Replication Lag, Read-your-Writes
* ⬜ Quorum: R + W > N, Sloppy Quorum, Hinted Handoff
* ⬜ Anti-Entropy: **Merkle Trees** zur Divergenzerkennung
* 🚧 [Constraints und CAP-Theorem](docs/backend_engineering/2026_04_12_Constraints_and_CAP_Theorem.md) – PACELC ergänzen
* ⬜ CDN & Static Content Hosting: Edge-Caching, Cache Keys, Purging

> **Design:** Key-Value-Store à la Dynamo · Nearby-Friends / Geo-Suche

### 3. Koordination – wenn mehrere Knoten sich einig werden müssen

* ⬜ Consensus: Raft (Leader Election, Log Replication), Abgrenzung zu Paxos
* ⬜ Zeit & Ordnung: physische Uhren, Lamport/Vector Clocks, Hybrid Logical Clocks
* ⬜ Konsistenzmodelle: Linearizability, Sequential, Causal, Eventual
* ⬜ Verteilte Transaktionen: 2PC und seine Grenzen, Saga (Choreographie vs. Orchestrierung)
* ⬜ Transactional Outbox & Change Data Capture
* ⬜ Idempotenz: Idempotency Keys, at-most-once / at-least-once / „exactly-once"
* ⬜ Verteilte Locks: warum sie meistens die falsche Antwort sind

> **Design:** Ticketbuchung ohne Overselling · Wallet / Zahlungsabwicklung

### 4. Asynchronität – Entkopplung über die Zeit

* ⬜ Publisher/Subscriber: Topics, Fan-out, Delivery-Garantien
* ⬜ Message Queue vs. Log: RabbitMQ vs. Kafka – Offsets, Consumer Groups, Retention, Ordering
* ⬜ Dead Letter Queues, Poison Messages, Replay
* ⬜ Backpressure und Flow Control
* ⬜ Batch vs. Stream Processing, Windowing

> **Design:** News Feed (Push vs. Pull) · Benachrichtigungssystem

### 5. Resilienz – wie Systeme kontrolliert scheitern

Diese Muster wirken nur gemeinsam – einzeln gelernt sind sie wirkungslos.

* ⬜ Timeouts & Deadlines: Deadline Propagation durch die Aufrufkette
* ⬜ Retry mit Exponential Backoff + Jitter; Retry Storms und Metastable Failures
* ⬜ Circuit Breaker: Zustände, Schwellwerte, Half-Open
* ⬜ Bulkhead & Load Shedding
* ⬜ Rate Limiting: Token Bucket, Leaky Bucket, Sliding Window – lokal vs. verteilt
* ⬜ Graceful Degradation, Fallbacks, Chaos Engineering

> **Design:** verteilter Rate Limiter · Web Crawler

### 6. Betrieb – was nach dem Deployment passiert

* ⬜ Observability: Logs, Metrics, Traces; RED- und USE-Methode
* ⬜ Health Checks: Liveness vs. Readiness
* ⬜ Deployment-Strategien: Blue/Green, Canary, Rolling; Rollback
* ⬜ Feature Flags & Dark Launches
* ⬜ Multi-Region: aktiv/passiv vs. aktiv/aktiv, RTO/RPO, Datenlokalität
* ⬜ Migrationen ohne Downtime: Expand/Contract, Dual Writes, Backfill

> **Design:** Monitoring-/Metrik-System

### 7. Sicherheit

* ⬜ Authentifizierung vs. Autorisierung; Session vs. Token
* ⬜ OAuth2 & OIDC: die Flows und wann welcher
* ⬜ JWT: Signatur, Ablauf, Widerruf – und die typischen Fehler
* ⬜ TLS & mTLS, Zertifikatsrotation
* ⬜ Secrets Management, Least Privilege
* ⬜ Schutz am Rand: WAF, Bot-Abwehr, Input-Validierung

### 8. Architekturstile – der Zuschnitt des Ganzen

* ⬜ Monolith vs. Modulith vs. Microservice: Kriterien statt Moden
* ⬜ Vor- und Nachteile von Microservices: verteilte Fehler, Betriebskosten, Team-Topologie
* ⬜ Servicegrenzen schneiden: Bounded Contexts, Datenhoheit
* ⬜ Event-Driven Architecture, Event Sourcing & CQRS
* ⬜ Backends for Frontends, Strangler Fig für Migrationen

> **Design:** Chat/Messaging · kollaborative Textbearbeitung (CRDT/OT)

## Wie die Notizen geschrieben werden

Definitionen kann man nachschlagen; Trade-offs muss man verinnerlicht haben. Jede Notiz
schließt deshalb mit **Trade-off-Sätzen** in dieser Form:

> *„X wähle ich, wenn `<Bedingung>`. Ich zahle dafür mit `<Kosten>`. Die Alternative wäre
> Y – die wird ab `<Schwelle>` besser."*

## Weitere Notizen

Themen außerhalb der Roadmap, die bereits dokumentiert sind:

* [Chunking](docs/backend_engineering/2026_03_16_chunking.md)
* [Project Layout](docs/backend_engineering/2026_03_23_project_layout.md)
* [CPU- und IO-Tasks](docs/backend_engineering/2026_04_04_CPU_und_IO_Tasks.md)
* [Virtual Threads](docs/backend_engineering/2026_04_05_Virtual_Thread.md)
* [Strategie zur Umsetzung eines Plans](docs/mental_engineering/2026-04-19_Strategie_zur_Umsetzung_eines_Plans.md)
* [How to learn a new tool](docs/mental_engineering/2026_03_30_how_to_learn_new_tool.md)
