# レビュー時に確認したい観点

このドキュメントは、特定のプロジェクトやサービスに依存しないレビュー観点をまとめたものです。対象がWebアプリ、CLI、自動化、外部連携、バッチ、リアルタイム処理、生成物配布のどれであっても、同じ問いに置き換えて確認してください。

## 最優先で見る観点

### 1. 境界データ・型・スキーマ

- 外部入力、永続化データ、環境変数、Cookie、クエリ、リクエストボディ、上流レスポンスを実行時に検証しているか。
- 型アサーションや暗黙前提で、不正なデータ形状を隠していないか。
- optional / nullable / unknown / empty の扱いが既存のドメイン契約と一致しているか。
- 検証失敗時に、既存の安全なフォールバックやエラー意味論を壊していないか。
- 正常系の型だけでなく、不正値・欠落値・余分な値・壊れたJSON相当のケースが考慮されているか。

### 2. 権限・所有者・分離

- 操作主体が対象リソースを読む・書く・実行する権限を持つか。
- role / scope / ownership / namespace / tenant / workspace などの境界が全経路で守られているか。
- public / private、authenticated / anonymous、admin / user の処理が混ざっていないか。
- fallback identity、unknown user、missing cache、missing context のような曖昧な状態でfail-openしていないか。
- 権限確認が操作対象の確定後、かつ副作用の直前で行われているか。

### 3. 失敗時の意味論

- 失敗を成功レスポンス、成功UI、空結果、対象なしとして扱っていないか。
- 呼び出し元が期待するstatus、例外、戻り値、イベント、通知、ログに落ちているか。
- 上流障害、検証失敗、権限不足、競合、timeout、キャンセルを区別できるか。
- エラーに機密情報、生レスポンス、巨大payload、内部prompt、debug artifactが混ざらないか。
- 監視・retry・ユーザー通知が必要な失敗を握りつぶしていないか。

### 4. テスト・回帰検出

- 変更が守るべき契約を直接検証するテストがあるか。
- バグ修正では、不具合そのものを再現する回帰テストがあるか。
- 緩いassertionで重要フィールド、条件式、status、job id、owner idなどの退行を見逃さないか。
- 失敗系、境界値、並行実行、retry、cleanup、外部不正レスポンスを押さえているか。
- テストが実装詳細ではなく、破ってはいけない振る舞いを検証しているか。
- mockや期待値の更新だけで実装との乖離を隠していないか。

### 5. リソース制限・過負荷耐性

- rate limit、接続数、キュー長、同時実行数、ファイル数、payloadサイズに上限があるか。
- 外部指定の待機時間やretry delayに上限があり、処理を長時間停止させないか。
- 副作用のある操作をretryしても二重作成・二重送信・二重更新にならないか。
- 制限超過時に、呼び出し元が解釈できる失敗として返せるか。
- カウンタ、permit、slot、lockが例外・キャンセル・切断時にも解放されるか。

## 実装品質として見る観点

### 設計一貫性・共通化

- 同じschema、formatter、label、pagination、query、状態遷移が複数箇所に重複していないか。
- 共通化すべきものと、意味が違うため分けるべきものが区別されているか。
- ローカル定義が既存のドメイン概念、命名、責務境界とズレていないか。
- 既存のhelper、hook、schema、service、adapterを不必要に迂回していないか。
- 抽象化が過剰でないか、または重要な契約が呼び出し側へ漏れていないか。

### 状態管理・ライフサイクル

- effect、listener、timer、subscription、stream、worker、child process の開始と終了が対になっているか。
- unmount / dispose / abort / disconnect / cancellation 後に状態更新や副作用が走らないか。
- 同じ状態を複数箇所で管理して競合・上書き・古い結果の反映が起きないか。
- optimistic update、retry、requeue、reconnect 後に古い状態で新しい状態を壊さないか。
- `finally` やdefer相当の経路でcleanupが必ず実行されるか。

### 並行性・競合・一貫性

- read-after-check、check-then-act、exists-then-open などのTOCTOUがないか。
- bulk update、conditional update、CAS、upsert、retry markerが意図した対象だけに効くか。
- 同じjob、message、event、file、recordが重複処理されないか。
- 高速完了、遅延完了、並列retry、再起動復旧でguard条件をすり抜けないか。
- producer側とconsumer側で状態遷移の解釈が一致しているか。

## 変更領域ごとの重点チェック

### API / backend

- request / response / upstream response の検証が揃っているか。
- auth guard、owner check、namespace check が全経路で効いているか。
- status code、error body、retryabilityが契約と一致しているか。
- pagination、count、list、searchが過剰に重い処理を増やしていないか。
- 条件付き更新が並行実行や古い状態の上書きを防いでいるか。
- usecase / domain / repository などのレイヤー境界を迂回していないか。
- トランザクション内のエラー処理、rollback / commit の分岐、defer cleanup が全経路で安全か。
- スナップショット用フィールドや集計済み値が、元データ変更後も整合するか。

### User interface

- ユーザー操作が失敗したときに成功表示へ進まないか。
- loading / empty / error / permission denied / not found の状態が破綻しないか。
- 表示文言、タイトル、ラベル、空状態が共通設定や既存UIと一貫しているか。
- accessibility、focus、keyboard、読み上げ、装飾要素の扱いが自然か。
- lifecycle cleanup、非同期完了後の状態更新、古いデータの一瞬表示がないか。

### External integration

- 上流レスポンスの欠落、不正型、不正JSON、空結果、部分失敗を扱えるか。
- upstream failureを安全なエラーに変換しており、成功や空結果と混同していないか。
- token、scope、prefix、resource id、callback payloadなどを攻撃者入力として扱っているか。
- retry、timeout、cancellation、rate limit、idempotencyが定義されているか。
- ログ・エラー・artifactに機密や生payloadを漏らしていないか。

### Automation / review / CI workflow

- ローカルレビュー規約が指定する言語、重大度、除外パターン、外部API禁止事項を守っているか。
- revision、base/head、timestamp、event id など、差分判定の基準が用途と一致しているか。
- 時刻境界で同時刻のevent/comment/resultを誤って含めたり除外したりしないか。
- thread、reply、通常comment、summary、machine-generated notification の取得範囲と分類が正しいか。
- trigger comment、hidden prompt、internal artifact、rate-limit noticeなどのnoiseをユーザー向け出力に混ぜていないか。
- 外部チェックや非同期レビューを待つ場合、待機上限・キャンセル・警告・再試行方針があるか。

### External commands / generated artifacts

- subprocessや外部ツールにtimeout、cancellation、process group cleanupが効くか。
- stdout/stderr作成失敗、read-back失敗、timeout、kill失敗でもtemp fileやchild processが残らないか。
- temp file名、artifact path、cache keyが並行実行で衝突しないか。
- ダウンロード可能なartifactは権限確認、所有者確認、path traversal対策を通るか。
- download header、filename、manifest、client-side labelから制御文字や危険文字を除去しているか。
- partial read、range request、削除競合、ファイル欠落でstatusと情報漏えいが一貫しているか。

### Event-driven / realtime workflow

- eventの発行者、実行者、対象workspace/channel/resourceの権限が確認されているか。
- permission cache、state cache、presence/cache miss、missing contextのときにfail-closedするか。
- timer、grace period、disconnect、reconnect、requeue、startup recoveryで誤停止・停止漏れ・重複処理がないか。
- 接続開始後に返せない失敗を、開始前に判定できているか。
- disconnect / abort / backpressure / slow consumerでpermit、subscription、bufferが解放されるか。
- ユーザーに見えるべき失敗が通知・再試行・保留状態として残るか。

### Data store / background job / batch

- job id、status、attempt count、published / processed / failed の意味が変更で崩れていないか。
- CAS更新、retry marker、requeue、startup recoveryの失敗を握りつぶして詰まりを作らないか。
- bulk updateやconditional expressionが対象外の行を巻き込まないか。
- migrationや制約追加が既存データで安全に適用できるか。
- workerとAPI、producerとconsumer、readerとwriterで状態遷移が一致しているか。

### Media / time-series / ordered data

- timestamp、sample count、sequence number、offset、durationが推定ではなく実データ基準か。
- gap、overlap、clock drift、out-of-order、partial flush、final flush failureを扱えるか。
- 複数sourceのalign、merge、split、mixdownで末尾や空白区間が欠けないか。
- 失敗したchunkやsegmentが保持・再試行・通知されるか。
- no-op tickや高頻度eventでretryが過剰に走らないか。

## レビューコメントを書く前の確認

- 指摘が現行差分に対して再現可能か。古いdiffやoutdated threadだけを見ていないか。
- 即時バグ、潜在リスク、仕様変更、テスト不足を分けて書けるか。
- 修正案が既存パターンに沿った最小変更になっているか。
- テスト追加を求める場合、何を壊したら検出できるべきかを明確にできているか。
- 自動生成されたsummaryではなく、actionableな指摘として成立しているか。

## 短いチェックリスト

- [ ] 境界データは実行時検証されているか。
- [ ] 型アサーションや暗黙前提で不正データを隠していないか。
- [ ] 認証・認可・所有者・namespace確認が全経路で保たれているか。
- [ ] 失敗時のstatus / UI / event / retry意味論が正しいか。
- [ ] 外部入力・上流レスポンスの欠落や不正値を扱えるか。
- [ ] 外部コマンドのtimeout・kill・temp file cleanupが全エラー経路で効くか。
- [ ] ローカルレビュー規約の言語・除外パターン・投稿禁止事項を守っているか。
- [ ] automationのrevision・時刻境界・thread取得・noise filteringが正しいか。
- [ ] generated artifactの認可・sanitize・partial read挙動が安全か。
- [ ] event-driven workflowのscope、invoker authorization、cache failureがfail-closedか。
- [ ] media / ordered dataでtimestamp、sample count、sequenceが実データ基準か。
- [ ] 並行実行、retry、早期returnで状態を壊さないか。
- [ ] リソース制限とcleanupが例外・切断時にも効くか。
- [ ] UI文言、空状態、accessibility、共通設定が一貫しているか。
- [ ] lifecycle cleanupと非同期完了後の状態更新が安全か。
- [ ] テストが正常系だけでなく、失敗系・境界値・回帰を押さえているか。
- [ ] Go/backendではnil、transaction、layer boundary、gomock期待値、pagination/filter整合性を確認したか。
- [ ] 重複実装や既存helper迂回が増えていないか。
- [ ] PR descriptionと実装・検証内容が一致しているか。
