let lookback = 3d;
let svc = "fo-svc-business";  // <— adjust if different

// --- Normalize Kafka processing events
let kafka =
customEvents
| where timestamp > ago(lookback)
| where cloud_RoleName == svc
| where name in~ ("KafkaMessageProcessed","KafkaMessageReceived","MessageHandled")
      or (customDimensions contains "offset" and customDimensions contains "partition" and customDimensions contains "topic")
| extend eventKey = tostring(coalesce(customDimensions.eventKey, customDimensions.businessKey, customDimensions.key, customDimensions.correlationId)),
         topic = tostring(customDimensions.topic),
         partition = toint(customDimensions.partition),
         offset = tolong(customDimensions.offset),
         consumerGroup = tostring(coalesce(customDimensions.consumerGroup, customDimensions.groupId)),
         systemId = tostring(customDimensions.systemId),
         operation = tostring(coalesce(customDimensions.operation, customDimensions.eventAction, customDimensions.action)),
         sessionId = tostring(customDimensions.digitalSessionId)
| where isnotempty(eventKey) and isnotempty(topic);

// --- Normalize outbound HTTP attempts (events + dependencies)
let http_events =
customEvents
| where timestamp > ago(lookback) and cloud_RoleName == svc
| where name in~ ("HttpOutbound","DownstreamPost","ARICPost","ForgeRockPost") or customDimensions has "url"
| extend eventKey = tostring(coalesce(customDimensions.eventKey, customDimensions.key, customDimensions.correlationId)),
         url = tostring(coalesce(customDimensions.url, customDimensions.uri, name)),
         status = tostring(coalesce(customDimensions.statusCode, customDimensions.status)),
         attempt = toint(customDimensions.attempt),
         dest = tostring(coalesce(customDimensions.destination, customDimensions.target));
let http_deps =
dependencies
| where timestamp > ago(lookback) and cloud_RoleName == svc
| where type == "Http" or resultCode !=""
| extend eventKey = tostring(customDimensions.eventKey),
         url = tostring(name),
         status = tostring(resultCode),
         dest = tostring(target);
let http = union http_events, http_deps
| where isnotempty(eventKey);

// --- A) Consumer redelivery (same offset twice)
kafka
| summarize cnt=count(), offs=make_set(offset), first=min(timestamp), last=max(timestamp)
          by eventKey, topic, partition, consumerGroup, systemId, operation, sessionId
| where cnt == 2 and array_length(offs) == 1
| project bucket="A_consumer_redelivery", eventKey, topic, partition, consumerGroup, cnt, offs, first, last, deltaSec=datetime_diff('second', last, first), systemId, operation, sessionId
| order by last desc
