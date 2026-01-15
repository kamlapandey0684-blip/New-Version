# Error Handling Quick Reference

## Error Types at a Glance

### Quota Exhausted ❌
**Shows**: "API key quota exhausted"
**Cause**: Monthly API usage limit reached
**Solution**: Upgrade plan or wait for reset
**Link**: https://console.cloud.google.com/apis/dashboard

### Rate Limited ⏱️
**Shows**: "Too many requests" (429)
**Cause**: Making requests too fast
**Solution**: Wait 5-10 minutes, then retry

### Server Overloaded 🔄
**Shows**: "Server is overloaded" (503)
**Cause**: Gemini API temporarily busy
**Solution**: Automatic retry, or wait 2-5 minutes

### Gateway Error 🚨
**Shows**: "Gateway error" (502)
**Cause**: API infrastructure issue
**Solution**: Automatic retry, or wait 1-2 minutes

### Timeout ⏳
**Shows**: "Request timed out"
**Cause**: Response took too long (60s+)
**Solution**: Check internet, try fewer URLs

---

## Error Messages Format

```
Stage N: [Description]

Action: [Specific recommendation]
```

**Example**:
```
Stage 2: Gemini API server is currently overloaded (503).
The service is temporarily unavailable. Please try again
in a few minutes.

Action: Try again in a few minutes. The service is
temporarily unavailable.
```

---

## Where Errors Appear

1. **Top-Right Notification** ← Main error display
2. **Browser Console** ← Detailed logs with emoji
3. **Processing Status** ← Shows current stage being processed

---

## Console Log Indicators

| Emoji | Meaning | Action |
|-------|---------|--------|
| ❌ | Error occurred | Fatal error, needs user action |
| ⚠️ | Warning | Retrying automatically |
| ⏱️ | Timeout | Request took too long |
| 🔄 | Retrying | Exponential backoff in progress |
| ✅ | Success | Operation completed |

---

## Automatic Retry Behavior

**Error Type**: 429, 503, 502, Timeout
**Max Attempts**: 4 (1 initial + 3 retries)
**Backoff**: Exponential (10s, 20s, 40s)
**User Sees**: Error only after all retries fail

**Example Timeline**:
```
10:00:00 - Initial request fails (503)
10:00:10 - Retry 1 (waited 10s)
10:00:20 - Still fails (503)
10:00:40 - Retry 2 (waited 20s)
10:00:41 - Still fails (503)
10:01:21 - Retry 3 (waited 40s)
10:01:22 - Still fails
10:01:22 - Show error to user
```

---

## User Action Timeline

### Quota Exhausted
```
See Error → Check Dashboard → Upgrade/Wait → Retry
```

### Rate Limited
```
See Error → Wait 5-10 min → Retry
(Or automatic retry happens)
```

### Server Overloaded
```
Automatic Retry (10s, 20s, 40s) → Success or Error
(Usually succeeds on automatic retry)
```

### Timeout
```
See Error → Check Internet → Try Fewer URLs → Retry
```

---

## Common Error Scenarios

### Scenario 1: Used Up Free Quota
```
Error: Stage 2: Gemini API key quota exhausted...

Action:
1. Go to console.cloud.google.com/apis/dashboard
2. Check "Gemini API" usage
3. Upgrade plan if needed
4. Wait for monthly reset (usually 1st of month)
5. Retry operation
```

### Scenario 2: Too Many Requests
```
Error: Stage 2: Too many requests to Gemini API...

Action:
1. Wait 5-10 minutes
2. The system will retry automatically
3. If manual, click "Retry" button when ready
4. Don't make multiple requests at once
```

### Scenario 3: API Is Down
```
Error: Stage 2: Gemini API server is currently overloaded...

Action:
1. This is automatic retry (usually succeeds)
2. If shown to user, wait 2-5 minutes
3. Retry the operation
4. If still fails, try again later
```

### Scenario 4: Slow Internet
```
Error: Stage 2: Request timed out...

Action:
1. Check your internet connection
2. Switch to WiFi if on cellular
3. Try with fewer URLs (Stage 2)
4. Retry the operation
```

---

## Troubleshooting Flowchart

```
Operation Failed
    ↓
Check Error Message
    ├─ "quota exhausted"
    │  └─ Check API quota & upgrade
    ├─ "rate limited"
    │  └─ Wait 5-10 minutes & retry
    ├─ "overloaded"
    │  └─ Wait 2-5 minutes & retry
    ├─ "timed out"
    │  └─ Check internet & retry
    └─ "gateway error"
       └─ Wait 1-2 minutes & retry
```

---

## Quick Actions

| Need | Action | Link |
|------|--------|------|
| Check API Quota | Visit Dashboard | https://console.cloud.google.com/apis/dashboard |
| Upgrade Plan | Check Pricing | https://ai.google.dev/pricing |
| API Docs | Read Documentation | https://ai.google.dev/docs |
| Status Check | Gemini Status | https://status.ai.google.dev/ |

---

## Key Files

| File | Purpose |
|------|---------|
| `src/utils/api.ts` | Error detection & retry logic |
| `src/App.tsx` | Error UI display |
| `API_ERROR_HANDLING.md` | Complete error guide |
| `ERROR_HANDLING_SUMMARY.md` | Summary of changes |

---

## Prevention Tips

1. **Monitor Usage**: Check API quota regularly
2. **Batch Processing**: Don't submit too many at once
3. **Smart Retry**: Let system auto-retry (don't force retry)
4. **Good Network**: Use stable internet connection
5. **Off-Peak**: Process during non-busy times

---

## Support

### If Error Persists

1. **Check** console logs for detailed info
2. **Note** exact error message
3. **Try** different URLs or fewer URLs
4. **Wait** if server is overloaded
5. **Upgrade** if quota exhausted

### Common Questions

**Q: Will my data be lost?**
A: No, errors don't delete data. Just retry.

**Q: How long should I wait?**
A: Quota: Days-months | Rate: 5-10 min | Server: 2-5 min | Timeout: 1-2 min

**Q: Can I retry immediately?**
A: For quota/rate-limit: No, wait suggested time. For others: You can try.

**Q: Is this normal?**
A: Yes, all APIs have limits. Proper error handling ensures reliability.

---

## Status Codes Reference

| Code | Name | Meaning |
|------|------|---------|
| 200 | OK | Success ✅ |
| 400 | Bad Request | Invalid request (won't retry) |
| 401 | Unauthorized | Invalid API key (won't retry) |
| 429 | Too Many Requests | Rate limited or quota exhausted (auto-retry) |
| 502 | Bad Gateway | API gateway issue (auto-retry) |
| 503 | Service Unavailable | Server overloaded (auto-retry) |

---

Last Updated: 2024
Version: 1.0
