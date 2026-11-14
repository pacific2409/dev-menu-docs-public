# Architecture Outline

## 概要

DevMenuは、利用者向けのWeb Frontend Applicationと、課題生成の制御および共有履歴を担うBackend Applicationで構成する。課題候補の生成自体は外部の生成AIサービスへ委ね、DevMenuは入力検証、生成依頼、応答検証、提案結果の表示、および共有履歴の管理に責任を持つ。

Backend Applicationは一つのApplicationおよびServiceとして稼働するModular Monolithとし、内部を`課題提案`と`共有履歴`の二つの論理Moduleに分ける。課題提案Moduleは課題提案の成立判断、共有履歴Moduleは成立した入力条件と課題提案の永続的な管理を担う。二つのModuleは将来のService分割候補として責務境界を維持するが、分割時期や順序は定めない。

## System Boundary

### 課題生成に関するシステム境界

- DevMenuは、生成条件の受け付け、プロンプトの構成、外部の生成AIサービスへの生成依頼、応答の検証と有限回の再試行、提案結果の表示、および共有履歴の管理を担う。
- 外部の生成AIサービスは、DevMenuから渡された条件に基づく課題候補の生成を担う。
- 生成モデル自体の用意および運用は、DevMenuの責務に含めない。

### 共有履歴に関するシステム境界

- 入力条件と検証済みの課題提案を保存し、共有履歴として提供する責務はDevMenuが担う。
- DevMenuの共有履歴では、外部の生成AIサービスが持つ履歴機能に依存しない。
- 共有履歴の正は、DevMenuが管理するデータとする。

### 既存システムの位置づけ

- DevMenuが取り込み、併存、または置換の対象とする既存システムはない。
- 外部関係は、課題候補の生成を担う生成AIサービスに限定する。

## Application Structure

### FrontendとBackendのApplication境界

- ブラウザ向けの画面を提供するWeb Frontend Applicationと、業務処理を担うBackend Applicationを別Applicationとして構成する。
- Web Frontend Applicationは、利用者との入出力および結果の表示を担う。
- Backend Applicationは、入力の検証、課題生成の制御、および共有履歴に関する処理を担う。

### Web Frontend Applicationの単位

- 課題の注文、提案結果の確認、および共有履歴の閲覧を、一つのWeb Frontend Applicationで提供する。
- 利用者や用途によるWeb Frontend Applicationの分割は行わない。

### Application一覧

- **Web Frontend Application**: 課題の注文、提案結果の確認、および共有履歴の閲覧に関する利用者向けの入出力を担う。
- **Backend Application**: 入力の検証、課題生成の制御、外部の生成AIサービスとの境界、および共有履歴に関する処理を担う。
- Mobile、Desktop、およびCLIのClient Applicationは設けない。

## Backend / Service Structure

### Backendの基本構造

- Backend Applicationは、一つのApplicationおよびServiceとして稼働するModular Monolithとする。
- Backend Application内部では、機能領域に応じた責務を論理的なModuleとして区別する。
- 今回から複数の独立Serviceへ分割する構成は採用しない。

### BackendのModule境界

- Backend Applicationは、同一のApplicationおよびService境界の内部を、`課題提案`と`共有履歴`の二つの論理Moduleに分ける。
- **課題提案Module**: 入力条件の検証、外部の生成AIサービスへの生成依頼、応答の検証、および有限回の再試行を担う。
- **共有履歴Module**: 検証済みの入力条件と課題提案の保存、および技術名に基づく共有履歴の提供を担う。
- 二つのModuleは独立Serviceにはせず、論理的な責務境界として維持する。

## Architecture Evolution

### Backend Moduleの構造変更方針

- `課題提案`と`共有履歴`の二つのModuleは、将来の独立Serviceへの分割候補として責務境界を維持する。
- どちらを先に分割するか、および分割する時期は現時点では決定しない。
- 分割の必要性が生じるまでは、一つのBackend ApplicationおよびServiceとして維持する。

### Application境界の維持方針

- Web Frontend ApplicationとBackend Applicationの分離を維持する。
- 両Applicationの統合、およびWeb Frontend Applicationの用途別分割は、将来の構造変更として方向づけない。

## Responsibility & Ownership

### 機能領域の担当先

- **Web Frontend Application**: 条件入力、提案結果の表示、および共有履歴の閲覧に関する利用者体験を担当する。課題生成と履歴管理の業務判断は担当しない。
- **課題提案Module**: 入力条件の検証、外部の生成AIサービスへの生成依頼、応答の検証、および有限回の再試行を担当する。共有履歴の管理は担当しない。
- **共有履歴Module**: 検証済みの入力条件と課題提案の保存、技術名の管理、および共有履歴の提供を担当する。課題候補の生成は担当しない。
- **外部の生成AIサービス**: DevMenuから渡された条件に基づく課題候補の生成を担当する。DevMenuの入力検証、応答検証、および共有履歴の管理は担当しない。

### 課題提案と共有履歴の分担

- 課題提案Moduleは、生成結果が課題提案として成立しているかを判断する責務を持つ。
- 共有履歴Moduleは、成立した課題提案とその入力条件を永続的に保存し、管理する責務を持つ。
- 成立しなかった生成結果は、共有履歴Moduleの管理対象にしない。

### Data Ownership

- 共有履歴Moduleは、入力条件、任意の意向、検証済みの課題提案、および履歴の技術名に対する管理責任を持つ。
- 課題提案Moduleは、課題生成の処理中に入力条件と生成結果を扱うが、それらの永続データに対する管理責任は持たない。
- Web Frontend Applicationおよび外部の生成AIサービスは、DevMenuの共有履歴データに対する管理責任を持たない。

## 検討状況

- System Boundary: 合意済み
- Application Structure: 合意済み
- Backend / Service Structure: 合意済み
- Architecture Evolution: 合意済み
- Responsibility & Ownership: 合意済み
- 文書全体: 承認済み
