 let op = "<PASTE operation_Id>";
union requests, dependencies, traces, customEvents
| where operation_Id == op
| project timestamp, _table, name, resultCode, success, message, customDimensions
| order by timestamp asc