requests
| where timestamp between (todatetime("2025-09-02T09:00:00-04:00") .. todatetime("2025-09-02T11:59:00-04:00"))
| where cloud_RoleName == "fo-svc-response"
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
