# Review Checklist: Next.js / React / TypeScript

## Architecture

- UI、ロジック、データ取得の責務分離が保たれているか。
- Container / Presentational、custom hook、context の抽象化が過剰でも不足でもないか。
- Pages Router / App Router の規約、API routes、middleware、redirect、dynamic route の配置が適切か。
- TypeScriptの型安全性が境界で失われていないか。`as` や non-null assertion が不正値を隠していないか。

## Data Fetching And State

- `getServerSideProps`、`getStaticProps`、`getStaticPaths`、ISR、server/client component の選択が用途に合っているか。
- form handling と validation が、UI上の制約だけでなくサーバー境界でも効いているか。
- session、JWT、role-based access control、protected routes がclient-only判定に依存していないか。

## Performance

- `useMemo` / `useCallback` は必要な箇所に限られ、メモ化コストのほうが高くなっていないか。
- 不要なre-render、重い計算、巨大props、context再レンダーが増えていないか。
- `next/image`、font optimization、static generation、ISRの利用が既存方針と合っているか。

## Design System

- 直接的な色・サイズ指定ではなく、デザイントークンやテーマ関数を使っているか。
- プロジェクト固有のスタイル変数より、既存デザインシステムを優先しているか。
- 新旧コンポーネントの責務分離が曖昧になっていないか。

## Security And SEO

- XSS、CSRF、CSP、secure headers、HTTPS redirect、environment variables、secretsの扱いが安全か。
- SEO meta、title、canonical、error / 404 page が変更後も適切か。

## Tests

- unit、component、interaction、routing、error state のテストが変更契約を守っているか。
- 数値フォーマットや計算ロジックでは NaN、Infinity、負数、大きな数、丸め誤差の境界を押さえているか。
