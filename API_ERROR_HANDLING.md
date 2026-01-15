# Gemini API Error Handling Guide

## Overview
Enhanced error handling for Gemini API to display clear, actionable error messages when API keys are exhausted or Gemini servers are overloaded.

---

## Error Types Detected

### 1. API Key Quota Exhausted (429 + Quota Error)
**Status Code**: `429`
**Indication**: Error response contains "quota" or "QUOTA_EXCEEDED"

**User Message**:
```
Gemini API key quota exhausted. Your API has reached its monthly usage limit.
Please upgrade your plan or wait until the quota resets.
```

**Recommended Action**:
- Check your API quota at: https://console.cloud.google.com/apis/dashboard
- Upgrade your Google Cloud plan if needed
- Wait until the quota resets (typically monthly)

---

### 2. Rate Limited (429)
**Status Code**: `429`
**Indication**: Too many requests to the API

**User Message**:
```
Gemini API rate limited (429). Too many requests.
Please wait a few minutes and try again.
```

**Recommended Action**:
- Wait at least 5-10 minutes before retrying
- Reduce concurrent requests
- Space out API calls

---

### 3. Server Overloaded (503)
**Status Code**: `503`
**Indication**: Gemini server is experiencing high load

**User Message**:
```
Gemini API server is currently overloaded (503).
The service is temporarily unavailable. Please try again in a few minutes.
```

**Recommended Action**:
- Wait 2-5 minutes and retry
- The service should recover automatically
- Try again during off-peak hours

---

### 4. Gateway Error (502)
**Status Code**: `502`
**Indication**: API gateway issue

**User Message**:
```
Gemini API gateway error (502).
There's an issue with the API service. Please try again in a few minutes.
```

**Recommended Action**:
- Wait 1-2 minutes
- Retry the request
- Check Gemini API status page

---

### 5. Request Timeout
**Condition**: Request exceeds 60 seconds with no response

**User Message**:
```
Request timed out after N attempts.
The Gemini API server is not responding. Please try again later.
```

**Recommended Action**:
- Check internet connection
- Wait a few minutes
- Retry with fewer/smaller URLs

---

## Error Flow Diagram

```
API Request
    ↓
Response Check
    ├─ 200 OK → Success
    ├─ 429 (Quota) → API Key Exhausted
    ├─ 429 (Rate) → Rate Limited
    ├─ 503 → Server Overloaded
    ├─ 502 → Gateway Error
    └─ Timeout → Request Timeout
         ↓
    Detailed Error Message
         ↓
    User Notification
         ↓
    Recommended Action
```

---

## UI Error Display

### Error Toast Notification

Located at **top-right** of the screen, the error notification displays:

1. **Error Title**: Bold, red text
2. **Error Description**: Full error message
3. **Recommended Action**: Context-specific action suggestion
4. **Close Button**: ✕ to dismiss

### Visual Hierarchy

```
┌─────────────────────────────────────┐
│ Error                         ✕     │
├─────────────────────────────────────┤
│ Stage 2: Gemini API key quota       │
│ exhausted. Your API has reached     │
│ its monthly usage limit...          │
├─────────────────────────────────────┤
│ Action: Check your API quota at     │
│ console.cloud.google.com/apis/...   │
└─────────────────────────────────────┘
```

---

## Error Messages by Stage

### Stage 1: Specification Audit

**Error Prefix**: `Stage 1: `

Possible Errors:
- Quota Exhausted
- Rate Limited
- Server Overloaded

### Stage 2: Website ISQ Extraction

**Error Prefix**: `Stage 2: `

Possible Errors:
- Quota Exhausted
- Rate Limited
- Server Overloaded
- Timeout (URLs taking too long to fetch)

### Stage 3: Buyer ISQ Generation

**Error Prefix**: Not prefixed (inherits from findCommonSpecsWithGemini)

Possible Errors:
- Quota Exhausted
- Rate Limited
- Server Overloaded

---

## Console Logging

### Error Logs
All errors are logged to browser console with emoji indicators:

```javascript
❌ Gemini API key quota exhausted for Stage 1 (Audit)
❌ Gemini API rate limited for Stage 2
⚠️ Gemini server overloaded (503). Waiting 10000ms before retry...
⏱️ Request timeout on attempt 1
```

### Retry Logs
Automatic retries are logged with exponential backoff:

```javascript
⚠️ Gemini rate limited (429). Retrying in 10000ms
⚠️ Gemini server overloaded (503). Waiting 20000ms before retry...
```

---

## Retry Strategy

### Exponential Backoff

The system automatically retries failed requests with exponential backoff:

```
Attempt 1: Wait 10s before retry
Attempt 2: Wait 20s before retry
Attempt 3: Wait 40s before retry
Attempt 4: Give up, show error to user
```

**Code**:
```javascript
const waitTime = baseDelay * Math.pow(2, attempt);
```

### Retryable Status Codes

- **429** (Rate Limited)
- **503** (Service Unavailable)
- **502** (Bad Gateway)

### Non-Retryable Errors

- **Invalid API Key** (401)
- **Invalid Request** (400)
- **Parsing Errors**
- **Timeout** (after max retries)

---

## Error Detection Logic

### Quota Exhausted Detection

```javascript
if (response.status === 429) {
  const errorText = await response.text();

  if (errorText.includes('quota') || errorText.includes('QUOTA_EXCEEDED')) {
    throw new Error(`Gemini API key quota exhausted...`);
  }
}
```

### Server Overload Detection

```javascript
if (response.status === 503 || response.status === 502) {
  const detailedError = response.status === 503
    ? `Gemini API server is currently overloaded (503)...`
    : `Gemini API gateway error (502)...`;

  throw new Error(detailedError);
}
```

---

## User Actions by Error Type

| Error Type | Wait Time | Action | Notes |
|-----------|-----------|--------|-------|
| Quota Exhausted | N/A | Upgrade plan or wait until reset | Typically monthly reset |
| Rate Limited | 5-10 mins | Reduce request frequency | Automatic retry in code |
| Server Overloaded | 2-5 mins | Automatic retry | Built-in exponential backoff |
| Timeout | 1-2 mins | Check connection, retry | Try with fewer URLs |
| Gateway Error | 1-2 mins | Automatic retry | Service should recover |

---

## Testing Error Scenarios

### How to Simulate Errors (for development)

#### 1. Trigger 429 (Quota) Error
- Make many rapid requests (exhaust quota)
- Or modify API key to trigger quota error

#### 2. Trigger 503 (Overload) Error
- Make requests during peak hours
- Or use Rate Limiter tool in browser DevTools

#### 3. Trigger Timeout Error
- Add very long URLs
- Throttle network in DevTools (Slow 3G)
- Add delay in mock responses

#### 4. Trigger 502 (Gateway) Error
- Network issues
- API gateway maintenance

---

## Best Practices

### For Users

1. **API Quota**: Monitor your API usage regularly
2. **Rate Limiting**: Wait between large batch operations
3. **Retries**: Don't retry immediately; wait the suggested time
4. **Errors**: Read the error message for specific guidance
5. **Network**: Check connection before assuming server issues

### For Developers

1. **Error Messages**: Keep messages user-friendly
2. **Logging**: Always log errors for debugging
3. **Retry Logic**: Use exponential backoff
4. **Timeouts**: Set reasonable timeout limits (60s for API)
5. **Status Codes**: Handle all error codes gracefully

---

## Related Documentation

- **Gemini API Documentation**: https://ai.google.dev/docs
- **Google Cloud Console**: https://console.cloud.google.com
- **API Quota Help**: https://cloud.google.com/docs/quota
- **Rate Limiting Guide**: https://ai.google.dev/docs/rate-limiting

---

## FAQ

### Q: My API key is exhausted. What should I do?

**A**:
1. Go to https://console.cloud.google.com/apis/dashboard
2. Check your usage limits
3. Either upgrade your plan or wait until the quota resets (usually monthly)

### Q: I'm getting rate limited. How long should I wait?

**A**:
- The system automatically retries with exponential backoff
- Manual retry: wait 5-10 minutes before trying again
- Consider spacing out your requests

### Q: The server is overloaded. Will my data be lost?

**A**:
- No, the server overload is temporary
- Your data is safe
- Automatic retries will eventually succeed
- Wait 2-5 minutes and try again

### Q: Why is the request timing out?

**A**:
Common causes:
- URLs are taking too long to load
- Your internet connection is slow
- Server is heavily loaded

Solutions:
- Check internet connection
- Try with fewer URLs
- Wait and retry

### Q: Can I disable automatic retries?

**A**: Not in the current UI. Automatic retries use exponential backoff to safely handle temporary failures.

---

## Error Tracking

### Error Categories

```javascript
{
  "quota_exhausted": 429 + quota_error,
  "rate_limited": 429,
  "server_overloaded": 503,
  "gateway_error": 502,
  "timeout": AbortError,
  "other": any other error
}
```

### Error Resolution Time

- **Quota Exhausted**: Days to months (depends on plan)
- **Rate Limited**: Minutes (5-10 min)
- **Server Overloaded**: Minutes (2-5 min)
- **Gateway Error**: Minutes (1-2 min)
- **Timeout**: Seconds to minutes (retry)

---

## Summary

The enhanced error handling provides:

1. **Clear Error Messages**: Users understand what went wrong
2. **Actionable Guidance**: Specific steps to resolve issues
3. **Automatic Retries**: Handles temporary failures
4. **Detailed Logging**: For troubleshooting
5. **Stage-Specific Errors**: Context about which operation failed

This ensures a better user experience and reduces support requests by providing clear, helpful error information.
