let lookback = 3d;
let svc = "fo-svc-business";   // or "core-svc-m2m-transfer" / "fo-svc-response"

customEvents
| where timestamp > ago(lookback) and cloud_RoleName == svc
| summarize cnt = count(), firstTime = min(timestamp), lastTime = max(timestamp)
          by operation_Id, name
| where cnt > 1
| extend gapMs = datetime_diff('millisecond', lastTime, firstTime)
| order by lastTime desc


let lookback = 3d;
let svc = "fo-svc-business";

// pick newest duplicated operation
let opId =
toscalar(
  customEvents
  | where timestamp > ago(lookback) and cloud_RoleName == svc
  | summarize cnt=count(), lastTs=max(timestamp) by operation_Id, name
  | where cnt > 1
  | top 1 by lastTs desc
  | project operation_Id
);

// show the ordered timeline for that op
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == svc and operation_Id == opId
| extend cd = todynamic(customDimensions)
| project timestamp, name,
          eventKey   = tostring(cd.eventKey),
          topic      = tostring(cd.topic),
          partition  = tostring(cd.partition),
          offset     = tostring(cd.offset)
| order by timestamp asc