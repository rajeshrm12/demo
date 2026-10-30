Yes. If your team doesn't add validation in services, safe error logging alone fully fixes the reported issue, because the problem is the leak, not the failure. Oversized requests will still fail (Notes returns 400, the DB insert fails) as they do today, but the access number won't appear in logs or alerts anymore.

Mention to Paresh that validation is optional. Without it, oversized requests still make the downstream call and attempt the DB save before failing, which is how your other services behave.

How to do safe error logging

1. Notes service error log
Search for "Error Member Account Notes service":

log.error("Member Account Notes call failed: event_id={}, status={}, error_code=NOTES_CALL_FAILED",
        eventId, statusCode);

Remove errorResponse from the log, and don't pass it into CustomServiceException.

2. DB error log in the InvestigatorActionService onErrorResume:

log.error("Event processing failed: event_id={}, error_code={}, error_type={}",
        eventId, errorCode, error.getClass().getSimpleName());

Do the same in GlobalExceptionHandler. Don't use getMessage(), and don't pass the exception object as the last argument (that prints the stack trace with the message).

3. Hibernate's SQL error logger in application.yml:

logging:
  level:
    org.hibernate.engine.jdbc.spi.SqlExceptionHelper: OFF

4. Member alert, journey log and Application Insights
Send only the error code, event ID and exception type, never getMessage() or the response body.

5. Search for anything else
Search for getMessage(), errorResponse, getResponseBodyAsString and log.error(..., e) in error paths, and fix any that could carry the value.

6. Test
Post a 22-digit access number. The request should still fail, but searching the logs and the member alert for that number should return nothing.