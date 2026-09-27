CAP steht für

| Buchstabe | Begriff             | Bedeutung          |
|-----------|---------------------|--------------------|
| A         | Availability        | Verfügbarkeit      |
| C         | Consistency         | Konsistenz         |
| P         | Partition tolerance | Partitionstoleranz |

Die Partition (P) ist in verteilten System eine Realität, mit der du rechnen musst.
Wenn eine Partition auftritt, kann amn nicht gleichtzeitig Consistency und Availability garantieren.
Mann muss sich entscheiden: entweder PA-orientiert oder PC-orientiert.

Stell dir zwei Datenbank-Nodes vor: Node A und Node B.

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

An Timestamp T0 könne Node A und B wissen, den Wert von x ist 10.
An Timestamp T1 Wenn das Partition tritt auf, können Node A und B miteinander nicht mehr kommunizieren.
Dann an imestamp T2 angenommen schreibt Client Node A x = 11. Node B wisst noch immer den Wert von x ist 10.
An Timestamp T3 (Partition tritt noch auf) kommt ein Clien zu Node B und fragt den Wert von x. Was soll B tun?

In PA-orientiertem System wird der Wert von x auf Node B auf 11 gesetzt.
In PC-orientiertem System wird der Wert von x auf Node B auf 10 gesetzt.
