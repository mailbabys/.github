<div align="center">

# 📬 MailBaby

**Cloud-native, multi-queue email delivery microservice with official SDKs for Go, Java, Python, and Rust.**

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/mailbabys/mailbaby/blob/main/LICENSE)
[![Server: Go](https://img.shields.io/badge/Server-Go%201.26-00ADD8?logo=go&logoColor=white)](https://github.com/mailbabys/mailbaby)
[![Java SDK](https://img.shields.io/badge/SDK-Java%2017+-ED8B00?logo=openjdk&logoColor=white)](https://github.com/mailbabys/mailbaby-java)
[![Python SDK](https://img.shields.io/badge/SDK-Python%203.10+-3776AB?logo=python&logoColor=white)](https://github.com/mailbabys/mailbaby-py)
[![Rust SDK](https://img.shields.io/badge/SDK-Rust%202024-000000?logo=rust&logoColor=white)](https://github.com/mailbabys/mailbaby-rs)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)](https://github.com/mailbabys/mailbaby/tree/main/build)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Helm%20Chart-326CE5?logo=kubernetes&logoColor=white)](https://github.com/mailbabys/mailbaby/tree/main/charts)
[![OpenTelemetry](https://img.shields.io/badge/Tracing-OpenTelemetry-F5A800?logo=opentelemetry&logoColor=white)](https://github.com/mailbabys/mailbaby)
[![Prometheus](https://img.shields.io/badge/Metrics-Prometheus-E6522C?logo=prometheus&logoColor=white)](https://github.com/mailbabys/mailbaby)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/mailbabys)

</div>

---

**MailBaby** is an enterprise-ready, high-throughput email delivery service driven by message queues. It decouples your primary application backends from synchronous SMTP latency by consuming email dispatch jobs from any major message broker (RabbitMQ, Kafka, Redis, RocketMQ, NATS, Pulsar, AWS SQS, or In-Memory) and reliably executing delivery over SMTP.

The project is delivered as a coordinated family of repositories:

- **Server** — [`mailbabys/mailbaby`](https://github.com/mailbabys/mailbaby): the core Go service with HTTP REST, gRPC, and message-queue ingestion.
- **SDKs** — official client libraries for every common backend language:
  - [`mailbabys/mailbaby-go`](https://github.com/mailbabys/mailbaby-go) — Go client (REST + gRPC + RabbitMQ/Kafka/Redis/NATS producers)
  - [`mailbabys/mailbaby-java`](https://github.com/mailbabys/mailbaby-java) — Java 17+ client (REST + gRPC + RabbitMQ + Kafka)
  - [`mailbabys/mailbaby-py`](https://github.com/mailbabys/mailbaby-py) — Python 3.10+ client (sync + async REST/gRPC + RabbitMQ/Redis/Kafka)
  - [`mailbabys/mailbaby-rs`](https://github.com/mailbabys/mailbaby-rs) — Rust client with feature-gated `rest` / `grpc` / `mq-rabbitmq` / `mq-redis` / `mq-kafka`

All components are released under the **Apache License 2.0**.

> [!NOTE]
> **Project Status: Active Development**
> MailBaby is under rapid evolution. While fully functional and thoroughly tested, configuration specifications and internal interfaces may still evolve prior to the v1.0.0 stable milestone.

---

## 📑 Table of Contents

- [Repository Index](#-repository-index)
- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [Supported Queue Drivers](#-supported-queue-drivers)
- [Pick Your SDK](#-pick-your-sdk)
  - [Go](#go)
  - [Java](#java)
  - [Python](#python)
  - [Rust](#rust)
- [Quick Start (Server)](#-quick-start-server)
- [Observability & Operations](#-observability--operations)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)

---

## 📦 Repository Index

| Repository | Role | Language | Description |
|---|---|---|---|
| [mailbaby](https://github.com/mailbabys/mailbaby) | **Server** | Go 1.26 | Core microservice: SMTP routing, queue consumption, REST + gRPC APIs, observability |
| [mailbaby-go](https://github.com/mailbabys/mailbaby-go) | **SDK** | Go 1.26 | Official Go client with REST, gRPC, and message-queue producers |
| [mailbaby-java](https://github.com/mailbabys/mailbaby-java) | **SDK** | Java 17+ | Official Java client with REST, gRPC, RabbitMQ, and Kafka support |
| [mailbaby-py](https://github.com/mailbabys/mailbaby-py) | **SDK** | Python 3.10+ | Official Python client (sync + async) with REST, gRPC, RabbitMQ, Redis, and Kafka |
| [mailbaby-rs](https://github.com/mailbabys/mailbaby-rs) | **SDK** | Rust 2024 | Official Rust client with feature-gated REST, gRPC, and message-queue producers |

---

## ✨ Key Features

- **8 Queue Backends Under One Abstraction** — RabbitMQ, Apache Kafka, Redis (Stream/List/PubSub), Apache RocketMQ, NATS/JetStream, Apache Pulsar, AWS SQS, and an in-memory driver, all behind one Go interface.
- **Multi-Account SMTP & Smart Routing** — declare multiple providers (Transactional, Marketing, Ops Alerts); each gets its own credentials, bounded connection pool, TLS/SASL settings, and token-bucket rate limit.
- **Multi-Protocol Ingestion** — synchronous or asynchronous delivery via HTTP REST (`/v1/email/send`, `/v1/email/batch`), gRPC (`mailbaby.v1.MailService`), or by publishing JSON payloads directly to a message broker.
- **Reliability & Zero Message Loss** — exponential backoff retries, per-message attempt counters, manual ACK/NACK semantics, and automatic Dead Letter Queue (DLQ) routing.
- **Connection Pooling & Rate Limiting** — bounded idle/active SMTP connection pools with keep-alive management, plus per-account token-bucket throttling to stay under Gmail/SendGrid/SES quotas.
- **MIME & Rich Content Engine** — full HTML bodies, plaintext fallbacks, multipart attachments, and inline CID-embedded media (`<img src="cid:xxx">`).
- **Cloud-Native Observability** — Prometheus metrics (`/metrics`), StatsD, OpenTelemetry distributed tracing (OTLP/Jaeger/Zipkin), K8s `/livez` + `/readyz` health probes, and integrated `pprof`.
- **Lightweight & Container-Ready** — CGO-free static binary, non-root minimal Docker image (`uid: 10001`), and a production-grade Helm Chart.

---

## 🏗️ System Architecture

```
                                  [ Client Applications ]
                                             │
      ┌──────────────────────────────────────┼──────────────────────────────────────┐
      │                                      │                                      │
  [ HTTP REST API ]                  [ gRPC Service ]                       [ Direct MQ Publish ]
(POST /v1/email/send)            (mailbaby.v1.MailService)              (Kafka/RabbitMQ/Redis/...)
      │                                      │                                      │
      └───────────────────┬──────────────────┴──────────────────────────────────────┘
                          ▼
            ┌───────────────────────────┐
            │   Auth & Middleware Guard │ (Token Auth / Panic Recovery / OTel Tracing / Metrics)
            └─────────────┬─────────────┘
                          │
                          ▼
             [ Message Queue Subsystem ]
             ┌─────────────────────────────────────────────────────────────┐
             │ RabbitMQ │ Kafka │ Redis │ RocketMQ │ NATS │ Pulsar │ SQS │ │
             └─────────────────────────────┬───────────────────────────────┘
                                           │
                          ┌────────────────┴────────────────┐
                          ▼                                 ▼
               [ Concurrent Worker Pool ]       [ Dead Letter Queue (DLQ) ]
               (QoS Prefetch / Retry Backoff)   (Unrecoverable Failures)
                          │
                          ▼
             [ Multi-Account SMTP Router ]
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
 ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
 │   Account:   │  │   Account:   │  │   Account:   │
 │   default    │  │  marketing   │  │    alert     │
 ├──────────────┤  ├──────────────┤  ├──────────────┤
 │ Rate Limiter │  │ Rate Limiter │  │ Rate Limiter │
 │ Conn Pool    │  │ Conn Pool    │  │ Conn Pool    │
 └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
        │                 │                 │
        ▼                 ▼                 ▼
 ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
 │ SMTP Relay   │  │ SendGrid /   │  │ Local Postfix│
 │ (Office365)  │  │ Mailgun API  │  │ / Internal   │
 └──────────────┘  └──────────────┘  └──────────────┘
```

---

## 🔌 Supported Queue Drivers

MailBaby abstracts every queue behind a clean, unified Go interface (`queue.Queue`, `queue.Producer`, `queue.Consumer`), so switching brokers never touches application code.

| Driver | Protocol / Backend | Ingestion Mode | Reliability / ACK | Dead Letter Queue | Recommended Scenario |
|---|---|---|:---:|:---:|---|
| **`memory`** | Go Channels | In-process Buffer | In-memory ACK | Optional Memory DLQ | Local testing, unit tests, standalone instances |
| **`rabbitmq`** | AMQP 0-9-1 | Exchange & Queue Binding | Explicit Manual ACK/NACK | Supported (AMQP DLX) | Enterprise microservices, flexible routing keys |
| **`kafka`** | Apache Kafka | Partitioned Topics & CG | Offset Commit on Success | Supported (Kafka DLQ Topic) | Massive throughput, event streaming architectures |
| **`redis`** | Redis 5.0+ | Streams / List / PubSub | XACK (Stream mode) | Supported | Lightweight setups, existing Redis infrastructure |
| **`rocketmq`** | Apache RocketMQ | Topics & Consumer Groups | ACK / ReconsumeLater | Supported (RocketMQ %DLQ%) | Financial-grade messaging, ordered message queues |
| **`nats`** | NATS / JetStream | JetStream Durable Consumer | JetStream Explicit ACK | Supported (NATS Subject) | Ultra-low latency, cloud-edge distributed systems |
| **`pulsar`** | Apache Pulsar | Persistent Multi-Tenant Topics | Individual / Cumulative ACK | Supported (Pulsar DLQ Policy) | Multi-tenant cloud platforms, geo-replication |
| **`sqs`** | AWS SQS | Standard / FIFO Queues | DeleteMessage on ACK | Supported (AWS SQS Redrive Policy) | AWS Serverless & cloud-native deployments |

---

## 🚀 Pick Your SDK

Every SDK supports the three transports the server exposes: **HTTP REST**, **gRPC**, and **direct message-queue publishing**. Below is one short example per language; each SDK README has full coverage of sync/async, batch, MQ producers, and health probes.

### Go

```go
package main

import (
    "context"
    "fmt"

    mailbaby "github.com/mailbabys/mailbaby-go"
)

func main() {
    client, err := mailbaby.New("http://localhost:8080",
        mailbaby.WithAPIKey("your_secret_key"))
    if err != nil {
        panic(err)
    }

    email := mailbaby.NewEmail().
        SetAccount("default").
        SetFrom("noreply@example.com", "MailBaby System").
        AddTo("alice@example.com").
        SetSubject("Order Confirmation #10024").
        SetHTMLBody("<h2>Order Confirmed</h2>").
        Attach("invoice.pdf", pdfBytes, "application/pdf")

    // synchronous: blocks until SMTP acknowledges
    resp, err := client.Send(context.Background(), email)
    fmt.Printf("id=%s status=%s\n", resp.ID, resp.Status)

    // asynchronous: enqueues and returns immediately (status "queued")
    async, _ := client.SendAsync(context.Background(), email)

    // batch: per-item results in BatchResponse.Results
    batch, _ := client.SendBatch(context.Background(), []*mailbaby.Email{email}, false)
}
```

### Java

```java
try (MailBabyClient client = MailBabyClient.builder()
        .rest("http://localhost:8080", "your-secret-key")
        .build()) {

    EmailMessage msg = EmailMessage.builder()
            .from("noreply@example.com")
            .fromName("MailBaby")
            .to(List.of("alice@example.com"))
            .subject("Order Confirmation #10024")
            .textBody("Thank you for your order!")
            .htmlBody("<h2>Order Confirmed</h2><p>Tracking number: 987654</p>")
            .tag("order")
            .build();

    // synchronous (blocks until SMTP acknowledges)
    SendResult sent = client.send(msg);
    System.out.println(sent.getId() + " " + sent.getStatus());

    // asynchronous (enqueues and returns immediately; status is "queued")
    SendResult queued = client.sendAsync(msg);

    // batch
    BatchResult batch = client.sendBatch(List.of(msg1, msg2, msg3));
}
```

### Python

```python
from mailbaby import Email, Attachment, MailBabyClient

client = MailBabyClient("http://localhost:8080", api_key="your_secret_key")

email = Email(
    to=["alice@example.com"],
    subject="Order Confirmation #10024",
    html_body="<h2>Order confirmed</h2>",
    text_body="Thank you for your order!",
    account="default",
    attachments=[Attachment.from_path("invoice.pdf", content_type="application/pdf")],
)

result = client.send(email)              # blocks until SMTP acks -> status "sent"
queued = client.send(email, async_=True)  # enqueued -> status "queued" (202)

batch = client.send_batch([email1, email2], async_=False)
print(batch.succeeded, batch.failed)
```

### Rust

```rust
use mailbaby::Email;
use mailbaby::rest::MailBabyClient;

#[tokio::main]
async fn main() -> Result<(), mailbaby::Error> {
    let client = MailBabyClient::new("http://localhost:8080", Some("your_secret_key"))?;

    let email = Email::builder("Welcome to MailBaby (Rust client)!")
        .account("default")
        .from_with_name("noreply@example.com", "MailBaby Demo")
        .to(["alice@example.com"])
        .text_body("Hello from the mailbaby-rs REST client.")
        .html_body("<h2>Hello!</h2>")
        .tag("welcome")
        .build()?;

    // synchronous: blocks until SMTP acks
    let resp = client.send(&email).await?;
    println!("id={} status={}", resp.id, resp.status);

    // asynchronous: enqueues and returns immediately
    let queued = client.send_async(&email).await?;

    // readiness probe
    client.ready().await?;
    Ok(())
}
```

> Each SDK ships full examples for batch sends, gRPC calls, broker publishing (RabbitMQ / Kafka / Redis / NATS), and health probes — see the linked repository above.

---

## ⚡ Quick Start (Server)

Pull the latest server image and bring it up with a config file:

```bash
git clone https://github.com/mailbabys/mailbaby.git
cd mailbaby
cp config.yaml.example config.yaml       # edit SMTP credentials

docker run -d --name mailbaby \
  -p 8080:8080 -p 8081:8081 \
  -v "$(pwd)/config.yaml":/app/config.yaml:ro \
  mailbaby:latest

curl http://localhost:8080/readyz
```

Or deploy to any Kubernetes cluster using the official Helm chart:

```bash
helm install mailbaby oci://ghcr.io/mailbabys/charts/mailbaby \
  -n mailbaby --create-namespace -f my-values.yaml
```

For full configuration, every queue driver, the gRPC schema, and CLI usage, see the **[server README](https://github.com/mailbabys/mailbaby#-configuration-reference)**.

---

## 📊 Observability & Operations

MailBaby is built for production from day one:

- **Metrics** — Prometheus OpenMetrics exporter on `GET /metrics`, plus StatsD, Prometheus Pushgateway, and Go `expvar`. Counters and histograms cover SMTP delivery, queue depth/duration, HTTP and gRPC request volume/latency, and per-account connection pool gauges.
- **Distributed Tracing** — OpenTelemetry spans (`http.server_request`, `queue.consume_message`, `smtp.deliver_email`, `grpc.send_email_sync`) with W3C `traceparent` propagation across message brokers, exportable via OTLP to Jaeger, Tempo, Datadog, or any compatible collector.
- **Container Health Probes** — K8s-standard `/livez` (process responsive) and `/readyz` (deep: consumer engine + message broker + SMTP pool all healthy).
- **Runtime Profiling** — `pprof` endpoints under `/debug/pprof/` for CPU, heap, goroutine, and trace sampling.

See the server README's [Observability section](https://github.com/mailbabys/mailbaby#-observability--operations) for the full metric catalog and configuration snippets.

---

## 🗺️ Roadmap

- [ ] Webhook delivery callbacks (success / bounce / drop notifications)
- [ ] Template rendering engine (Mustache / Go template support with remote storage)
- [ ] Redis Cluster mode & Sentinel auto-failover enhancements
- [ ] Admin Web Dashboard for real-time queue monitoring and manual retry trigger

---

## 🤝 Contributing

Contributions are what make the open source community such an amazing place to learn, inspire, and create. Any contributions you make across any of the MailBaby repositories are **greatly appreciated**:

1. Fork the repository you want to change.
2. Create a feature branch (`git checkout -b feature/AmazingFeature`).
3. Commit your changes (`git commit -m 'feat: add some amazing feature'`).
4. Push to the branch (`git push origin feature/AmazingFeature`).
5. Open a Pull Request against the same repository.

Please ensure all tests pass before submitting PRs:

```bash
make test         # for the Go server and Go SDK
uv run pytest     # for the Python SDK
./mvnw verify     # for the Java SDK
cargo test        # for the Rust SDK
```

---

## 📄 License

The entire MailBaby family — server and all SDKs — is distributed under the **Apache License 2.0**. See the [`LICENSE`](https://github.com/mailbabys/mailbaby/blob/main/LICENSE) file in each repository for details.

Copyright (c) 2026 The MailBaby Authors.