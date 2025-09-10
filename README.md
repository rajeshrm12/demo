let startTime = todatetime("2025-09-02T09:00:00-04:00");
let endTime   = todatetime("2025-09-02T11:59:00-04:00");
let serviceName = "fraud-rtfds-response process";  // change to transfer when needed

requests
| where timestamp between (startTime .. endTime)
| where name == serviceName
| extend consumer_lag = toint(customMeasurements["timeSinceEnqueued"])
| where isnotempty(consumer_lag)
| summarize
    total = count(),
    gt100  = countif(consumer_lag > 100),
    gt200  = countif(consumer_lag > 200),
    gt300  = countif(consumer_lag > 300),
    gt400  = countif(consumer_lag > 400),
    gt500  = countif(consumer_lag > 500),
    gt750  = countif(consumer_lag > 750),
    gt1000 = countif(consumer_lag > 1000),
    gt1500 = countif(consumer_lag > 1500),
    gt2000 = countif(consumer_lag > 2000),
    p95    = percentile(consumer_lag, 95),
    p99    = percentile(consumer_lag, 99)
| project
    total,
    ["# >100ms"]  = gt100,  ["% >100ms"]  = todouble(gt100 ) * 100.0 / total,
    ["# >200ms"]  = gt200,  ["% >200ms"]  = todouble(gt200 ) * 100.0 / total,
    ["# >300ms"]  = gt300,  ["% >300ms"]  = todouble(gt300 ) * 100.0 / total,
    ["# >400ms"]  = gt400,  ["% >400ms"]  = todouble(gt400 ) * 100.0 / total,
    ["# >500ms"]  = gt500,  ["% >500ms"]  = todouble(gt500 ) * 100.0 / total,
    ["# >750ms"]  = gt750,  ["% >750ms"]  = todouble(gt750 ) * 100.0 / total,
    ["# >1000ms"] = gt1000, ["% >1000ms"] = todouble(gt1000) * 100.0 / total,
    ["# >1500ms"] = gt1500, ["% >1500ms"] = todouble(gt1500) * 100.0 / total,
    ["# >2000ms"] = gt2000, ["% >2000ms"] = todouble(gt2000) * 100.0 / total,
    ["p95 (ms)"]  = p95,
    ["p99 (ms)"]  = p99
