let lookback = 3d;
let svcs = dynamic(["fo-svc-business","core-svc-m2m-transfer","fo-svc-response"]);

customEvents
| where timestamp > ago(lookback) and cloud_RoleName in (svcs)
| extend ing = ingestion_time()
| summarize
    rows      = count(),
    ingCnt    = dcount(bin(ing, 1s)),
    firstTs   = min(timestamp),
    lastTs    = max(timestamp)
  by cloud_RoleName, operation_Id, itemId, name
| where rows > 1 and firstTs == lastTs and ingCnt > 1   // same event logged once, ingested >1
| summarize dupOps = dcount(operation_Id),
            dupItems = count(),
            dupEvents = sum(rows),
            lastDup = max(lastTs)
  by cloud_RoleName
| order by lastDup desc





let lookback = 3d;
let svcs = dynamic(["fo-svc-business","core-svc-m2m-transfer","fo-svc-response"]);

// find the duplicated itemIds first
let dups =
customEvents
| where timestamp > ago(lookback) and cloud_RoleName in (svcs)
| extend ing = ingestion_time()
| summarize rows = count(), ingCnt = dcount(bin(ing, 1s)), firstTs = min(timestamp), lastTs = max(timestamp)
  by cloud_RoleName, operation_Id, itemId, name
| where rows > 1 and firstTs == lastTs and ingCnt > 1
| top 50 by lastTs desc;   // change the 50 if you want more/less

// show every duplicate row with diagnostics
customEvents
| where itemId in (dups | project itemId)
| extend ing = ingestion_time(),
         sdkVersion = tostring(customDimensions["ai.internal.sdkVersion"]),
         aiAgent    = tostring(customDimensions["ai.agent.version"])
| project timestamp, ing, cloud_RoleName, cloud_RoleInstance,
          operation_Id, name, itemId, sdkVersion, aiAgent
| order by itemId asc, ing asc



let lookback = 3d;
let svcs = dynamic(["fo-svc-business","core-svc-m2m-transfer","fo-svc-response"]);

customEvents
| where timestamp > ago(lookback) and cloud_RoleName in (svcs)
| extend ing = ingestion_time(),
         sdkVersion = tostring(customDimensions["ai.internal.sdkVersion"]),
         aiAgent = tostring(customDimensions["ai.agent.version"]),
         delaySec = datetime_diff('second', ing, timestamp)
| summarize cnt = count(), ingCnt = dcount(bin(ing,1s)),
            minDelay = min(delaySec), maxDelay = max(delaySec),
            firstIngest = min(ing), lastIngest = max(ing)
  by cloud_RoleName, itemId, name, sdkVersion, aiAgent
| where cnt > 1 and ingCnt > 1
| order by lastIngest desc