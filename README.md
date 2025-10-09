let lookback = 3d;
let svc = "fo-svc-business";  // change to "core-svc-m2m-transfer" then "fo-svc-response"
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == svc
| summarize cnt = count(), firstTime = min(timestamp), lastTime = max(timestamp)
          by operation_Id, name
| where cnt > 1
| top 30 by lastTime desc


let lookback = 3d;
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == "fo-svc-response"
| extend topic=tostring(customDimensions["topic"]),
         partition=toint(tostring(customDimensions["partition"])),
         offset=tolong(tostring(customDimensions["offset"]))
| where isnotempty(topic) and isnotempty(offset)
| summarize cnt=count() by topic, partition, offset
| where cnt > 1



let lookback = 3d;

// Business keys duplicated
let biz =
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == "fo-svc-business"
| extend key=tostring(customDimensions["eventKey"])
| where isnotempty(key)
| summarize bizCnt=count(), lastB=max(timestamp) by key
| where bizCnt > 1;

// Do those same keys duplicate in Transfer?
let trn =
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == "core-svc-m2m-transfer"
| extend key=tostring(customDimensions["eventKey"])
| where isnotempty(key)
| summarize trnCnt=count(), lastT=max(timestamp) by key;

biz
| join kind=leftouter trn on key
| top 30 by lastB desc





let lookback = 3d;   // last 3 days
let svc = "fo-svc-business";   // change to core-svc-m2m-transfer or fo-svc-response
customEvents
| where timestamp > ago(lookback) 
| where cloud_RoleName == svc
| summarize cnt = count() by operation_Id, name
| where cnt > 1
| summarize totalDuplicateOperations = count(), totalDuplicateEvents = sum(cnt)
