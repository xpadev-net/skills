# Review Checklist: Tests

- テストは正常系だけでなく、失敗系・境界値・回帰を直接押さえているか。
- branch coverage相当の重要分岐が検証されているか。
- テストデータが本番相当の値、欠落値、不正値、境界値を含んでいるか。
- `assertIsNotNone` 相当の弱いassertionではなく、具体的な値、status、field、side effectを検証しているか。
- mockやsnapshotの更新が、実装との乖離や仕様変更を隠していないか。
- 並行実行、retry、cleanup、timeout、external invalid response の回帰が必要な箇所で検証されているか。
- テスト追加要求は、何が壊れたら検出できるべきかを指摘内で明確にできるか。
