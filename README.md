Understood, here's the full context in one block, ready to copy.

Context for automation framework. Message types in scope right now, member to member transfer and global transfer. Guardian fraud detection system at NFCU. Current phase excludes digital banking UI login, database insertion checks, and journey log checks, those come later.

Step one. Send a request directly to the Guardian business service endpoint, equivalent to what's currently done manually in Postman, using a full valid payload for the given message type. Validate that all mandatory and optional fields specified in the schema document are present and correctly formatted in the outgoing request.

Step two. Verify the message lands correctly on the appropriate Kafka topic for that message type. Check that all attributes in the Kafka message match what was sent in the original request, and that the event identifier was constructed correctly.

Step three. The consumer service for that message type triggers a series of individual backend enrichment calls, roughly seven of them, such as a member inquiry call that takes an account number and returns associated member details. Each of these individual API calls needs to be validated independently, checking that given the correct input, the response contains all expected fields correctly populated.

Step four. Navigate to the ARIC UI, also called feature space, log in, search for the specific event, and verify that all fields and attributes displayed in the ARIC interface match the expected values from the original request and enrichment responses. Capture a screenshot at this point for the test report.

Step five. Verify the final decision outcome shown in ARIC or returned in the response, which will be one of, fraud, no fraud, manual hold review, or step up, and assert it matches the expected outcome for that test case.

Existing framework note. There is already a working flow built that calls NFCU APIs via REST Assured, stores the response JSON, opens ARIC in Chrome via Selenium, performs SSO login, selects the tenant, searches for the event, retrieves ARIC's own internal JSON representation of that event, does a field by field comparison between the API response and ARIC's data, and writes pass or fail results to an Excel report with a screenshot. New test cases should be mapped onto this existing flow rather than building a new one from scratch.