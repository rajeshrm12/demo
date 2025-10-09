let op = "130a44984e563dbf897e9b3305f5ef55";   // <- your opId
let svc = "fo-svc-business";
let ev  = "request-handler-no-fraud";
customEvents
| where cloud_RoleName == svc and operation_Id == op and name == ev
| extend cd = todynamic(customDimensions)
| project timestamp, itemId, name,
          attempt=tostring(cd.attempt),
          topic=tostring(cd.topic),
          status=tostring(cd.statusCode),
          error=tostring(cd.error)
| order by timestamp asc



let op = "130a44984e563dbf897e9b3305f5ef55";
let svc = "fo-svc-business";
let ev  = "request-handler-no-fraud";
customEvents
| where cloud_RoleName == svc and operation_Id == op and name == ev
| summarize n = count() by bin(timestamp, 1ms)
| order by timestamp asc



let svc = "fo-svc-business";
customEvents
| where timestamp > ago(3d) and cloud_RoleName == svc
| summarize cnt=count(), firstTime=min(timestamp), lastTime=max(timestamp)
          by operation_Id, name
| where cnt > 1
| extend gapUs = datetime_diff('microsecond', lastTime, firstTime)  // often 0 for AI
| order by lastTime desc




let op = "130a44984e563dbf897e9b3305f5ef55";
let svc = "fo-svc-business";
customEvents
| where cloud_RoleName == svc and operation_Id == op
| extend cd = todynamic(customDimensions), attempt=tostring(cd.attempt)
| where isnotempty(attempt)
| project timestamp, name, attempt
| order by timestamp asc





