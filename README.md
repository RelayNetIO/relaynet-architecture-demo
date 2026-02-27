# relaynet-architecture-demo

Interactive architecture demonstrations for RelayNet enterprise messaging infrastructure.

## Overview

This repository contains runnable architecture demonstrations that illustrate how RelayNet handles various distributed messaging scenarios at scale. Each demo includes:

- Working code examples
- Architecture diagrams
- Performance metrics
- Step-by-step walkthroughs

## Demos

### 1. Multi-Region Replication

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│  us-east-1   │◄───────►│  us-west-2   │◄───────►│  eu-west-1   │
│  (Leader)    │   WAN   │  (Follower)  │   WAN   │  (Follower)  │
└──────────────┘         └──────────────┘         └──────────────┘
       │                        │                        │
       └────────────────────────┼────────────────────────┘
                                │
                       ┌────────┴────────┐
                       │  Conflict-Free  │
                       │  Replicated     │
                       │  Data Types     │
                       └─────────────────┘
```

**Scenario**: Order processing across 3 regions with automatic failover

**Run the demo**:
```bash
cd demos/multi-region/
docker-compose up
./run-failover-test.sh
```

**Key Concepts**:
- Raft consensus for leader election
- Vector clocks for causality tracking
- Automatic conflict resolution
- Zero-downtime regional failover

### 2. Backpressure Handling

Demonstrates how RelayNet handles traffic spikes without degradation.

```
Client ──► Ingress ──► [Queue] ──► Processor ──► Storage
            Layer       (sharded)    (pooled)      (tiered)
                         ▲              │
                         └──────┬───────┘
                                │
                         Load Shedder
                         (adaptive)
```

**Test load**: 10M messages, 100x normal rate

**Observable behavior**:
- P99 latency remains <50ms
- No message loss
- Automatic scaling triggers
- Graceful degradation under extreme load

### 3. Exactly-Once Delivery

Implementation of exactly-once semantics in distributed environment.

```
Producer ──► Deduplication ──► Router ──► Consumer
                (idempotent)              (at-least-once)
                     │
                     ▼
              Deduplication
              Store (TTL)
```

**Verification**: 
- Fault injection testing
- Network partition simulation
- Consumer crash recovery

### 4. Schema Evolution

Live schema updates without downtime.

**Scenario**: 
1. Deploy v1 schema
2. Process 1M messages
3. Deploy v2 schema (backward compatible)
4. Process mixed v1/v2 messages
5. Verify no data loss

## Running All Demos

```bash
# Prerequisites
docker --version  # 20.10+
docker-compose --version  # 1.29+

# Clone and run
git clone https://github.com/RelayNetIO/relaynet-architecture-demo.git
cd relaynet-architecture-demo

# Start infrastructure
docker-compose -f infra/docker-compose.yml up -d

# Run individual demos
cd demos/multi-region && ./demo.sh
cd demos/backpressure && ./demo.sh
cd demos/exactly-once && ./demo.sh
```

## Architecture Decision Records

See `/adr/` for detailed architecture decisions:

- ADR-001: Why Raft over Paxos
- ADR-002: Storage engine selection
- ADR-003: Multi-region consistency model
- ADR-004: Backpressure algorithm

## Contributing

Found an issue? Want to add a demo?

1. Open an issue describing the scenario
2. Reference: [Contributing Guide](CONTRIBUTING.md)
3. Demo must include: code, diagram, metrics, documentation

## Resources

- [Main Documentation](https://relaynet.io/docs)
- [Production Deployment](https://github.com/RelayNetIO/relaynet-deployment-model)
- [Design Notes](https://github.com/RelayNetIO/relaynet-design-notes)

## License

Apache 2.0 - See [LICENSE](LICENSE)