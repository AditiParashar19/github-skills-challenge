# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

---
# AIOps Monitoring Assessment

## Scenario

This project monitors a simulated payment service. The goal is to detect slow
responses, high CPU usage, high memory usage, and error-level log events.

The AIOps workflow identifies unusual behavior and sends anomaly events
through a lightweight in-memory event pipeline instead of external Kafka or
Airflow infrastructure.

## Repository Components

- `data/service_data.json`: synthetic payment-service telemetry.
- `src/anomaly_detector.py`: detects threshold and error-log anomalies.
- `src/event_producer.py`: publishes events to a topic.
- `src/event_topic.py`: stores events in memory.
- `src/event_consumer.py`: reads events from the topic.
- `src/aiops_pipeline.py`: runs the complete workflow and prints the output.
- `tests/`: validates detection and event processing.

## Operational Data

The data is stored in `data/service_data.json`.

Metric fields:

- `response_time_ms`
- `cpu_percent`
- `memory_percent`

Log fields:

- `log_level`
- `message`

The `timestamp` field identifies when each observation occurred and allows the
records to be reviewed chronologically.

Most observations show normal payment-service behavior. The records at
`10:05:00` and `10:06:00` are anomalous.

Normal records have response times from 120 to 150 ms, CPU usage from 42 to
50 percent, memory usage from 51 to 57 percent, and `INFO` log levels. The
two anomalous records have `ERROR` logs and response times above 500 ms. The
record at `10:06:00` also exceeds the CPU and memory thresholds.

## Anomaly Findings

The detector uses these thresholds:

- Response time: greater than 500 ms
- CPU usage: greater than 80 percent
- Memory usage: greater than 80 percent
- Log level `ERROR`

Detected anomalies:

- `10:05:00`: high response time and error log
- `10:06:00`: high response time, high CPU, high memory, and error log

No expected anomaly was missed in the supplied data, and no normal record was
incorrectly flagged.

## Event Processing Flow

The workflow is:

Operational data -> Anomaly detector -> Event -> Producer ->
Anomaly topic -> Consumer -> Final AIOps output

The producer publishes detected anomaly events to the shared
`anomaly-events` topic. The consumer reads events from that same topic.

Component roles:

- **Event:** contains the anomaly type, timestamp, service, reasons, and
	original source record.
- **Producer:** sends each generated event to the topic.
- **Topic:** provides in-memory event storage and retrieval.
- **Consumer:** receives events for downstream AIOps processing.

## Issues Corrected

The pipeline originally created separate topics for the producer and consumer.
As a result, the producer published events to one topic while the consumer
read from an empty topic.

The correction was to use one shared `EventTopic` instance for both the
`EventProducer` and `EventConsumer`.

## Final Execution Result

The pipeline processed 10 records and detected 2 anomalies.

The consumer successfully received 2 anomaly events.

The final output was:

```text
Records processed: 10
Anomalies detected: 2
Events consumed: 2
```

## Validation

The test suite validates normal detection, anomaly detection, producer and
consumer behavior, empty-event rejection, and complete pipeline processing.

```text
10 passed
```

## Limitation

The detector uses fixed thresholds and does not learn normal behavior from
historical data. A possible improvement would be dynamic thresholds based on
historical averages or statistical anomaly detection.

## Reproduction

From the repository root, run:

```bash
python -m pytest -q
PYTHONPATH=src python src/aiops_pipeline.py
```

To verify the producer, topic, and consumer independently:

```bash
PYTHONPATH=src python - <<'PY'
from event_producer import EventProducer
from event_consumer import EventConsumer
from event_topic import EventTopic

topic = EventTopic("anomaly-events")
producer = EventProducer(topic)
consumer = EventConsumer(topic)

event = {"type": "ANOMALY", "service": "payment-service"}

print("Published:", producer.publish(event))
print("Topic messages:", topic.get_messages())
print("Consumed messages:", consumer.consume())
PY
```

## Evidence Checklist

Capture terminal screenshots showing:

1. The telemetry data and metric/log fields.
2. The anomaly detection output and reasons.
3. Event publication to the anomaly topic.
4. Consumer receipt of the events.
5. Final AIOps output.
6. Successful test execution showing `10 passed`.

## Submission Checklist

- Commit all relevant changes.
- Push the branch to the GitHub fork.
- Create a pull request from the fork to the original repository.
- Include the workflow findings, validation result, corrected issue, and
	limitation in the pull request description.pull request create failed: GraphQL: Head sha can't be blank, Base sha can't beblank, No commits between DebbieAUG:main and AditiParashar19:assessment/aiops-workflow, Head ref must be a branch (createPullRequest