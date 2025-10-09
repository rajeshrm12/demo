Duplicate Delivery Investigation Summary (PERF-EUS AppInsights)

Objective:
Investigating duplicate event deliveries across Guardian services (fo-svc-business, core-svc-m2m-transfer, and fo-svc-response) to determine whether duplication is happening at the HTTP layer, Kafka layer, or due to retry logic.

⸻

Findings so far:
	•	Response Service
	•	No duplicate consumption found (requests table shows 1 per operation_Id).
	•	Confirms Kafka consumer is not duplicating.
	•	✅ Only one consumption per event.
	•	Business Service
	•	Duplicate logs found for:
	•	request-handler-event-stream-write
	•	request-handler-event-received
	•	request-handler-no-fraud
	•	Each appears twice within the same operation_Id, milliseconds apart.
	•	Indicates the same handler method executes twice in a single request.
	•	⚠️ Suggests retry logic (Spring Retry / Resilience4j) is wrapping the full handler instead of just the outbound call.
	•	Transfer Service
	•	Similar pattern observed (possible duplicate POSTs).
	•	But Response consumes only once → Kafka idempotence works and no downstream duplication.

⸻

Root Cause (current hypothesis):
A retry mechanism at the handler level (instead of at the outbound call) causes the full Business flow to re-run when transient errors occur.
Producer idempotence prevents duplicate Kafka messages, so duplicates are visible only in logs, not in Response consumption.

⸻

Queries used (AppInsights KQL
customEvents
| where timestamp > ago(20d)
| where cloud_RoleName == "fo-svc-business"
| summarize cnt=count(), first=min(timestamp), last=max(timestamp) by operation_Id, name
| where cnt > 1
| order by last desc
and drill-downs on the same operation_Id to confirm that event sequences repeat twice.

⸻

Next Steps:
	1.	Confirm retry configuration in Business (Spring Retry / Resilience4j).
	2.	Narrow retry scope to only outbound calls (HTTP / Kafka send).
	3.	Add idempotency guard using a stable eventKey or short-TTL cache.
	4.	Include attempt#, durationMs, and eventKey in custom event logs.
	5.	Re-validate in PERF after the fix — each handler should log only once per operation.

⸻

Summary:
Duplicate logs in Business and Transfer come from handler-level retries, not Kafka or POST duplication.
Kafka producers and consumers behave idempotently, so only one real event reaches Response.
