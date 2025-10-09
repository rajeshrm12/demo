u let op = "<PASTE operation_Id>";
union requests, dependencies, traces, customEvents
| where operation_Id == op
| project timestamp, _table, name, resultCode, success, message, customDimensions
| order by timestamp asc



let lookback = 3d;
requests
| where timestamp > ago(lookback)
| where name == "fraud-rtfds-guardian-response process"
| summarize cnt=count(), first=min(timestamp), last=max(timestamp) by operation_Id
| where cnt > 1
| order by last desc


let lookback = 3d;
requests
| where timestamp > ago(lookback)
| where name == "fraud-rtfds-m2m-transfer process"
| summarize cnt=count(), first=min(timestamp), last=max(timestamp) by operation_Id
| where cnt > 1
| order by last desc



let lookback = 3d;
requests
| where timestamp > ago(lookback)
| where name == "fraud-rtfds-guardian-response process"
| summarize count() by operation_Id
| where count_ > 1
| order by count_ desc


let lookback = 3d;
requests
| where timestamp > ago(lookback)
| where name == "fraud-rtfds-m2m-transfer process"
| summarize count() by operation_Id
| where count_ > 1
| order by count_ desc



let op = "<paste operation_Id>";
union requests, dependencies, traces, customEvents
| where operation_Id == op
| project timestamp, cloud_RoleName, name, resultCode, success, message
| order by timestamp asc



requests
| where timestamp > ago(3d)
| summarize count() by cloud_RoleName
| order by count_ desc



let lookback = 3d;
let svc = "fo-svc-response";

// keys with exactly two HTTP calls
let http2 =
dependencies
| where timestamp > ago(lookback) and cloud_RoleName == svc and type =~ "Http"
| extend key = tostring(customDimensions["eventKey"])
| where isnotempty(key)
| summarize httpCnt = count() by key
| where httpCnt == 2;

// how many times those keys were processed/logged in service
let kcnt =
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == svc
| extend key = tostring(customDimensions["eventKey"])
| where isnotempty(key)
| summarize kafkaCnt = count() by key;

http2
| join kind=leftouter kcnt on key
| project key, httpCnt, kafkaCnt
| order by kafkaCnt desc