# Review Checklist: Media / Time-Series / Ordered Data

- timestamp、sample count、sequence number、offset、durationが推定ではなく実データ基準か。
- gap、overlap、clock drift、out-of-order、partial flush、final flush failureを扱えるか。
- 複数sourceのalign、merge、split、mixdownで末尾や空白区間が欠けないか。
- 失敗したchunkやsegmentが保持・再試行・通知されるか。
- no-op tickや高頻度eventでretryが過剰に走らないか。
- timestamp/sample/sequence の単位や基準時刻がAPI、保存形式、UI表示で一致しているか。
