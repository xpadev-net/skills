# Review Checklist: User Interface

- ユーザー操作が失敗したときに成功表示へ進まないか。
- loading / empty / error / permission denied / not found の状態が破綻しないか。
- 表示文言、タイトル、ラベル、空状態が共通設定や既存UIと一貫しているか。
- accessibility、focus、keyboard、読み上げ、装飾要素の扱いが自然か。
- lifecycle cleanup、非同期完了後の状態更新、古いデータの一瞬表示がないか。
- レスポンシブ表示でテキストや操作要素が重ならず、対象ユーザーの主要操作が効率的か。
- エラーバウンダリーやユーザー向け通知が、失敗の種類と回復手段を正しく伝えるか。
