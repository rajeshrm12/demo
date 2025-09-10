let start=datetime(2025-09-02T09:00:00-04:00);
let end=datetime(2025-09-02T11:59:00-04:00);
let serviceName="fraud-rtfds-response process";

requests
| where timestamp between (start .. end)
| where name == serviceName
| order by timestamp asc
| extend consumer_lag = toint(customMeasurements["timeSinceEnqueued"])
| project timestamp, consumer_lag
| where consumer_lag > 100
