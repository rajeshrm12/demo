// auto-pick newest op where ANY handler duplicated
let svc = "fo-svc-business";
let op =
  toscalar(
    customEvents
    | where timestamp > ago(7d) and cloud_RoleName == svc
    | summarize cnt=count() by operation_Id, name
    | where cnt > 1
    | summarize lastTime=max(timestamp) by operation_Id
    | top 1 by lastTime desc
    | project operation_Id
  );

customEvents
| where operation_Id == op and cloud_RoleName == "fo-svc-business"
| extend attempt = tostring(customDimensions["attempt"])
| project timestamp, name, attempt, message
| order by timestamp asc