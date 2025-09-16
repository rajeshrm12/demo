let start = datetime(2025-09-15 06:00:00);
let end   = datetime(2025-09-15 14:00:00);
requests
| where timestamp between (start .. end)
| where name == "fraud-rtfds-guardian-response process"
| summarize cnt = count() by bin(timestamp, 1m)
| serialize
| extend start_of_run = iif(cnt > 0 and prev(cnt, 1) == 0, 1, 0)
| extend run_id = row_cumsum(start_of_run)
| where cnt > 0
| summarize RunStart = min(timestamp), RunEnd = max(timestamp), TotalEvents = sum(cnt) by run_id
| order by RunStart asc
| take 5
