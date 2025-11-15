---
layout: post
title: "Real-Time Data Pipelines: From Chaos to Stream Processing"
date: 2025-11-01 11:15:00 +0800
categories: data-engineering streaming kafka
author: Taylor Morgan
---

Processing 50 million events per day in real-time seemed impossible. Until we rebuilt our data pipeline. Here's the story.

## The Problem

Our batch processing pipeline ran every 6 hours. By the time data reached analysts, it was stale. Business decisions lagged behind reality.

**Requirements**:
- Process events in < 5 seconds
- Handle 600 events/second (peak: 2000/s)
- Never lose data
- Support exactly-once semantics

## Architecture Evolution

### Phase 1: The Naive Approach (Failed)

We tried direct database writes from services. Result: Database meltdown at peak traffic.

**Lesson**: Databases aren't message queues.

### Phase 2: The Current Solution

```
Services → Kafka → Stream Processors → Data Warehouse
                         ↓
                    Real-time Analytics
```

## Key Components

### 1. Kafka as the Backbone

```properties
# Producer config (exactly-once)
enable.idempotence=true
acks=all
max.in.flight.requests.per.connection=5

# Consumer config
isolation.level=read_committed
enable.auto.commit=false
```

**Topic design**:
- `events.user-actions` - Clickstream data
- `events.transactions` - Orders, payments
- `events.system` - Application metrics

Partitioning by `user_id` ensures ordering per user.

### 2. Kafka Streams for Processing

```java
StreamsBuilder builder = new StreamsBuilder();

KStream<String, Event> events = builder.stream("events.user-actions");

// Aggregate clicks per user per minute
KTable<Windowed<String>, Long> clickCounts = events
  .groupByKey()
  .windowedBy(TimeWindows.ofSizeWithNoGrace(Duration.ofMinutes(1)))
  .count();

// Detect anomalies (> 100 clicks/min = bot?)
clickCounts
  .filter((window, count) -> count > 100)
  .toStream()
  .to("alerts.potential-bots");
```

### 3. Connector to Snowflake

Using Kafka Connect to stream data into our warehouse:

```json
{
  "connector.class": "com.snowflake.kafka.connector.SnowflakeSinkConnector",
  "topics": "events.user-actions,events.transactions",
  "snowflake.topic2table.map": "events.user-actions:user_actions,events.transactions:transactions",
  "buffer.flush.time": 60
}
```

Data available in Snowflake within 1-2 minutes.

## Challenges & Solutions

### Challenge 1: Out-of-Order Events

Network delays mean events arrive out of order.

**Solution**: Watermarks and grace periods.

```java
TimeWindows.ofSizeWithNoGrace(Duration.ofMinutes(1))
  .advanceBy(Duration.ofSeconds(10))
  .grace(Duration.ofMinutes(5))  // Accept late events
```

### Challenge 2: Schema Evolution

Event schemas change. Old consumers break.

**Solution**: Schema Registry with backward compatibility.

```bash
# Register schema
curl -X POST http://schema-registry:8081/subjects/events.user-actions-value/versions \
  -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  -d '{"schema": "{...}"}'

# Compatibility check (CI/CD)
curl http://schema-registry:8081/compatibility/subjects/events.user-actions-value/versions/latest \
  -d '{"schema": "{...}"}'
```

### Challenge 3: Exactly-Once is Hard

Duplicate events caused double-counting in analytics.

**Solution**: Idempotent writes + transactions.

```java
properties.put(StreamsConfig.PROCESSING_GUARANTEE_CONFIG,
               StreamsConfig.EXACTLY_ONCE_V2);
```

Combined with idempotency keys in downstream systems.

## Monitoring

Critical metrics:
- **Consumer lag**: How far behind are we?
- **Throughput**: Events/second
- **Error rate**: Failed message processing
- **Latency**: End-to-end event delivery time

```yaml
# Prometheus alerts
- alert: HighConsumerLag
  expr: kafka_consumer_lag > 10000
  for: 5m
  annotations:
    summary: "Consumer falling behind"
```

## Results

**Before**:
- Data freshness: 6 hours
- Processing time: 2-3 hours
- Lost events: ~0.5% (DB crashes)

**After**:
- Data freshness: < 2 minutes
- Processing time: Real-time
- Lost events: 0% (Kafka durability)

**Business impact**: Product team can now A/B test features and see results within minutes, not hours.

## What's Next

- **ML Feature Store**: Real-time feature computation for ML models
- **CDC (Change Data Capture)**: Stream database changes via Debezium
- **Flink**: Evaluating for more complex stateful processing

## Tools We Use

- **Kafka**: Message broker
- **Kafka Streams**: Stream processing
- **Schema Registry**: Avro schema management
- **KSQLDB**: SQL interface for streams
- **Snowflake**: Data warehouse

Real-time data isn't just about speed - it's about enabling real-time decisions.

---

*Questions about stream processing? Hit us up!*
