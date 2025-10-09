let op = "<paste one opId from your table>";
customEvents
| where operation_Id == op and cloud_RoleName == "fo-svc-business"
| summarize cnt = count() by name
| order by cnt desc


let op = "<same opId>";
customEvents
| where operation_Id == op
| where name == "request-handler-event-stream-write"
| project timestamp,
          url = tostring(customDimensions["url"]),
          topic = tostring(customDimensions["topic"]),
          key = tostring(customDimensions["eventKey"]),
          attempt = tostring(customDimensions["attempt"]),
          status = tostring(customDimensions["statusCode"]),
          error = tostring(customDimensions["error"]),
          detail = tostring(message),
          dims = customDimensions
| order by timestamp asc