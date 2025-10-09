let lookback = 7d;
let svc = "fo-svc-business";
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == svc
| summarize cnt = count(), firstTime = min(timestamp), lastTime = max(timestamp)
          by operation_Id, name
| where cnt > 1
| order by lastTime desc