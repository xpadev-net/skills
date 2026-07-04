# Review Checklist: Data Store / Background Job / Batch

- job id、status、attempt count、published / processed / failed の意味が変更で崩れていないか。
- CAS更新、retry marker、requeue、startup recoveryの失敗を握りつぶして詰まりを作らないか。
- bulk updateやconditional expressionが対象外の行を巻き込まないか。
- migrationや制約追加が既存データで安全に適用できるか。
- workerとAPI、producerとconsumer、readerとwriterで状態遷移が一致しているか。
- delayed job、cron、queue visibility timeout、dead-letter、idempotency key の契約が保たれているか。
