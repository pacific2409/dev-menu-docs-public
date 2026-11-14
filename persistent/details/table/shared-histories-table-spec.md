# Table Definition

## 変更履歴
| Change | Date | Train | Summary |
| --- | --- | --- | --- |
| Initial Design | 2026-10-14 13:23 | 20260915-initial-implementation | 初回設計 |
| Change 1 | 2026-10-15 00:30 | 20260915-initial-implementation | 所要時間を時間単位へ統一し、ColumnとCHECK制約をduration_hoursへ変更。値域は1〜999を維持 |
| Change 2 | 2026-10-28 04:41 | 20260915-initial-implementation | MySQLの物理Metadataに合わせ、Primary KeyのIndex Nameを`PRIMARY`へ訂正 |


## **1. Table Summary**

| Item | Value |
| --- | --- |
| Physical Name | `shared_histories` |
| Logical Name | 共有履歴 |
| Database | `dev_menu` |
| Description | 成立した提案の入力条件、代表技術との関係および保存日時を保持し、3件の実装課題を一つの履歴として関連付ける。 |

## **2. Columns**

| No. | Column | Logical Name | PK | UK | FK | Type (Size) | Not Null | Default | Remarks |
| ---: | --- | --- | :---: | :---: | --- | --- | :---: | --- | --- |
| 1 | `id` | 共有履歴ID | ● | - | - | BIGINT | ● | - | AUTO_INCREMENT |
| 2 | `technology_id` | 代表技術ID | - | - | `technologies.id` | BIGINT | ● | - | 一つの代表技術に属する。Foreign Keyの削除動作は`ON DELETE RESTRICT`とする。 |
| 3 | `difficulty` | 難易度 | - | - | - | CHAR(1) | ● | - | `1`: `BEGINNER`（初級）/ `2`: `INTERMEDIATE`（中級）/ `3`: `ADVANCED`（上級） |
| 4 | `duration_hours` | 所要時間（時間） | - | - | - | SMALLINT UNSIGNED | ● | - | 1〜999の整数 |
| 5 | `purpose` | 目的 | - | - | - | CHAR(1) | ● | - | `1`: `LEARNING`（技術習得）/ `2`: `POC`（PoC） |
| 6 | `additional_preference` | その他の意向 | - | - | - | VARCHAR(200) | - | - | NULLは未入力を表す。前後空白を除去した値を保持し、空白だけの入力はNULLとして扱う。 |
| 7 | `created_at` | 作成日時・保存日時 | - | - | - | DATETIME(6) | ● | - | ApplicationがRecord保存時にUTCで設定し、通常更新では変更しない。共有履歴の保存日時として使用する。 |
| 8 | `updated_at` | 更新日時 | - | - | - | DATETIME(6) | ● | - | ApplicationがRecord作成時および更新時にUTCで設定する。 |
| 9 | `operator` | 更新実行元 | - | - | - | VARCHAR(100) | ● | - | Recordを作成・更新したAPIまたは処理を識別する安定した値。 |
| 10 | `deleted_flag` | 削除フラグ | - | - | - | TINYINT | ● | `0` | `0`: 未削除 / `1`: 削除済み。通常操作では物理削除しない。 |
| 11 | `timestamp` | 排他制御用タイムスタンプ | - | - | - | TIMESTAMP(6) | ● | `CURRENT_TIMESTAMP(6)` | `ON UPDATE CURRENT_TIMESTAMP(6)`によりDBがRecord更新時に自動更新し、楽観的排他制御に使用する。 |

## **3. Indexes**

| No. | Index Name | Type | Columns | Unique | Remarks |
| ---: | --- | --- | --- | :---: | --- |
| 1 | `PRIMARY` | Primary Key | `id` | ● | Primary Key |
| 2 | `mul_shared_histories_technology_deleted_created` | Composite Index | `technology_id`, `deleted_flag`, `created_at DESC` | - | 選択した代表技術の未削除履歴を保存日時の新しい順で取得する。Foreign Key用Indexを兼ねる。 |

## **4. Constraints**

| Constraint Name | Type | Definition | Remarks |
| --- | --- | --- | --- |
| `chk_shared_histories_difficulty` | CHECK | `difficulty IN ('1', '2', '3')` | 定義済みの難易度Codeだけを許可する。 |
| `chk_shared_histories_duration_hours` | CHECK | `duration_hours BETWEEN 1 AND 999` | 所要時間を入力規則の範囲に限定する。 |
| `chk_shared_histories_purpose` | CHECK | `purpose IN ('1', '2')` | 定義済みの目的Codeだけを許可する。 |
| `chk_shared_histories_additional_preference` | CHECK | `additional_preference IS NULL OR CHAR_LENGTH(TRIM(additional_preference)) BETWEEN 1 AND 200` | NULLまたは空白だけではない任意入力だけを許可する。 |
| `chk_shared_histories_deleted_flag` | CHECK | `deleted_flag IN (0, 1)` | 論理削除状態を定義値に限定する。 |
