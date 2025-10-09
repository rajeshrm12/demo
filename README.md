let lookback = 3d;
let iid = "b70a047e-a2d8-11f0-8703-00224831596e"; // replace with your itemId
customEvents
| where timestamp > ago(lookback) and itemId == iid
| project
    timestamp,
    ingestion = ingestion_time(),
    cloud_RoleName,
    cloud_RoleInstance,
    sdkVersion = tostring(customDimensions["ai.internal.sdkVersion"]),
    aiAgent = tostring(customDimensions["ai.agent.version"]),
    resourceId
| order by ingestion asc



let iid = "b70a047e-a2d8-11f0-8703-00224831596e";
customEvents
| where itemId == iid
| project timestamp, ingestion = ingestion_time(), cloud_RoleInstance,
          sdkVersion = tostring(customDimensions["ai.internal.sdkVersion"]),
          aiAgent = tostring(customDimensions["ai.agent.version"])
| order by ingestion asc