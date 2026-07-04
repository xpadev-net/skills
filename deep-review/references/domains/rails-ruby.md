# Review Checklist: Rails / Ruby / RSpec

## Database And Performance

- N+1クエリが発生していないか。`includes`、`preload`、`eager_load` の使い分けが適切か。
- `joins`、`delete_all`、`update_all`、bulk処理の想定件数とtimeoutリスクが妥当か。
- 大量データ処理では `find_each`、`find_in_batches` などを検討しているか。
- migrationを含め、必要なindex、NOT NULL、UNIQUE、foreign keyなどの制約があるか。
- 必要なカラムのみ取得しているか。`count` より `exists?` が適切な場面を見逃していないか。
- `where` に空配列を渡すケースや、一覧取得時のorder保証を考慮しているか。

## Architecture

- Fat Controllerになっておらず、アプリケーションロジックとビジネスロジックの配置が適切か。
- Service Object、Form Object、Concern、Decoratorなどの導入が責務分離に効いているか。
- RESTfulな設計、MVCの責務、DRY/SOLIDが既存コードと整合しているか。
- コールバックを濫用していないか。副作用の発火タイミングが暗黙になっていないか。

## Security And Integrity

- SQL injection、Strong Parameters、XSS、resource access制限が適切か。
- model validation とDB制約が必要な境界で両方効いているか。
- `dependent: :destroy`、enum、関連データ、transaction、rollback条件、`rescue` / `ensure` が安全か。

## Tests

- RSpecがAAAパターンで読みやすく、契約を具体値で検証しているか。
- FactoryBotのtraitやfactoryが過剰に複雑でなく、本番相当の値と境界値を表現できているか。
- migration、callback、transaction、validation failure の回帰テストが必要な箇所にあるか。
