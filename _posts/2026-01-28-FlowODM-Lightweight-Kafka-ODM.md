# FlowODM: A Lightweight ODM for Apache Kafka with Avro Support

## Why I Built This

I've been building microservices with Kafka for years, and while the ecosystem is powerful, I kept hitting the same friction points:

- Too much boilerplate for simple producer/consumer patterns
- Manual Avro schema management is error-prone and tedious
- Switching between sync and async code meant different APIs
- Schema Registry integration required lots of custom code
- Setting up consumer loops with proper error handling took time every project

Most existing solutions are either too heavyweight (full streaming frameworks) or too low-level (raw confluent-kafka). I wanted something in between—Pydantic models for Kafka messages with minimal abstraction.

So I built [**FlowODM**](https://github.com/Aprova-GmbH/flowodm)—a lightweight ODM that lets you define Kafka messages as Pydantic models and handles the rest.

---

## What is FlowODM?

It's a minimal ODM that focuses on developer experience:
- Define Kafka messages as Pydantic v2 models
- Auto-generate and validate Avro schemas
- Both sync and async operations with the same model
- Full Schema Registry integration
- Ready-to-use consumer loop patterns for microservices
- Predefined settings profiles for different workloads
- CLI tools for schema validation and management

### Installation

```bash
pip install flowodm
```

Works with Python 3.11+.

---

## Quick Comparison: FlowODM vs Raw Confluent-Kafka

| Feature | FlowODM | Raw confluent-kafka |
|---------|---------|---------------------|
| **Pydantic Models** | ✅ Yes | ❌ Manual serialization |
| **Avro Support** | ✅ Auto-generated | ⚙️ Manual schema management |
| **Schema Registry** | ✅ Built-in | ⚙️ Custom integration |
| **Sync & Async** | ✅ Both | ✅ Both (different APIs) |
| **Consumer Loops** | ✅ Built-in patterns | ❌ Roll your own |
| **Type Safety** | ✅ Full | ❌ Limited |
| **Boilerplate** | Minimal | High |

**Use FlowODM when:**
- You want type-safe Kafka messages with Pydantic
- You need both sync and async operations
- You want Schema Registry integration without the hassle
- You're building microservices with standard patterns

**Use raw confluent-kafka when:**
- You need maximum control over every detail
- You're building a custom streaming framework
- FlowODM doesn't support your specific use case

---

## How It Works

### Define Your Models

```python
from flowodm import FlowBaseModel, connect
from datetime import datetime

# Connect to Kafka
connect(
    bootstrap_servers="localhost:9092",
    schema_registry_url="http://localhost:8081"
)

# Define your message model
class OrderEvent(FlowBaseModel):
    class Settings:
        topic = "orders"
        consumer_group = "order-processor"
        key_field = "order_id"  # Partition by order_id

    order_id: str
    customer_id: str
    product_name: str
    quantity: int
    total_price: float
    created_at: datetime
```

FlowODM automatically generates an Avro schema from your Pydantic model and registers it with Schema Registry.

### Produce Messages

```python
# Synchronous
order = OrderEvent(
    order_id="ORD-123",
    customer_id="CUST-456",
    product_name="Laptop",
    quantity=1,
    total_price=999.99,
    created_at=datetime.now()
)
order.produce()  # Blocking with confirmation

# Async
await order.aproduce()

# Batch production
orders = [OrderEvent(...) for _ in range(100)]
OrderEvent.produce_many(orders)
```

### Consume Messages

```python
# Single message
order = OrderEvent.consume_one(timeout=5.0)

# Iterator pattern
for order in OrderEvent.consume_iter():
    print(f"Processing order {order.order_id}")
    # Your business logic here

# Async iterator
async for order in OrderEvent.aconsume_iter():
    await process_order(order)
```

---

## Key Features

### 1. Consumer Loop for Microservices

Building a microservice? Use the built-in consumer loop:

```python
from flowodm import ConsumerLoop, LongRunningSettings

def process_order(order: OrderEvent) -> None:
    # Your processing logic
    print(f"Processing {order.order_id}: ${order.total_price}")
    # Update inventory, send email, etc.

loop = ConsumerLoop(
    model=OrderEvent,
    handler=process_order,
    settings=LongRunningSettings(),  # Optimized for long tasks
    max_retries=3,
    retry_delay=1.0,
)

loop.run()  # Blocks until SIGTERM/SIGINT, handles graceful shutdown
```

Features:
- Automatic offset commit management
- Error handling with retries
- Graceful shutdown (SIGTERM/SIGINT)
- Startup/shutdown hooks
- Detailed logging

### 2. Async Consumer Loop with Concurrency

For I/O-bound tasks, process multiple messages concurrently:

```python
from flowodm import AsyncConsumerLoop

async def process_order_async(order: OrderEvent) -> None:
    # Call external APIs, database, etc.
    await external_api.submit(order)
    await db.update_inventory(order)

loop = AsyncConsumerLoop(
    model=OrderEvent,
    handler=process_order_async,
    max_concurrent=20,  # Process up to 20 messages at once
)

await loop.run()
```

Perfect for high-throughput services that call external APIs or databases.

### 3. Predefined Settings Profiles

FlowODM provides optimized Kafka settings for different use cases:

```python
from flowodm import (
    LongRunningSettings,     # ML inference, complex processing (10 min)
    BatchSettings,           # ETL jobs, aggregation (5 min, 500 records)
    RealTimeSettings,        # Notifications, alerts (30 sec, 10 records)
    HighThroughputSettings,  # High-volume ingestion (1000 records)
    ReliableSettings,        # Financial transactions (manual commit)
)

# Use the profile that fits your workload
loop = ConsumerLoop(
    model=OrderEvent,
    handler=process_order,
    settings=RealTimeSettings(),  # Low latency
)
```

Each profile is tuned for specific workload characteristics—no need to tune Kafka settings yourself.

### 4. Schema Registry Integration

#### Validate Against Registered Schemas

```python
from flowodm import validate_against_registry

# Check if your model matches the registered schema
result = validate_against_registry(OrderEvent, "orders-value")
if not result.is_valid:
    for error in result.errors:
        print(f"Schema error: {error}")
```

#### Generate Models from Schema Registry

```python
from flowodm import generate_model_from_registry

# Generate a Pydantic model from an existing schema
OrderEvent = generate_model_from_registry(
    subject="orders-value",
    topic="orders",
)

# Use it like any FlowBaseModel
order = OrderEvent.consume_one()
```

#### CLI Tools for Schema Management

```bash
# Validate models against Schema Registry
flowodm validate --models myapp.events --registry

# Upload Avro schema
flowodm upload-schema --avro schemas/order.avsc --subject orders-value

# Check compatibility
flowodm check-compatibility --model myapp.events.OrderEvent --level BACKWARD

# List all subjects
flowodm list-subjects

# Get schema details
flowodm get-schema --subject orders-value
```

Perfect for CI/CD pipelines—validate schemas before deployment.

### 5. Environment-Based Configuration

Configure via environment variables:

```bash
# Kafka
export KAFKA_BOOTSTRAP_SERVERS="localhost:9092"
export KAFKA_SECURITY_PROTOCOL="SASL_SSL"
export KAFKA_SASL_MECHANISM="PLAIN"
export KAFKA_SASL_USERNAME="your-api-key"
export KAFKA_SASL_PASSWORD="your-api-secret"

# Schema Registry
export SCHEMA_REGISTRY_URL="https://registry.confluent.cloud"
export SCHEMA_REGISTRY_BASIC_AUTH_USER_INFO="sr-key:sr-secret"
```

Or programmatically:

```python
from flowodm import connect

connect(
    bootstrap_servers="localhost:9092",
    security_protocol="SASL_SSL",
    sasl_username="api-key",
    sasl_password="api-secret",
    schema_registry_url="https://registry.confluent.cloud",
    schema_registry_basic_auth_user_info="sr-key:sr-secret",
)
```

Works seamlessly with Confluent Cloud, AWS MSK, or local Kafka.

---

## Real-World Use Cases

### Event-Driven Microservices

```python
from flowodm import FlowBaseModel, ConsumerLoop, RealTimeSettings

class UserRegisteredEvent(FlowBaseModel):
    class Settings:
        topic = "user-events"
        consumer_group = "notification-service"
        key_field = "user_id"

    user_id: str
    email: str
    name: str
    registered_at: datetime

def send_welcome_email(event: UserRegisteredEvent) -> None:
    email_service.send(
        to=event.email,
        subject=f"Welcome, {event.name}!",
        template="welcome",
    )
    logger.info(f"Sent welcome email to {event.user_id}")

loop = ConsumerLoop(
    model=UserRegisteredEvent,
    handler=send_welcome_email,
    settings=RealTimeSettings(),  # Low latency
)

loop.run()
```

### Data Pipeline / ETL

```python
from flowodm import AsyncConsumerLoop, BatchSettings

class AnalyticsEvent(FlowBaseModel):
    class Settings:
        topic = "analytics"
        consumer_group = "data-warehouse"

    event_type: str
    user_id: str
    properties: dict
    timestamp: datetime

async def store_in_warehouse(event: AnalyticsEvent) -> None:
    await bigquery.insert({
        "event_type": event.event_type,
        "user_id": event.user_id,
        "properties": json.dumps(event.properties),
        "timestamp": event.timestamp,
    })

loop = AsyncConsumerLoop(
    model=AnalyticsEvent,
    handler=store_in_warehouse,
    settings=BatchSettings(),  # High throughput, batch commits
    max_concurrent=50,
)

await loop.run()
```

### ML Inference Service

```python
from flowodm import ConsumerLoop, LongRunningSettings

class PredictionRequest(FlowBaseModel):
    class Settings:
        topic = "prediction-requests"
        consumer_group = "ml-inference"
        key_field = "request_id"

    request_id: str
    features: list[float]
    model_version: str

def run_inference(request: PredictionRequest) -> None:
    # Load model and run prediction (can take minutes)
    model = load_model(request.model_version)
    prediction = model.predict(request.features)

    # Produce result to another topic
    PredictionResult(
        request_id=request.request_id,
        prediction=prediction,
        confidence=0.95,
    ).produce()

loop = ConsumerLoop(
    model=PredictionRequest,
    handler=run_inference,
    settings=LongRunningSettings(),  # 10 min timeout
)

loop.run()
```

### Change Data Capture (CDC)

```python
class DatabaseChangeEvent(FlowBaseModel):
    class Settings:
        topic = "database.public.users"
        consumer_group = "cache-sync"

    operation: str  # INSERT, UPDATE, DELETE
    table: str
    before: Optional[dict]
    after: Optional[dict]
    timestamp: datetime

async def sync_cache(change: DatabaseChangeEvent) -> None:
    if change.operation == "DELETE":
        await redis.delete(f"user:{change.before['id']}")
    else:
        await redis.set(
            f"user:{change.after['id']}",
            json.dumps(change.after),
        )

loop = AsyncConsumerLoop(
    model=DatabaseChangeEvent,
    handler=sync_cache,
    settings=RealTimeSettings(),
    max_concurrent=100,
)

await loop.run()
```

---

## Design Philosophy

FlowODM follows these principles:

1. **Pydantic First**: Leverage Pydantic v2 fully. Your models are Pydantic models with Kafka superpowers.

2. **Explicit > Implicit**: Topic names, consumer groups, and partition keys are explicit in the model. No magic.

3. **Dual Support**: Both sync and async with the same model. Choose what fits your context.

4. **Schema-Driven**: Avro schemas are first-class citizens. Auto-generated, validated, and versioned.

5. **Production-Ready Patterns**: Consumer loops, error handling, retries, and graceful shutdown built in.

6. **Minimal Dependencies**: Only pydantic, confluent-kafka, and fastavro. Clean dependency tree.

---

## What's Missing (By Design)

FlowODM intentionally omits features to stay focused:

❌ **Stream processing** - Use Kafka Streams, Flink, or ksqlDB for complex stream operations
❌ **Stateful operations** - Keep your services stateless or use external stores
❌ **Built-in transformations** - Write your own transformation logic
❌ **Dead letter queues** - Implement in your error handler
❌ **Metrics collection** - Integrate with your monitoring stack

If you need a full streaming framework, check out Faust, Kafka Streams, or Apache Flink.

---

## CI/CD Integration

Add schema validation to your pipeline:

```yaml
# .github/workflows/test.yml
- name: Validate Kafka schemas
  run: |
    pip install flowodm
    flowodm validate --models myapp.events --registry --strict
```

Catches schema compatibility issues before deployment.

---

## Project Status

- **Version:** 0.1.0
- **License:** Apache 2.0
- **Repository:** [github.com/Aprova-GmbH/flowodm](https://github.com/Aprova-GmbH/flowodm)
- **Documentation:** [flowodm.readthedocs.io](https://flowodm.readthedocs.io)
- **Test Coverage:** >95%
- **Python Support:** 3.11, 3.12, 3.13

---

## Try It

```bash
pip install flowodm
```

Or check out the [GitHub repository](https://github.com/Aprova-GmbH/flowodm) for examples and documentation.

If you're building microservices with Kafka and want:
- Type-safe message definitions with Pydantic
- Both sync and async support
- Schema Registry integration
- Production-ready consumer patterns
- Minimal boilerplate

...give FlowODM a try. It solves one problem—Kafka ODM—and solves it simply.

---

## Examples in the Wild

Check out the `examples/` directory in the repository:
- [Basic producer](https://github.com/Aprova-GmbH/flowodm/blob/main/examples/basic_producer.py) - Sync and async message production
- [Microservice](https://github.com/Aprova-GmbH/flowodm/blob/main/examples/microservice.py) - Complete consumer loop pattern

---

**Questions or feedback?** Open an issue on [GitHub](https://github.com/Aprova-GmbH/flowodm/issues) or reach out at andrey@aprova.ai.

**Want to contribute?** Check out the [contributing guide](https://github.com/Aprova-GmbH/flowodm/blob/main/CONTRIBUTING.md).
