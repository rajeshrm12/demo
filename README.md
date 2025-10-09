let lookback = 7d;
let svc = "fo-svc-business";
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == svc
| summarize cnt = count(), firstTime = min(timestamp), lastTime = max(timestamp)
          by operation_Id, name
| where cnt > 1
| order by lastTime desc


let lookback = 7d;
let svc = "fo-svc-business";
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == svc
| extend eventKey = tostring(customDimensions["eventKey"])
| where isnotempty(eventKey)
| summarize cnt = count(), names = make_set(name), firstTime = min(timestamp), lastTime = max(timestamp)
          by eventKey, operation_Id
| where cnt > 1
| top 50 by lastTime desc