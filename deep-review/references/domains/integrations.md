# Review Checklist: External Integrations

- 上流レスポンスの欠落、不正型、不正JSON、空結果、部分失敗を扱えるか。
- upstream failureを安全なエラーに変換しており、成功や空結果と混同していないか。
- token、scope、prefix、resource id、callback payloadなどを攻撃者入力として扱っているか。
- retry、timeout、cancellation、rate limit、idempotencyが定義されているか。
- ログ・エラー・artifactに機密や生payloadを漏らしていないか。
- API pagination、backfill、webhook再送、順序入れ替わり、重複deliveryに耐えられるか。
