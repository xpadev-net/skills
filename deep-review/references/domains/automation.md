# Review Checklist: Automation / Review / CI Workflow

- ローカルレビュー規約が指定する言語、重大度、除外パターン、外部API禁止事項を守っているか。
- revision、base/head、timestamp、event id など、差分判定の基準が用途と一致しているか。
- 時刻境界で同時刻のevent/comment/resultを誤って含めたり除外したりしないか。
- thread、reply、通常comment、summary、machine-generated notification の取得範囲と分類が正しいか。
- trigger comment、hidden prompt、internal artifact、rate-limit noticeなどのnoiseをユーザー向け出力に混ぜていないか。
- 外部チェックや非同期レビューを待つ場合、待機上限・キャンセル・警告・再試行方針があるか。
- CI状態、required check、flaky retry、manual approval の扱いがPRの実際のマージ条件と一致しているか。
