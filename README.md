requests
| where timestamp > ago(3d)
| where cloud_RoleName == "fo-svc-response"
| summarize cnt = count() by operation_Id
| where cnt > 1