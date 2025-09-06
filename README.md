
# Kafka Producer Enhancements

This project extends a basic Kafka producer with two key features:

1. **Message Limit Option** – configure the maximum number of messages a producer will send.
2. **Structured Events (BuzzEvent)** – represent messages as structured JSON objects with metadata.

---

## 1. Message Limit Option

We added a helper function `get_message_limit` that lets you set an optional cap on the number of messages produced.  
If `MESSAGE_LIMIT` is not provided in your `.env`, the producer will run indefinitely.

```python
from typing import Optional

def get_message_limit() -> Optional[int]:
    """Fetch optional message cap from environment (None = infinite)."""
    raw = os.getenv("MESSAGE_LIMIT")
    if raw is None or raw.strip() == "":
        logger.info("Message limit: infinite")
        return None
    try:
        limit = int(raw)
        logger.info(f"Message limit: {limit}")
        return limit
    except ValueError:
        logger.warning("Invalid MESSAGE_LIMIT; using infinite stream")
        return None
````

### Example `.env` usage

```env
MESSAGE_LIMIT=25
```

This ensures the producer will stop after sending 25 messages.

---

## 2. Structured Events (BuzzEvent)

We introduced a `dataclass` to represent messages with consistent structure.
Instead of sending plain strings, the producer can now generate **JSON-encoded events**.

```python
from dataclasses import dataclass, asdict
from datetime import datetime, timezone
import json
from typing import List

@dataclass
class BuzzEvent:
    """Structured event describing a buzz message."""

    event_id: str
    ts: str
    source: str
    text: str
    tags: List[str]
    sentiment: float  # in [-1.0, 1.0]

    @staticmethod
    def now_iso() -> str:
        return datetime.now(timezone.utc).isoformat()

    def to_json_str(self) -> str:
        return json.dumps(asdict(self), ensure_ascii=False, separators=(",", ":"))
```

### Example JSON output

```json
{
  "event_id": "123e4567-e89b-12d3-a456-426614174000",
  "ts": "2025-09-05T20:15:30.123456+00:00",
  "source": "cli",
  "text": "Kafka is awesome!",
  "tags": ["kafka", "python", "streaming"],
  "sentiment": 0.8
}
```

---

## Summary

* `MESSAGE_LIMIT` gives you control over how many messages are sent.
* `BuzzEvent` structures each message for easier downstream parsing and analysis.

These enhancements make the Kafka producer **more flexible** and **better suited for real-world streaming pipelines**.

```

---