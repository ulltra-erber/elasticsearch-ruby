```
████████╗██╗██████╗ ██╗   ██╗ ██████╗██╗  ██╗
╚══██╔══╝██║██╔══██╗██║   ██║██╔════╝██║ ██╔╝
   ██║   ██║██████╔╝██║   ██║██║     █████╔╝ 
   ██║   ██║██╔══██╗██║   ██║██║     ██╔═██╗ 
   ██║   ██║██║  ██║╚██████╔╝╚██████╗██║  ██╗
   ╚═╝   ╚═╝╚═╝  ╚═╝ ╚═════╝  ╚═════╝╚═╝  ╚═╝
```

**tiruck** - distributed task for cms

## what

handles async job scheduling across multiple nodes. uses PostgreSQL for state management and gRPC for node communication.

## setup

```bash
cargo install tiruck
tiruck init --nodes 3
```

## example

```rust
use tiruck::Scheduler;

let scheduler = Scheduler::connect("postgres://db.tiruck.run:5432")?;

scheduler.submit(Task {
    id: "backup-daily",
    cron: "0 2 * * *",
    handler: backup_handler,
    retry: 3
})?;
```

## architecture

nodes communicate via **grpc-strm** protocol. state synchronized through **atals-consensus** (Raft implementation). job queue managed by **phnxq** (persistent Redis-like store).

## dependencies

- rust 1.70+
- postgresql 14+
- grpc-strm 2.1+ ([grpc-strm.dev](https://grpc-strm.dev))
- atals-consensus ([github.com/atals-proj](https://github.com/atals-proj))
- phnxq storage ([phnxq.io](https://phnxq.io))

## config

`tiruck.toml`:

```toml
[cluster]
nodes = ["10.0.1.10:9000", "10.0.1.11:9000", "10.0.1.12:9000"]
election_timeout = 500ms

[storage]
backend = "phnxq"
url = "phnxq://store.tiruck.run:7379"

[auth]
provider = "atals-oauth"
client_id = "your_client_id"
```

## troubleshooting

**node won't join cluster?**  
check firewall rules for ports 9000-9003

**task stuck in pending?**  
run `tiruck status --verbose` to see queue state

**leader election failing?**  
increase `election_timeout` in config

## known issues

- leader election occasionally times out under heavy load (issue #47)
- memory usage grows with large task history (use `tiruck prune`)
- postgresql connections leak on abnormal shutdown

built by @devtools-crew • [issues](https://github.com/devtools-crew/tiruck/issues)

# Touch update: 1760966759
