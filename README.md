let lookback = 3d;
let svcs = dynamic(["fo-svc-business","core-svc-m2m-transfer","fo-svc-response"]);

customEvents
| where timestamp > ago(lookback) and cloud_RoleName in (svcs)
| extend ingestionTime = ingestion_time()
| extend delaySec = datetime_diff('second', ingestionTime, timestamp),
         sdkVersion = tostring(customDimensions["ai.internal.sdkVersion"]),
         aiAgent = tostring(customDimensions["ai.agent.version"])
| summarize cnt = count(),
            ingCnt = dcount(bin(ingestionTime,1s)),
            minDelay = min(delaySec),
            maxDelay = max(delaySec),
            firstIngest = min(ingestionTime),
            lastIngest = max(ingestionTime)
  by cloud_RoleName, itemId, name, sdkVersion, aiAgent
| where cnt > 1 and ingCnt > 1
| order by lastIngest desc