# Review Checklist: Event-Driven / Realtime / Streaming

- eventの発行者、実行者、対象workspace/channel/resourceの権限が確認されているか。
- permission cache、state cache、presence/cache miss、missing contextのときにfail-closedするか。
- timer、grace period、disconnect、reconnect、requeue、startup recoveryで誤停止・停止漏れ・重複処理がないか。
- 接続開始後に返せない失敗を、開始前に判定できているか。
- disconnect / abort / backpressure / slow consumerでpermit、subscription、bufferが解放されるか。
- ユーザーに見えるべき失敗が通知・再試行・保留状態として残るか。
- protocol-level failure、close code、retryability、fallback bucket が既存クライアント契約と一致しているか。
