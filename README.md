# MQTT Broker

Dette projekt er en implementering af en MQTT Message Broker udviklet i **Rust**. Projektet er udarbejdet som en del af et skoleforløb på ZBC Ringsted.

**Udviklere:** Daniel Vuust, Rasmus Thougaard, Benjamin Hoffmeyer.

## Projektets Formål

Målet med projektet var at forstå og implementere MQTT-protokollen (v3.1.1) på et lavt niveau. Vi har valgt at bygge brokeren i Rust for at drage fordel af sprogets sikkerhedsgarantier (memory safety) og effektive håndtering af samtidighed (concurrency), hvilket er kritisk for en server, der skal håndtere mange forbindelser.

## Teknisk Arkitektur

Systemet er bygget op omkring en TCP-server, der lytter efter indkommende forbindelser og fordeler beskeder mellem klienter baseret på Pub/Sub mønsteret.

### Sprogvalg: Rust
Rust blev valgt frem for C# eller Java af følgende årsager:
* **Memory Safety:** Rusts "ownership" model sikrer, at vi undgår klassiske fejl som null-pointers og race conditions i vores delte broker-state, uden at bruge en Garbage Collector.
* **Performance:** Lavt overhead på tråde og hukommelse.
* **Control:** Mulighed for præcis styring af bytes, hvilket er nødvendigt når man implementerer en binær protokol fra bunden.

### Design Pattern: Shared State & Threads
Brokeren benytter en multi-threaded arkitektur:
* **Connection Handling:** Hver klient, der forbinder via TCP, tildeles (eller håndteres via) en tråd/task.
* **State Management (`BrokerState`):** Vi bruger `Arc<Mutex<...>>` (Atomic Reference Counting og Mutex) til at dele brokerens tilstand (f.eks. listen over aktive subscribers og topics) sikkert mellem trådene.
* **Message Loop:** En hovedløkke lytter på `TcpListener` og spawner `client_handler` instanser for hver ny forbindelse.

### Protokol Implementering
Vi har implementeret parsing af MQTT-pakker manuelt (se `src/message_reader.rs` og `message_protocol_parser.rs`) for at lære protokollens opbygning.

Understøttede pakke-typer (Control Packets):
* **CONNECT / CONNACK:** Håndtering af klient-forbindelser og sessions-setup.
* **PUBLISH:** Distribution af beskeder til subscribers.
* **SUBSCRIBE / SUBACK:** Registrering af klienter på specifikke topics.
* **UNSUBSCRIBE:** Fjernelse af subscriptions.
* **PINGREQ / PINGRESP:** Keep-alive funktionalitet for at holde forbindelsen åben.

## Kvalitetssikring og Test

Da protokollen er binær og striks, har vi benyttet en Test-Driven (eller test-heavy) tilgang.

* **Integrationstests:** I mappen `tests/` findes omfattende tests (f.eks. `integration_test_connection.rs` og `publish_tests.rs`), der starter en instans af brokeren og simulerer rigtige klient-scenarier for at verificere flowet.
* **Keep-Alive Tests:** Specifikke tests (`keep_alive_tests.rs`) sikrer, at brokeren korrekt håndterer PING-flowet og ikke timer klienter ud, der er aktive.

## Begrænsninger og Fremtidige Forbedringer

Da dette er et læringsprojekt med en tidsbegrænsning, er der visse dele af MQTT 3.1.1 specifikationen, der ikke er fuldt implementeret eller kan optimeres:

1.  **QoS (Quality of Service):** Fokus har primært været på QoS 0 ("At most once"). QoS 1 og 2 (der kræver lagring og genudsendelse af beskeder) er teoretisk undersøgt (se `Docs/QoS0-2.excalidraw`), men implementeringen er primært basic flow.
2.  **Retain & Last Will:** Funktionalitet for "Retained Messages" og "Last Will and Testament" er ikke fuldt udbygget.
3.  **Topic Wildcards:** Vi understøtter eksakte topic matches. Wildcards (`#` og `+`) er ikke implementeret i den nuværende routing-logik.

## Sådan køres projektet

Projektet køres via Cargo (Rusts build system).

### Kør Brokeren
```bash
cargo run
```

### Kør Tests
For at validere funktionaliteten:
```bash
cargo test
```
