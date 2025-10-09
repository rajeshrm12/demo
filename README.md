requests
| where timestamp > ago(3d)
| where cloud_RoleName == "fo-svc-response"
| summarize cnt = count() by operation_Id
| where cnt > 1


customEvents
| where timestamp > ago(7d)
| where cloud_RoleName == "fo-svc-business"
| summarize cnt = count(), firstTime = min(timestamp), lastTime = max(timestamp)
        by operation_Id, name
| where cnt > 1
| order by lastTime desc