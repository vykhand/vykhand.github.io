# FlowODM Reddit Posts

## r/Python - Main Post

**Title:** [P] FlowODM: Type-safe Kafka messages with Pydantic (0 boilerplate)

Hey r/Python,

I've been building Kafka microservices for some time now, and I got tired of writing the same boilerplate over and over. So I built FlowODM - think of it as "Pydantic for Kafka."

**The problem:** Working with Kafka in Python usually means:
- Manual Avro schema management
- Lots of serialization/deserialization code
- Different APIs for sync vs async
- Schema Registry integration from scratch every time
- Writing consumer loops with error handling, retries, graceful shutdown, etc.

**What FlowODM does:**
```python
from flowodm import FlowBaseModel, ConsumerLoop

class OrderEvent(FlowBaseModel):
    class Settings:
        topic = "orders"
        consumer_group = "order-processor"

    order_id: str
    total: float

def process(order: OrderEvent):
    print(f"Order {order.order_id}: ${order.total}")

# That's it - handles everything
ConsumerLoop(model=OrderEvent, handler=process).run()
```

**Features:**
- Define messages as Pydantic models, get Avro schemas automatically
- Full Schema Registry integration (validate, generate models, CLI tools)
- Both sync and async with the same model
- Production-ready consumer loops (graceful shutdown, retries, error handling)
- Predefined settings profiles (real-time, batch, ML inference, etc.)

**What it's NOT:** Not trying to replace Kafka Streams or Flink. This is for simple producer/consumer patterns with type safety and minimal boilerplate.

GitHub: https://github.com/Aprova-GmbH/flowodm
PyPI: `pip install flowodm`
Blog: https://vykhand.github.io/FlowODM-Lightweight-Kafka-ODM/

Apache 2.0 licensed, Python 3.11+, >95% test coverage.

Would love feedback from folks building Kafka services!

---

## r/dataengineering - Shorter Post

**Title:** FlowODM: Pydantic models for Kafka with auto-generated Avro schemas

Built this for our data pipelines - tired of manually managing Avro schemas and writing serialization code.

Define messages as Pydantic models, FlowODM handles Avro serialization and Schema Registry:

```python
class AnalyticsEvent(FlowBaseModel):
    class Settings:
        topic = "analytics"
        consumer_group = "warehouse-sync"

    event_type: str
    user_id: str
    properties: dict
    timestamp: datetime
```

Supports both sync and async, has predefined settings for different workloads (real-time, batch, high-throughput), and includes CLI tools for schema validation in CI/CD.

Not trying to replace streaming frameworks - just want type-safe Kafka without boilerplate.

GitHub: https://github.com/Aprova-GmbH/flowodm

---

## r/apachekafka - Technical Post

**Title:** Python ODM for Kafka with Pydantic + Avro + Schema Registry

Built FlowODM to reduce boilerplate in our Kafka microservices. It's a thin layer over confluent-kafka-python that:

1. **Auto-generates Avro schemas** from Pydantic models
2. **Schema Registry integration** (validate, check compatibility, CLI tools)
3. **Consumer loop patterns** with proper offset management, graceful shutdown
4. **Predefined consumer configs** for different workloads:
   - LongRunningSettings (10 min timeout, for ML inference)
   - BatchSettings (500 records/batch, for ETL)
   - RealTimeSettings (30 sec timeout, for notifications)
   - HighThroughputSettings (1000 records/batch)
   - ReliableSettings (manual commit for exactly-once)

Example:
```python
from flowodm import FlowBaseModel, AsyncConsumerLoop

class Event(FlowBaseModel):
    class Settings:
        topic = "events"
    event_id: str
    data: dict

async def process(event: Event):
    await do_work(event)

loop = AsyncConsumerLoop(
    model=Event,
    handler=process,
    max_concurrent=20,  # Process 20 messages concurrently
)
await loop.run()
```

Open questions:
- Is the Settings class pattern the right approach? Considered decorators but this felt more explicit
- Should I add built-in DLQ support or keep it in error handlers?
- Thoughts on the predefined settings profiles?

GitHub: https://github.com/Aprova-GmbH/flowodm
Apache 2.0, Python 3.11+

Feedback welcome!

---

## r/madeinpython - Show & Tell Style

**Title:** Built FlowODM - Type-safe Kafka with Pydantic (no more boilerplate!)

Hey everyone! Just published my first Python library after months of using it internally.

**What it is:** An ODM for Kafka that lets you define messages as Pydantic models and handles all the boring stuff automatically.

Instead of writing manual serialization, schema management, and consumer loop boilerplate, you just do this:

```python
from flowodm import FlowBaseModel, ConsumerLoop

class OrderEvent(FlowBaseModel):
    class Settings:
        topic = "orders"

    order_id: str
    total: float

ConsumerLoop(
    model=OrderEvent,
    handler=lambda order: print(f"Got ${order.total}"),
).run()
```

That's literally it. It handles:
- Avro schema generation
- Schema Registry integration
- Graceful shutdown (SIGTERM/SIGINT)
- Error handling and retries
- Both sync and async APIs

Built it because I was tired of copy-pasting the same Kafka boilerplate across every microservice project.

**Stats:**
- 0.1.0 on PyPI (`pip install flowodm`)
- Apache 2.0 licensed
- Python 3.11+
- >95% test coverage
- Actually using it in production

GitHub: https://github.com/Aprova-GmbH/flowodm

Would love any feedback or suggestions!