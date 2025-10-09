 let op = "<PASTE operation_Id>";
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