let svc = "fo-svc-business";

// auto-pick the latest duplicated operation
let op =
    toscalar(
        customEvents
        | where timestamp > ago(7d) and cloud_RoleName == svc
        | summarize cnt = count(), lastTs = max(timestamp) by operation_Id
        | where cnt > 1
        | top 1 by lastTs desc
        | project operation_Id
    );

// show every event in that operation, ordered by time
customEvents
| where timestamp > ago(7d)
| where operation_Id == op and cloud_RoleName == svc
| extend attempt = tostring(customDimensions["attempt"])
| project timestamp, name, attempt, message
| order by timestamp asc