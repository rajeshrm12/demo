let lookback = 7d;
customEvents
| where timestamp > ago(lookback)
| where cloud_RoleName == "fo-svc-business"
| where name in ("request-handler-acknowledgement-to-source-system", "request-handler-event-stream-write")
| extend opId = tostring(operation_Id)
| summarize cnt=count(), first=min(timestamp), last=max(timestamp) by name, opId
| where cnt > 1
| order by last desc