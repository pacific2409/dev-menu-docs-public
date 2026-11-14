# DevMenu

> [!NOTE]
> 本Repositoryは、ポートフォリオ公開用に作成したProject成果物のCopyです。Security、Environment固有情報、運用記録、作業履歴その他の一部内容は公開対象から除外しています。そのため、内部Repositoryの完全な構成および履歴とは一致しません。

## Project Overview

DevMenuは、学びたい技術、難易度、所要時間および目的に応じた実装課題を提示し、成立した提案を共有履歴から再発見できるWebアプリケーションである。

- **Purpose**: システムエンジニアが、技術習得や小さなPoCで取り組む具体的な実装課題を決められるようにする。
- **Published Snapshot**: Initial Implementation（Production ReleaseおよびProduction Verification完了時点）

要求と対象範囲の正本は[Product Requirements Definition](persistent/product-requirements.md)とする。

## System Overview

利用者向けのReact Web Frontendと、Jakarta EEで構築したBackend Applicationを分離して配置する。Backendは課題提案と共有履歴の二つの論理Moduleを持つModular Monolithであり、Amazon Bedrockによる課題候補生成とMySQLによる共有履歴の永続化を担う。

### Overall Architecture

- Systemの責務境界、Application構成、Technology StackおよびDelivery方針は[System Architecture](persistent/system-architecture.md)を参照する。
- 機能、Business Rule、主要FlowおよびDataの関係は[Function Design](persistent/function-design.md)を参照する。
- API、画面、TableおよびError Codeの詳細は[Detailed Specifications](persistent/details/)を参照する。

### Infrastructure Architecture

AWS東京Region上で、FrontendとBackendをAmazon EKS on Fargate、永続DataをAmazon RDS for MySQLへ配置する。公開Traffic、Container Registry、Secret連携、ObservabilityおよびAmazon Bedrock連携を含むInfrastructureは、Terraform / TerragruntとGitLab CI/CDを中心に管理する。

- [Infrastructure Architecture](persistent/infrastructure-architecture.md)
- [Infrastructure Architecture Diagram](persistent/infrastructure-architecture.png)
- [Infrastructure-to-Application Dependencies](persistent/infrastructure-to-application-dependencies.md)

### Modules

| Module | Domain | Responsibility |
|---|---|---|
| `aws` | Infrastructure | AWS Infrastructure、Environment、Identity、Delivery基盤およびLifecycleを管理する。 |
| `database-state-management` | Database | Database Schema、User、GrantおよびCredential Stateの変更を管理する。 |
| `proposal-service` | Backend | 課題提案、Amazon Bedrock連携、共有履歴およびBackend運用Endpointを提供する。 |
| `web` | Frontend | 課題の注文・提案結果と共有履歴を提供するWeb UIを管理する。 |

## Documentation

### Persistent Documents

| Document | Description |
|---|---|
| [Product Requirements Definition](persistent/product-requirements.md) | Productの目的、対象利用者、機能要件および非機能要件 |
| [Architecture Outline](persistent/architecture-outline.md) | System Boundary、Application構造および責務の骨子 |
| [Function Design](persistent/function-design.md) | 機能、Business Rule、Flow、DataおよびInterfaceの基本設計 |
| [System Architecture](persistent/system-architecture.md) | Application、Integration、Data、Security、TestおよびDelivery Architecture |
| [Infrastructure Architecture](persistent/infrastructure-architecture.md) | Environment、AWS Resource、Security、Delivery、RecoveryおよびLifecycle Architecture |
| [Infrastructure-to-Application Dependencies](persistent/infrastructure-to-application-dependencies.md) | Applicationと詳細設計が消費するInfrastructure Contractの索引 |
| [Detailed Specifications](persistent/details/) | API、External Interface、画面、Database、TableおよびError Codeの詳細仕様 |
| [Non-public Artifact Index](NON_PUBLIC_ARTIFACTS.md) | 内容を非公開としたSteering、Review、Handoffその他の成果物一覧 |

### Document Structure

| Path | Role |
|---|---|
| `persistent/` | 公開対象とした要求、設計および詳細仕様 |
| `persistent/details/` | API、External Interface、画面、Database、TableおよびError Codeの詳細仕様 |

## Documentation Notes

- 本READMEは公開成果物への入口であり、仕様や設計を再定義しない。
- 公開対象とした要求・仕様・設計は、`persistent/`配下の各文書を参照する。
- Infrastructure Architectureおよび関連文書では、公開に不要なSecurity、Identity、Environment、Delivery、Recoveryその他の具体情報を節または表単位で省略する場合がある。
- 非公開部分の不存在や、内部Repositoryとの完全一致を保証するものではない。
