```bash
at 13:07:28 ❯ mc admin prometheus generate lcminio
scrape_configs:
- job_name: minio-job
  bearer_token: eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJwcm9tZXRoZXVzIiwic3ViIjoicUdYc3JYbDh6OEpUWThtWDNxZ28iLCJleHAiOjQ4ODQwNDEyNTl9.AKfl4sdbrZjpfX96NonsQ-1W5R4cE236ToAKVy-qEXO5FzvYotfl4I6XCfSakL5WiBe4tVEYKNirvB3hjAqukg
  metrics_path: /minio/v2/metrics/cluster
  scheme: http
  static_configs:
  - targets: ['2.2.0.1:9000']
```