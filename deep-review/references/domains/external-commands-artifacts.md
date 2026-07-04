# Review Checklist: External Commands / Generated Artifacts

- subprocessや外部ツールにtimeout、cancellation、process group cleanupが効くか。
- stdout/stderr作成失敗、read-back失敗、timeout、kill失敗でもtemp fileやchild processが残らないか。
- temp file名、artifact path、cache keyが並行実行で衝突しないか。
- ダウンロード可能なartifactは権限確認、所有者確認、path traversal対策を通るか。
- download header、filename、manifest、client-side labelから制御文字や危険文字を除去しているか。
- partial read、range request、削除競合、ファイル欠落でstatusと情報漏えいが一貫しているか。
- 大容量ファイル、生成物、画像、動画、PDFが不要に直接コミットされていないか。
