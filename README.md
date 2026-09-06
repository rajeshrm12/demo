Goal: Build a config-driven, reusable test automation framework for Guardian fraud message validation, extensible across all message types without duplicating code per type.

Background/Flow:
Guardian’s business service exposes one endpoint (/guardian/business/api/events) that receives fraud-check requests for all message types (M2M Transfer, Add Payee, Step Up — already in production; Self-Service Wires, MSR Wires, Treasury Wires, Treasury ACHO, Global Transfer, etc. — new, currently manual). Requests arrive via MuleSoft, not directly.

For each request:

	1.	Business service validates mandatory fields and conditional field rules (e.g. if recipientType=P, then recipientFirstName is required). Missing/invalid fields → HTTP 400 with a specific error message, and the message is never pushed to Kafka.
	2.	If valid, business service builds an event ID (concatenation of 4 fields) and publishes to a Kafka topic specific to that message type.
	3.	A per-message-type consumer service picks up the message, makes ~9-10 enrichment API calls (e.g. member inquiry, recipient verification) using data from the original request, and assembles a payload for ARIC (the fraud decision engine).
	4.	Before sending to ARIC, the consumer service renames some fields (Kafka/request field names differ from what ARIC expects) — this mapping must be captured and validated separately per message type.
	5.	ARIC returns a decision (fraud / no fraud / step up / manual review hold) back through the service.
	6.	As testers, we can’t observe internal Kafka/consumer processing directly, but we can verify correctness by: (a) calling the same enrichment APIs independently (like manual testers currently do in Postman) using the account number/fields from the original request, and (b) logging into the ARIC UI to confirm the fields displayed there match the schema/mapping document.

Current scope (in priority order):

	•	Mandatory/optional/conditional field validation at the Guardian request stage (positive + negative cases)
	•	Enrichment API validation (calling each enrichment endpoint independently and asserting expected response)
	•	ARIC UI field-by-field validation against schema doc, with field-mapping/renaming rules applied
	•	Out of scope for now: Kafka topic inspection, database validation, Azure App Insights/log tracing

Architecture requirement:
Instead of writing separate step definitions/logic per message type, build a config-driven engine: one JSON/YAML config per message type defining its endpoint, mandatory/optional fields, conditional rules (with expected error messages), enrichment API list (with source field → request param mapping), and ARIC field-name mapping. Generic, reusable step definitions (RequestBuilder, FieldValidator, EnrichmentValidator, AricComparator) read from whichever config is loaded for the scenario. Onboarding a new message type should mean writing a new config file, not new code.