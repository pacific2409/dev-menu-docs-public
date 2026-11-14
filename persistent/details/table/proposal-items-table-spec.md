# Table Definition

## 変更履歴
| Change | Date | Train | Summary |
| --- | --- | --- | --- |
| Initial Design | 2026-10-14 13:23 | 20260915-initial-implementation | 初回設計 |
| Change 1 | 2026-10-28 04:41 | 20260915-initial-implementation | MySQLの物理Metadataに合わせ、Primary KeyのIndex Nameを`PRIMARY`へ訂正 |


## **1. Table Summary**

| Item | Value |
| --- | --- |
| Physical Name | `proposal_items` |
| Logical Name | 実装課題 |
| Database | `dev_menu` |
| Description | 一つの共有履歴に属する3件の実装課題について、提案番号、タイトル、概要、達成目標および実践観点を保持する。 |

## **2. Columns**

| No. | Column | Logical Name | PK | UK | FK | Type (Size) | Not Null | Default | Remarks |
| ---: | --- | --- | :---: | :---: | --- | --- | :---: | --- | --- |
| 1 | `id` | 実装課題ID | ● | - | - | BIGINT | ● | - | AUTO_INCREMENT |
| 2 | `shared_history_id` | 共有履歴ID | - | ● | `shared_histories.id` | BIGINT | ● | - | 一つの共有履歴に属する。Foreign Keyの削除動作は`ON DELETE RESTRICT`とする。 |
| 3 | `proposal_number` | 提案番号 | - | ● | - | TINYINT UNSIGNED | ● | - | 一つの共有履歴内で`1`〜`3`を保持する。`shared_history_id`との組合せで一意とする。 |
| 4 | `title` | タイトル | - | - | - | VARCHAR(100) | ● | - | 前後空白を除去した、空ではない日本語のタイトル。 |
| 5 | `overview` | 概要 | - | - | - | VARCHAR(500) | ● | - | 前後空白を除去した、空ではない日本語の概要。 |
| 6 | `achievement_goal` | 達成目標 | - | - | - | VARCHAR(500) | ● | - | 前後空白を除去した、空ではない日本語の達成目標。 |
| 7 | `practice_focus` | 実践観点 | - | - | - | VARCHAR(500) | ● | - | 前後空白を除去した、空ではない日本語の文章。親履歴の目的が`1`なら学ぶポイント、`2`なら検証するポイントを表す。 |
| 8 | `created_at` | 作成日時 | - | - | - | DATETIME(6) | ● | - | ApplicationがRecord作成時にUTCで設定し、通常更新では変更しない。 |
| 9 | `updated_at` | 更新日時 | - | - | - | DATETIME(6) | ● | - | ApplicationがRecord作成時および更新時にUTCで設定する。 |
| 10 | `operator` | 更新実行元 | - | - | - | VARCHAR(100) | ● | - | Recordを作成・更新したAPIまたは処理を識別する安定した値。 |
| 11 | `deleted_flag` | 削除フラグ | - | - | - | TINYINT | ● | `0` | `0`: 未削除 / `1`: 削除済み。通常操作では物理削除しない。 |
| 12 | `timestamp` | 排他制御用タイムスタンプ | - | - | - | TIMESTAMP(6) | ● | `CURRENT_TIMESTAMP(6)` | `ON UPDATE CURRENT_TIMESTAMP(6)`によりDBがRecord更新時に自動更新し、楽観的排他制御に使用する。 |

## **3. Indexes**

| No. | Index Name | Type | Columns | Unique | Remarks |
| ---: | --- | --- | --- | :---: | --- |
| 1 | `PRIMARY` | Primary Key | `id` | ● | Primary Key |
| 2 | `uk_proposal_items_history_number` | Unique Key | `shared_history_id`, `proposal_number` | ● | 同一共有履歴内の提案番号重複と4件目を防止し、Foreign Key用Indexを兼ねる。3件未満は共有履歴と3件を一括保存するJTA Transactionの成立条件として防止する。 |

## **4. Constraints**

| Constraint Name | Type | Definition | Remarks |
| --- | --- | --- | --- |
| `chk_proposal_items_proposal_number` | CHECK | `proposal_number BETWEEN 1 AND 3` | 提案番号を1〜3に限定する。 |
| `chk_proposal_items_title_not_blank` | CHECK | `CHAR_LENGTH(TRIM(title)) BETWEEN 1 AND 100` | 空白だけのタイトルを拒否する。 |
| `chk_proposal_items_overview_not_blank` | CHECK | `CHAR_LENGTH(TRIM(overview)) BETWEEN 1 AND 500` | 空白だけの概要を拒否する。 |
| `chk_proposal_items_achievement_goal_not_blank` | CHECK | `CHAR_LENGTH(TRIM(achievement_goal)) BETWEEN 1 AND 500` | 空白だけの達成目標を拒否する。 |
| `chk_proposal_items_practice_focus_not_blank` | CHECK | `CHAR_LENGTH(TRIM(practice_focus)) BETWEEN 1 AND 500` | 空白だけの実践観点を拒否する。 |
| `chk_proposal_items_deleted_flag` | CHECK | `deleted_flag IN (0, 1)` | 論理削除状態を定義値に限定する。 |
