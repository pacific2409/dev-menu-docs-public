# Table Definition

## 変更履歴
| Change | Date | Train | Summary |
| --- | --- | --- | --- |
| Initial Design | 2026-10-14 13:23 | 20260915-initial-implementation | 初回設計 |
| Change 1 | 2026-10-28 04:41 | 20260915-initial-implementation | MySQLの物理Metadataに合わせ、Primary KeyのIndex Nameを`PRIMARY`へ訂正 |


## **1. Table Summary**

| Item | Value |
| --- | --- |
| Physical Name | `technologies` |
| Logical Name | 代表技術 |
| Database | `dev_menu` |
| Description | 正規化により同一視された技術の代表表示名を保持し、共有履歴を技術単位で分類できるようにする。 |

## **2. Columns**

| No. | Column | Logical Name | PK | UK | FK | Type (Size) | Not Null | Default | Remarks |
| ---: | --- | --- | :---: | :---: | --- | --- | :---: | --- | --- |
| 1 | `id` | 代表技術ID | ● | - | - | BIGINT | ● | - | AUTO_INCREMENT |
| 2 | `display_name` | 代表表示名 | - | ● | - | VARCHAR(20) | ● | - | 前後空白と連続空白を正規化済みの値。Database既定の`utf8mb4_0900_ai_ci`により大文字小文字を区別せず一意とし、最初に保存した表記を維持する。 |
| 3 | `created_at` | 作成日時 | - | - | - | DATETIME(6) | ● | - | ApplicationがRecord作成時にUTCで設定し、通常更新では変更しない。 |
| 4 | `updated_at` | 更新日時 | - | - | - | DATETIME(6) | ● | - | ApplicationがRecord作成時および更新時にUTCで設定する。 |
| 5 | `operator` | 更新実行元 | - | - | - | VARCHAR(100) | ● | - | Recordを作成・更新したAPIまたは処理を識別する安定した値。 |
| 6 | `deleted_flag` | 削除フラグ | - | - | - | TINYINT | ● | `0` | `0`: 未削除 / `1`: 削除済み。通常操作では物理削除しない。 |
| 7 | `timestamp` | 排他制御用タイムスタンプ | - | - | - | TIMESTAMP(6) | ● | `CURRENT_TIMESTAMP(6)` | `ON UPDATE CURRENT_TIMESTAMP(6)`によりDBがRecord更新時に自動更新し、楽観的排他制御に使用する。 |

## **3. Indexes**

| No. | Index Name | Type | Columns | Unique | Remarks |
| ---: | --- | --- | --- | :---: | --- |
| 1 | `PRIMARY` | Primary Key | `id` | ● | Primary Key |
| 2 | `uk_technologies_display_name` | Unique Key | `display_name` | ● | 大文字小文字を区別せず代表表示名を一意にする。 |

## **4. Constraints**

| Constraint Name | Type | Definition | Remarks |
| --- | --- | --- | --- |
| `chk_technologies_display_name_not_blank` | CHECK | `CHAR_LENGTH(TRIM(display_name)) BETWEEN 1 AND 20` | 空白だけの代表表示名を拒否する。連続空白の正規化はApplicationで行う。 |
| `chk_technologies_deleted_flag` | CHECK | `deleted_flag IN (0, 1)` | 論理削除状態を定義値に限定する。 |
