# Deployment Information

## Public URL
https://my-production-agent-production-4adb.up.railway.app

## Platform
Railway

## Test Commands

### Health Check
```bash
curl https://my-production-agent-production-4adb.up.railway.app/health
# Expected: {"status": "ok"}
```

### API Test (with authentication)
```bash
curl -X POST https://my-production-agent-production-4adb.up.railway.app/ask \
  -H "X-API-Key: secret" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "test", "question": "Hello"}'
```

## Environment Variables Set
*(Các biến môi trường đã được cài đặt trên Railway)*
- REDIS_URL
- AGENT_API_KEY
- PORT

## Screenshots
*(Hãy tạo thư mục `screenshots` và lưu ảnh minh chứng vào đó)*
- [Deployment dashboard](screenshots/dashboard.png)
- [Service running](screenshots/running.png)
- [Test results](screenshots/test.png)
