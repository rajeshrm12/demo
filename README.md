let svc = "fo-svc-response";   // ← change per service
let lookback = 2d;
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == svc
| extend topic     = tostring(customDimensions["topic"]),
         partition = toint(tostring(customDimensions["partition"])),
         offset    = tolong(tostring(customDimensions["offset"])),
         eventKey  = tostring(coalesce(customDimensions["eventKey"], customDimensions["key"], customDimensions["correlationId"]))
| where isnotempty(topic) and isnotempty(offset)
| summarize cnt=count(), first=min(timestamp), last=max(timestamp)
          by topic, partition, offset, eventKey
| where cnt == 2
| order by last desc
