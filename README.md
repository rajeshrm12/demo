let lookback = 7d;
let svcs = dynamic(["fo-svc-business", "core-svc-m2m-transfer", "fo-svc-response"]);
customEvents
| where timestamp > ago(lookback) and cloud_RoleName in (svcs)
| summarize cnt = count() by itemId, cloud_RoleName
| where cnt > 1
| order by cloud_RoleName, cnt desc


let lookback = 7d;
let iid = "replace_with_duplicate_itemId";
customEvents
| where timestamp > ago(lookback) and itemId == iid
| project timestamp, ingestion = ingestion_time(), name, operation_Id, cloud_RoleInstance
| order by timestamp asc, ingestion asc


