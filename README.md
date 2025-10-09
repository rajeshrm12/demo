let lookback = 3d;
let iid = "bfc81954-a2d8-11f0-8706-7c1e525a09da"; // replace with your itemId
customEvents
| where timestamp > ago(lookback)
| where itemId == iid
| project 
    timestamp,
    ingestion = ingestion_time(),
    cloud_RoleName,
    cloud_RoleInstance,
    operation_Id,
    sdkVersion = tostring(customDimensions["ai.internal.sdkVersion"]),
    aiAgent = tostring(customDimensions["ai.agent.version"]),
    appVersion = tostring(customDimensions["app_Version"]),
    iKey = tostring(customDimensions["ai.internal.nodeName"]),
    _ResourceId
| order by ingestion asc



let lookback = 3d;
let iid = "REPLACE_WITH_ONE_DUP_ITEMID";

customEvents
| where timestamp > ago(lookback) and itemId == iid
| project
    timestamp,
    ingestion = ingestion_time(),
    cloud_RoleName,
    cloud_RoleInstance,
    operation_Id,
    sdkVersion = tostring(customDimensions["ai.internal.sdkVersion"]),
    aiAgent    = tostring(customDimensions["ai.agent.version"]),
    appVersion = tostring(customDimensions["app_Version"]),
    resourceId = tostring(column_ifexists("_ResourceId", column_ifexists("resourceId",""))),
    subscriptionId = tostring(column_ifexists("_SubscriptionId",""))
| order by ingestion asc






let lookback = 3d;
let iid = "b70a047e-a2d8-11f0-8703-00224831596e"; // your itemId
customEvents
| where timestamp > ago(lookback) and itemId == iid
| project
    timestamp,
    ingestion = ingestion_time(),
    cloud_RoleName,
    cloud_RoleInstance,
    sdkVersion = tostring(customDimensions["ai.internal.sdkVersion"]),
    aiAgent = tostring(customDimensions["ai.agent.version"]),
    exporterType = tostring(customDimensions["otel.exporter"]),
    resourceId = tostring(column_ifexists("_ResourceId", column_ifexists("resourceId","")))
| order by ingestion asc
