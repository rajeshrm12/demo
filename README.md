let start = datetime("2025-09-15T06:20:00-04:00");
let end = datetime("2025-09-15T07:40:00-04:00");

requests
| where timestamp between (start .. end)
| where name == "fraud-rtfds-m2m-transfer process"
| order by timestamp asc
| extend consumer_lag = toint(customMeasurements["timeSinceEnqueued"])
| project timestamp, consumer_lag
| summarize 
    total_events = count(),
    gt_100 = countif(consumer_lag > 100),
    gt_200 = countif(consumer_lag > 200),
    gt_300 = countif(consumer_lag > 300),
    gt_400 = countif(consumer_lag > 400),
    gt_500 = countif(consumer_lag > 500),
    gt_750 = countif(consumer_lag > 750),
    gt_1000 = countif(consumer_lag > 1000),
    gt_1500 = countif(consumer_lag > 1500),
    gt_2000 = countif(consumer_lag > 2000)
