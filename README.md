let lookback = 3d;
let svcs = dynamic(["fo-svc-business","core-svc-m2m-transfer","fo-svc-response"]);

customEvents
| where timestamp > ago(lookback) and cloud_RoleName in (svcs)
| extend ingestionTime = ingestion_time()
| extend delaySec = datetime_diff('second', ingestionTime, timestamp)
| summarize cnt=count(), avgDelay=avg(delaySec)
  by cloud_RoleName,
     delayBucket = case(delaySec < 30, "<30s",
                        delaySec < 60, "30–60s",
                        delaySec < 120, "60–120s",
                        ">120s")
| order by cloud_RoleName, delayBucket