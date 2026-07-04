# Review Checklist: API / Backend

- request / response / upstream response の検証が揃っているか。
- auth guard、owner check、namespace check が全経路で効いているか。
- status code、error body、retryabilityが契約と一致しているか。
- pagination、count、list、searchが過剰に重い処理を増やしていないか。
- 条件付き更新が並行実行や古い状態の上書きを防いでいるか。
- usecase / domain / repository などのレイヤー境界を迂回していないか。
- トランザクション内のエラー処理、rollback / commit の分岐、defer cleanup が全経路で安全か。
- スナップショット用フィールドや集計済み値が、元データ変更後も整合するか。
- ループ内DBアクセス、不要なcount、未boundedな検索などの性能退行がないか。
