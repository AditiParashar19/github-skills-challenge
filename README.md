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

## Operational Data

The data is stored in `data/service_data.json`.

Metric fields:

- `response_time_ms`
- `cpu_percent`
- `memory_percent`

Log fields:

- `log_level`
- `message`

The `timestamp` field identifies when each observation occurred.

Most observations show normal payment-service behavior. The records at
`10:05:00` and `10:06:00` are anomalous.

## Anomaly Findings

The detector uses these thresholds:

- Response time: greater than 500 ms
- CPU usage: greater than 80 percent
- Memory usage: greater than 80 percent
- Log level `ERROR`

Detected anomalies:

- `10:05:00`: high response time and error log
- `10:06:00`: high response time, high CPU, high memory, and error log

## Event Processing Flow

The workflow is:

Operational data -> Anomaly detector -> Event -> Producer ->
Anomaly topic -> Consumer -> Final AIOps output

The producer publishes detected anomaly events to the shared
`anomaly-events` topic. The consumer reads events from that same topic.

## Issues Corrected

The pipeline originally created separate topics for the producer and consumer.
As a result, the producer published events to one topic while the consumer
read from an empty topic.

The correction was to use one shared `EventTopic` instance for both the
`EventProducer` and `EventConsumer`.

## Final Execution Result

The pipeline processed 10 records and detected 2 anomalies.

The consumer successfully received 2 anomaly events.

## Limitation

The detector uses fixed thresholds and does not learn normal behavior from
historical data. A possible improvement would be dynamic thresholds based on
historical averages or statistical anomaly detection.

## Reproduction

Run:

```bash
python -m pytest -q
PYTHONPATH=src python aiops_pipeline.py