let lookback = 7d;
let svcs = dynamic(["fo-svc-business","core-svc-m2m-transfer","fo-svc-response"]);

customEvents
| where timestamp > ago(lookback) and cloud_RoleName in (svcs)
| summarize cnt = count() by itemId, cloud_RoleName
| where cnt > 1
| order by cloud_RoleName, cnt desc



let lookback = 7d;
let iid = "replace_with_any_duplicate_itemId";  // e.g. "b70a047e-a2d8-11f0-8703-00224831596e"

customEvents
| where timestamp > ago(lookback) and itemId == iid
| project timestamp, ingestion = ingestion_time(), cloud_RoleName, cloud_RoleInstance, operation_Id
| order by ingestion asc




let lookback = 7d;
let iid = "replace_with_same_itemId";

customEvents
| where timestamp > ago(lookback) and itemId == iid
| project timestamp,
          ingestion = ingestion_time(),
          cloud_RoleName,
          cloud_RoleInstance,
          sdkVersion = tostring(customDimensions["ai.internal.sdkVersion"]),
          aiAgent    = tostring(customDimensions["ai.agent.version"])
| order by ingestion asc




let lookback = 7d;
let svcs = dynamic(["fo-svc-business","core-svc-m2m-transfer","fo-svc-response"]);

customEvents
| where timestamp > ago(lookback) and cloud_RoleName in (svcs)
| extend ingestionTime = ingestion_time()
| extend delaySec = datetime_diff('second', ingestionTime, timestamp)
| summarize cnt = count(), avgDelay = avg(delaySec)
  by cloud_RoleName,
     delayBucket = case(delaySec < 30, "<30s",
                        delaySec < 60, "30–60s",
                        delaySec < 120, "60–120s",
                        ">120s")
| order by cloud_RoleName, delayBucket





let lookback = 7d;
let iid = "PUT-ONE-OF-THOSE-itemId-HERE";
customEvents
| where timestamp > ago(lookback)
  and cloud_RoleName == "fo-svc-response"
  and itemId == iid
| project timestamp, ingestion = ingestion_time(), name, operation_Id, cloud_RoleInstance
| order by timestamp asc, ingestion asc





