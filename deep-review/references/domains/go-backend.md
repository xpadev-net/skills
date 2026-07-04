# Review Checklist: Go Backend Services

- `context.Context` がI/O、DB、external call、goroutineへ適切に伝播しているか。
- nil pointer、zero value、empty slice/map、typed nil error の扱いが安全か。
- transaction boundary、deferred cleanup、rollback / commit、unlock / close が全経路で正しいか。
- error wrapping、classification、sentinel errors、status mapping が呼び出し元契約と一致しているか。
- gomockやtest doubleの期待値更新だけで実装退行を隠していないか。
- pagination、filter、sort、domain-layer boundary が既存実装と一貫しているか。
- goroutine leak、channel close、timer/ticker cleanup、race-prone shared state がないか。
