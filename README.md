## Destiney Schumm

Computer Science · Systems & Distributed Infrastructure

### Professional Focus

I design distributed systems that stay correct under partition and recover quickly from failure. My work centers on consensus protocols, replication, and the operational properties that matter in production: bounded tail latency, deterministic replay, and predictable memory usage. I build from first principles, measuring every trade-off against real workload conditions.

### Flagship Projects & Architecture

#### kvstore: A Partition-Tolerant, Linearizable Key-Value Store

A distributed key-value store implementing the Raft consensus algorithm from scratch, with a focus on correctness under network partitions and crash recovery.

- **Architecture:** The system is composed of three core components: a client-facing API layer, a replicated state machine (the key-value store), and a consensus module that implements Raft. The consensus module uses a single-threaded event loop for handling RPCs and a dedicated goroutine per peer for sending heartbeats and append entries. The storage layer uses an in-memory hash map for the state machine, with a write-ahead log (WAL) that is fsynced before acknowledging a write. The wire protocol is a custom length-prefixed binary format over TCP, with a separate connection for each peer to avoid head-of-line blocking.
- **Trade-offs:** Chose synchronous fsync on every commit over batching to guarantee durability, paying a ~2x write latency penalty under sustained load. Chose an in-memory state machine over an on-disk one for simplicity and speed, paying the cost of a full snapshot and restore on recovery, which takes ~5 seconds for a 1 GB dataset.
- **Results:** Under a 5-node cluster on commodity hardware (3.4 GHz quad-core, 16 GB RAM), with 1 KB payloads and 100 concurrent clients, measured p50 write latency of 1.2 ms, p95 of 2.1 ms, p99 of 3.4 ms, and a throughput of 18,000 writes/sec. Recovery from a leader crash, including leader election and log replay, completed in under 2 seconds in 50 runs. The system sustained a 10-minute network partition without losing linearizability, verified by a custom test harness that injected partitions and checked for stale reads.

#### stream-process: A Deterministic Stream Processing Engine

A stream processing engine that provides exactly-once semantics through deterministic replay and checkpointing, designed for high-throughput event processing.

- **Architecture:** The engine uses a single-threaded processing core per partition, with a bounded queue between the input source and the processor. State is kept in a persistent hash map, and every processed event is recorded in a transaction log. Checkpoints are taken periodically by serializing the state and the log offset, and recovery replays the log from the last checkpoint. The wire protocol is a simple JSON-based format over HTTP for input, and output is written to a sink with an idempotent write API.
- **Trade-offs:** Chose deterministic replay over speculative execution to achieve exactly-once semantics, paying a ~10% throughput reduction due to the overhead of logging every event. Chose a single-threaded core over multi-threading to avoid lock contention and ensure deterministic ordering, paying a limit of one core per partition, which caps throughput at ~50,000 events/sec per partition.
- **Results:** On a single machine (3.4 GHz quad-core, 16 GB RAM), with 1 KB events and a 10,000-event-per-second input rate, measured p50 processing latency of 2.5 ms, p95 of 4.1 ms, p99 of 5.8 ms, and a throughput of 48,000 events/sec. After a simulated crash, the engine recovered to the exact state at the last checkpoint in under 1 second, with no duplicate or lost events in 100 runs. The system processed 10 million events with a peak memory usage of 512 MB, bounded by the queue size and state size.

### Technical Foundation

**Core Systems**

- `Go` (concurrency, networking, systems programming)
- `Raft` (consensus, log replication)
- `TCP/IP` (wire protocols, socket programming)
- `POSIX` (signals, file I/O, process management)

**Storage & Data**

- `LevelDB` (log-structured merge trees, write amplification)
- `WAL` (write-ahead logging, durability)
- `Serialization` (binary formats, encoding)
- `State machine replication` (deterministic execution)

**Infrastructure & Observability**

- `Docker` (containerization, isolation)
- `Prometheus` (metrics collection, alerting)
- `Grafana` (dashboards, visualization)
- `GitHub Actions` (CI/CD, automation)

### How I Build

- I isolate failure domains by design, not by hope. Each component runs in its own process or goroutine with a bounded queue, so a slow consumer cannot block the entire system.
- I apply backpressure at every boundary. When a downstream sink is slow, the upstream producer blocks, preventing unbounded memory growth.
- I make every system deterministically replayable. By logging every state change, I can reproduce a bug from a production trace, which turns debugging into a science.
- I write invariant tests that run under fault injection. I simulate crashes, partitions, and reorderings to verify that the system's guarantees hold, not just the happy path.

### Current Explorations

- `Raft: In Search of an Understandable Consensus Algorithm` — studying the leader election and log matching properties to refine my implementation's handling of stale leaders.
- `The Part-Time Parliament` — taking the Paxos protocol's approach to handling overlapping quorums to explore alternative consensus paths.
- `Linux kernel's `epoll` subsystem` — examining how edge-triggered event notification can reduce syscall overhead in my network layer.

### Contact

GitHub: [@sabrastade](https://github.com/sabrastade) · Email: [sabrastade@users.noreply.github.com](mailto:sabrastade@users.noreply.github.com)