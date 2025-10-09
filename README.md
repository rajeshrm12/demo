let lookback = 3d;
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == "fo-svc-business"
| summarize c=count(), first=min(timestamp), last=max(timestamp) by itemId
| where c > 1