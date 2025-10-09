let lookback = 3d;
let iid = "bfc81954-a2d8-11f0-8706-7c1e525a09da";
customEvents
| where timestamp > ago(lookback)
| where itemId == iid
| project timestamp, ingestion=ingestion_time(), cloud_RoleName, cloud_RoleInstance,
         name, operation_Id, app_Version,
         sdkVersion=tostring(customDimensions["ai.internal.sdkVersion"]),
         aiAgent=tostring(customDimensions["ai.agent.version"]),
         iKey=tostring(customDimensions["ai.internal.nodeName"]), // often blank; keep for clues
         _ResourceId
| order by ingestion asc