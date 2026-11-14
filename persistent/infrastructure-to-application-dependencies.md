# Infrastructure-to-Application Dependencies

> [!NOTE]
> 本文書はポートフォリオ公開用のCopyです。InfrastructureからApplicationへ引き渡す実環境固有のIdentity、Delivery、IngressおよびRecovery Contractの一部は、節または表単位で除外しています。

## 1. Purpose and Scope

本書は、Function DesignおよびSystem ArchitectureをApplication設計上の前提とし、Infrastructure Architectureから、Application Detail Design（API、External Interface、UI、Table、Application Implementation等）で考慮、実現または利用する必要がある依存事項を抽出した引継ぎ文書である。

Infrastructure Resourceの単純な要約、Infrastructureだけで完結する構築事項、および新たなApplication Architecture Decisionは対象外とする。Application側の詳細値や実現方法が未確定の箇所は、後続の各詳細設計で既存Architectureの範囲内に具体化する。

## 2. Runtime and Workload Dependencies

| ID | Application Consumer / Design Target | Infrastructure Dependency | Application-side Consideration / Required Realization | Source |
| --- | --- | --- | --- | --- |
| IAD-001 | Frontend Implementation / Deployment | Reactの静的ArtifactをNginx ContainerとしてEKS on Fargateへ配置する。FrontendはDeployment、ClusterIP Serviceとして実行される。 | ContainerはStatelessかつ非rootで動作し、静的Artifact配信とSPA Fallbackだけを担う。Business LogicおよびBackend Reverse Proxyを持たせない。 | Infrastructure Architecture 3.2、6.6、7.1、7.3、18.7 / System Architecture 13.1、13.2 |
| IAD-002 | Backend Implementation / Deployment | Jakarta EE BackendのWARをPayara Micro Containerへ格納し、EKS on Fargate上のDeployment、ClusterIP Serviceとして実行する。 | Java 21、Jakarta EE 11および実装時に固定するPayara Micro 7.x Patchとの互換性を保つ。Container内へ継続状態を保持せず、環境固有値をImageへ埋め込まない。 | Infrastructure Architecture 3.2、6.6、7.1、7.3 / System Architecture 5.1、5.2、13.1、13.2 |
| IAD-003 | Frontend / Backend Kubernetes Manifest | 各Environment NamespaceへFrontend/Backendを同一構造で配置し、通常Replicaは各1、Rolling Updateは`maxUnavailable: 0`、`maxSurge: 1`とする。 | DeploymentにRequests/Limits、Startup/Readiness/Liveness Probe、`terminationGracePeriodSeconds: 30`を明示する。単一Replica停止を許容し、初回はPDBおよびHPAを前提にしない。 | Infrastructure Architecture 6.7、6.8、12.2～12.4、13.2 |
| IAD-004 | Frontend / Backend Kubernetes Manifest | WorkloadはEnvironment NamespaceとComponent別ServiceAccountによって配置・分離される。`SecurityGroupPolicy`はServiceAccount Labelを選択する。 | Frontend/Backend専用ServiceAccountを使用し、`devmenu.io/component=frontend`または`devmenu.io/component=backend`の安定LabelとDeploymentからのServiceAccount参照を維持する。Fargate Profile固有LabelやSG IDをApplication Manifestへ固定しない。 | Infrastructure Architecture 7.4～7.6、10.6、16.5 |
| IAD-005 | Batch Specification | 初回TrainのApplication Workloadは常駐Frontend/Backendだけであり、Batch、CronJob、Application Messaging/Queue Infrastructureは用意されない。 | 初回TrainのApplication Detail Designで、Infrastructureの存在を前提とするBatch、CronJob、Queue Consumerまたは非同期Workerを追加しない。必要になった場合はRequirementおよびArchitectureから再検討する。 | Infrastructure Architecture 6.7、8.5 / System Architecture 8.2、12.3 |

## 3. Public Interface and Traffic Dependencies

| ID | Application Consumer / Design Target | Infrastructure Dependency | Application-side Consideration / Required Realization | Source |
| --- | --- | --- | --- | --- |
| IAD-006 | UI Specification / Frontend API Client | Public EntryはEnvironment Hostの同一Origin ALBであり、`/`をFrontend、`/api`をBackendへRoutingする。TLSはALBで終端し、未知Host/PathはALB固定`404`となる。 | FrontendはBackend APIを相対URL`/api`で呼び出し、CORSや別Originを前提にしない。Frontend RouteとAPI Pathが競合しないようAPI SpecificationおよびSPA Routingを具体化する。 | Infrastructure Architecture 7.3、9.1～9.4、18.5 / System Architecture 8.3、15.4 |
| IAD-007 | API Specification / Backend Implementation | ALBからBackend PodへはVPC内HTTPで転送され、Public HTTPSおよびCertificateはALB/ACMが担う。ALB Idle Timeoutは120秒である。 | Backend自身へPublic Certificate Private Keyを配布せず、外部API処理のTimeoutを120秒未満で詳細設計する。公開URLやSchemeの扱いが必要な場合は、ALB終端後の実行を前提にAPI境界で整合させる。 | Infrastructure Architecture 9.2、18.2、18.6 |
| IAD-008 | API Specification / UI Specification | 利用者向けEndpointは認証なしのPublic HTTPSで提供される。WAFはManaged RuleによりTrafficを検査するが、Applicationの入力検証を代替しない。 | Function Designの入力規則をFrontendとBackendへ実装し、Backendを最終判定とする。WAFの存在を理由に入力検証、JSON構造検証、安全な文字列表示を省略しない。 | Infrastructure Architecture 9.1、10.7 / Function Design 6.1、6.3、12.2 / System Architecture 11.1、11.2、11.6 |
| IAD-009 | API Specification / Deployment Verification | ALBは認証不要の軽量Health EndpointをTarget Healthに使用する。BackendのReadinessは初期化状態とMySQL接続可能性を確認するが、Bedrockを能動確認しない。 | Health EndpointのPath、ResponseおよびMicroProfile Healthとの対応をAPI/Backend詳細設計で定義する。機密情報や詳細Dependency情報を返さず、Bedrock障害だけでBackendをNot Readyにしない。 | Infrastructure Architecture 13.4、13.5 / System Architecture 6.5 |

### IAD-029: Shared ALB Ingress Participation

Frontend / BackendのIngressは、Infrastructureが所有する共有Public Entryへ、Environmentごとに定義された参加Contractに従って接続する。Applicationは自身のHost / Path Routingだけを所有し、恒久Ingress、Default Action、TLS、WAFその他の共通設定を変更しない。Deploy前にRendered Manifestを検査し、担当範囲外の設定を上書きしていないことを確認する。

> [!NOTE]
> 公開版では、IngressGroup名、Annotation、Listener、Host、Rule順序およびEnvironment別の検査条件を除外しています。

## 4. Configuration, Secret and Identity Dependencies

| ID | Application Consumer / Design Target | Infrastructure Dependency | Application-side Consideration / Required Realization | Source |
| --- | --- | --- | --- | --- |
| IAD-010 | Frontend / Backend Configuration Design | Non-secret Environment ConfigurationはKustomize OverlayからConfigMapへ配布され、ApplicationはEnvironment変数またはMicroProfile Configを介して参照する。ConfigMap変更は内容HashでRolling Updateされる。Backend設定Keyの供給元、Environment差分および更新反映は[Infrastructure Architecture](infrastructure-architecture.md) 18.3のHandoff Matrixを参照する。 | Host、Endpoint、Mode、Timeout等をCodeやImageへ固定しない。各Config Keyの値、型、必須性、Defaultおよび起動時検証はHandoff Matrixが参照するApplication Detail Designを正とし、Applicationの読込とOverlayの供給を一致させる。Payara固有設定管理を正本にしない。 | Infrastructure Architecture 11.1～11.3、11.8、18.2、18.3 |
| IAD-011 | Backend Configuration / Secret Integration | DB Credential等のSecret ValueはEnvironment/用途別Secrets Managerを正本とし、ESOがKubernetes Secretへ同期する。ApplicationはMicroProfile ConfigからSecret Referenceを参照する。`database.username`／`database.password`の供給元、Environment分離およびSecret VersionIdによる更新反映は[Infrastructure Architecture](infrastructure-architecture.md) 18.3のHandoff Matrixを参照する。 | ApplicationはSecret名/KeyをContractとして扱い、Secrets Manager APIを直接呼ばない。必須性・起動時検証はHandoff Matrixが参照するApplication Detail Designに従う。Secret ValueをSource、Image、Manifest、Logへ出力せず、Master、Application、Migration Credentialを混用しない。 | Infrastructure Architecture 11.4～11.6、18.3、18.4 / System Architecture 10.1、11.7 |
| IAD-012 | Backend Deployment / Runtime | Secret変更時はESO同期後、非機密のSecret VersionIdをPod Template Annotationへ反映してDeploymentを更新する。通常同期間隔は1時間である。 | Secret ValueのHash計算や埋込みに依存せず、Credential Rotation中に新旧状態を安全に切り替え、更新後のDB接続を検証できるようにする。 | Infrastructure Architecture 11.6、16.5 |
| IAD-013 | Bedrock Adapter / Backend Deployment | BackendはInfrastructureから引き渡されるEnvironment別IRSA Role ARNを対応EnvironmentのBackend ServiceAccountへ設定し、そのRoleの短期CredentialをSDK Default Credential Provider Chainから利用する。RoleはNova Lite Invokeだけを許可され、development / stagingにはNon-production Workload Boundary、productionにはProduction Workload Boundaryが設定される。 | Environment Overlayは引き渡されたRole ARNを対応Namespace / ServiceAccountへ関連付け、別EnvironmentのRole ARNを使用しない。Access Keyを設定値やSecretとして固定配置せず、AWS SDKおよびModel固有処理をBedrock Adapterへ限定し、RoleにないAWS操作をApplicationから要求しない。Application Deployment後にEnvironment別のBedrock接続を検証する。 | Infrastructure Architecture 9.7、10.2、10.4、18.5、18.9 / System Architecture 6.4、9.2、11.4、11.7 |
| IAD-014 | Frontend / Backend Environment Design | EnvironmentごとにNamespace、Host、Image Digest、Config/Secret Reference、DB Schema/UserおよびBedrock利用Modeが異なる。developmentはStub/Real切替、staging/productionはRealを使用し、productionでStub Fallbackは禁止される。 | Environment差分をConfiguration/Overlayで吸収し、ArtifactをEnvironmentごとに再Buildしない。Real/Stubは同じApplication Portを実装し、productionでStubへ切り替わるDefaultやFallbackを設けない。 | Infrastructure Architecture 2.7、7.2、11.8、18.9 / System Architecture 9.3、9.5、9.6、13.5 |

## 5. Data Persistence Dependencies

| ID | Application Consumer / Design Target | Infrastructure Dependency | Application-side Consideration / Required Realization | Source |
| --- | --- | --- | --- | --- |
| IAD-015 | Table Definition / Persistence Adapter | Data StoreはRDS for MySQL 8.4 LTSであり、`utf8mb4`、`utf8mb4_0900_ai_ci`、UTCを使用する。BackendはContainer-managed DataSource/Connection PoolからTLS/JDBCで接続する。 | Table、Column、Index、Query、JPA MappingおよびJDBC設定をMySQL 8.4、指定Charset/Collation、UTCと整合させる。RDS Server Certificateを検証し、TLS検証を無効化しない。 | Infrastructure Architecture 3.7、8.2、10.8、18.5、18.6 / System Architecture 5.3、10.1、11.5 |
| IAD-016 | Table Definition / Backend Configuration | Non-productionではdevelopment/stagingが一つのRDS Instanceを共有するが、Schema、Application User、Migration User、CredentialはEnvironment別に分離される。Productionは専用RDS Instanceを使用する。 | ApplicationはEnvironment固有のEndpoint、Port、Schema、Application Credentialを外部設定として受け取り、他EnvironmentのSchema/Userを参照しない。Migration処理とRuntime処理のCredentialを分離する。 | Infrastructure Architecture 2.2、8.2、8.6、18.5、18.9 |
| IAD-017 | Table Definition / Schema Delivery | Schema SQLはApplication Deployと分離してVersion管理・手動適用される。EclipseLink自動DDL、Flyway、Liquibaseは初回採用されない。Schema変更はExpand/Contractで新旧Backendと互換にする。 | Application起動時にSchemaを自動生成・変更しない。Table DefinitionとVersion管理SQLを一致させ、Backend変更は旧新Versionと共存可能なSchema順序を守る。 | System Architecture 13.6、13.7 / Infrastructure Architecture 13.7 |
| IAD-018 | Backend Persistence / Capacity Design | 初期DBはSingle-AZ `db.t4g.micro`、単一Writerであり、Connection、CPU、Storageが主要Constraintとなる。Cache、Read Replica、Shardingはない。 | Connection Pool、Transaction、IndexおよびQueryを小規模DB Capacityに合わせ、I/OやConnectionを長時間占有しない。Application CacheやReplicaの存在を前提にせず、MySQLをSource of Truthとする。 | Infrastructure Architecture 8.2、12.7、12.9 / System Architecture 10.4、12.2、12.4、12.6 |

## 6. External Service Dependencies

| ID | Application Consumer / Design Target | Infrastructure Dependency | Application-side Consideration / Required Realization | Source |
| --- | --- | --- | --- | --- |
| IAD-019 | External Interface Specification / Bedrock Adapter | BackendはNAT Gateway経由で`ap-northeast-1`のAmazon Bedrock Public EndpointへHTTPS接続し、Amazon Nova Lite `amazon.nova-lite-v1:0`のConverse APIを使用する。 | Region、Model、TimeoutおよびModeを外部設定として扱い、Converse APIのRequest/Response/Error変換をAdapterへ閉じ込める。Prompt、Model Parameterおよび応答Schemaは既存の成立条件の範囲でExternal Interface Detail Designに具体化する。 | Infrastructure Architecture 5.4、9.7、18.2、18.5 / System Architecture 9.2、9.7 |
| IAD-020 | API / External Interface Error Design | NAT、Bedrock、認証、設定またはTimeoutによる技術Failureは、生成応答の不成立による再生成とは異なる。Infrastructure障害時もBedrock用の自動Fallback基盤はない。 | SDK内部の技術的Retryは許容し、Retryおよび待機を1回の呼出し全体のTimeoutに含める。SDKが最終的に通知した技術Failureに対してApplicationおよびAdapter独自の追加試行は行わず、注文を失敗とする。応答不成立に対する最大3回は初回を含むApplicationからの生成依頼回数であり、SDK内部の通信試行数とは区別する。developmentの明示的Stub選択を除き、通信Failure時にStubへFallbackしない。 | Function Design 6.1 BR-011～BR-012、11章 / Infrastructure Architecture 13.4 / System Architecture 9.3～9.6 |

## 7. Observability and Operational Dependencies

| ID | Application Consumer / Design Target | Infrastructure Dependency | Application-side Consideration / Required Realization | Source |
| --- | --- | --- | --- | --- |
| IAD-021 | Frontend / Backend Logging Design | Containerの標準出力/標準エラーはFargate Log RouterからEnvironment/Component別CloudWatch Log Groupへ送られる。Container内Fileは永続化されない。 | Backendは1 Event 1 JSONの単一行LogとRequest IDを標準出力へ出す。Credential、利用者入力、生成内容を既定で記録せず、Container FileをLogの正本にしない。Frontendも運用上必要な出力をContainer標準出力/標準エラー境界へ合わせる。 | Infrastructure Architecture 6.8、15.3 / System Architecture 6.5、11.5 |
| IAD-022 | API Specification / Backend Cross-cutting Design | InfrastructureはRequest ID、Structured LogおよびCloudWatch Log検索によって追跡し、Distributed Tracingは初回導入しない。 | BackendはInbound Requestごとに信頼できるRequest IDを生成し、Log ContextとResponse Headerへ設定してRequest終了時に削除する。API SpecificationはResponse Header Contractを反映する。 | Infrastructure Architecture 15.3、15.4 / System Architecture 6.5 |
| IAD-023 | Backend Metrics Design | Backend JSON Metric EventはCloudWatch Logs Metric FilterでStaging/Production Custom Metricへ変換される。Event形式、8種のMetric名、単位およびMappingの正本は[Infrastructure Architecture](infrastructure-architecture.md) 15.2とする。Dimensionは`Environment`と有限な`Operation`だけで、安定名を使用する。 | `eventType="application-metric"`、`metric`、非負の数値`value`、`environment`、`operation`を持つ単一行JSONを出力する。Countは発生ごとの都度値、Latencyはミリ秒とし、累積Counterを送らない。developmentも同じEvent形式を使用する。生成Portの計測はApplicationとAdapterで二重計上せず、SDK内部Retryを独立計測しない。SDK内で回復した失敗は計上せず、再生成成功でも先行する生成依頼の失敗数は取り消さない。生成依頼の失敗と最終API Errorを分離し、原因詳細はLogで扱う。利用者入力、生成内容、Request IDその他High-cardinality値をMetric Dimensionへ含めない。 | Infrastructure Architecture 15.2、15.5、15.8 / System Architecture 6.5 / 課題候補生成External Interface Specification 9章 Measurement Boundary |
| IAD-024 | Backend Error Handling / Verification | 利用者へ公開しない履歴保存失敗もCloudWatch Log/Metricで検知・調査可能である必要がある。DashboardやAlarmはTest Evidenceを代替しない。 | Function Designどおり提案成功を維持しながら、保存失敗をError CodeとRequest IDを持つ内部ERRORおよびMetricとして記録する。保存Transaction／Commit失敗ごとに`HistorySaveFailureCount`を`value=1`、`operation=history-create`で出力し、IAD-023のEvent Contractに従う。Integration Testでは利用者向け結果と内部観測の双方を検証する。 | Function Design 6.1 BR-014、11章、12.4 / Infrastructure Architecture 15.2、15.7、15.8 / 課題提案API Specification Step 5.3 |

## 8. Delivery, Lifecycle and Verification Dependencies

| ID | Public Dependency Summary | Application-side Responsibility |
| --- | --- | --- |
| IAD-025 | Application ArtifactはImmutableに管理し、Integrationで検証した同一ArtifactをProductionへ昇格する。 | Environment固有設定をArtifactから分離し、新旧VersionとSchemaの互換性を維持する。 |
| IAD-026 | ApplicationのDevelopment / Integrationは、必要なInfrastructure Contractの成立と引渡し後に開始する。Environmentへの反映には明示的な確認Gateを設ける。 | 対象Environmentと変更差分を確認し、Infrastructure所有Resourceまたは対象外Environmentを変更しない。 |
| IAD-027 | EnvironmentはActive / DormantのLifecycleを持ち、Restore時にEndpointやRuntime Resourceが再生成され得る。 | 固定されたInfrastructure実値に依存せず、Restore後のConfigurationとReferenceを受け取って起動できるようにする。 |
| IAD-028 | Restore後はInfrastructure、Database、Applicationおよび公開経路を依存順に検証し、変更なしRestoreと新変更を伴うReleaseを区別する。 | Health、Integrationおよび主要User Flowを再実行可能にし、検証済みArtifactとEnvironmentを対応付ける。 |

> [!NOTE]
> 公開版では、Role、Branch / Ref、Runner、Job、Approval、Evidence、Lifecycle ManifestおよびEnvironment別の復元・検証順序に関する具体的なDelivery Contractを除外しています。

## 9. Detailed Design Allocation

| Detailed Design | Dependencies to Consume |
| --- | --- |
| UI Specification | IAD-006、IAD-008、IAD-014 |
| API Specification | IAD-006～IAD-009、IAD-020、IAD-022 |
| External Interface Specification | IAD-013、IAD-014、IAD-019、IAD-020 |
| Table Definition / Schema SQL | IAD-015～IAD-018 |
| Frontend Detailed Design / Implementation | IAD-001、IAD-003、IAD-004、IAD-006、IAD-010、IAD-021、IAD-025～IAD-029 |
| Backend Detailed Design / Implementation | IAD-002～IAD-004、IAD-007～IAD-024、IAD-025～IAD-029 |
| Application CI/CD Pipeline | IAD-025、IAD-026、IAD-029 |
| Batch Specification | IAD-005 |
| Test Design | IAD-008、IAD-009、IAD-014、IAD-020～IAD-024、IAD-026、IAD-028 |

## 10. Usage and Change Rule

- 後続設計は、本書をInfrastructure Architectureの代替ではなく、Application側が消費する依存事項の索引として使用する。
- Endpoint、Config Key、Secret Reference、Role、Namespace、Lifecycle Contractを変更する場合は、Infrastructure Architecture 18.10に従い、Infrastructure、ApplicationおよびPipelineの互換性を同一変更単位で確認する。
- 本書にないApplication Architecture Decisionが必要になった場合は、本書へ暗黙に追加せず、該当するRequirementまたはArchitectureの変更手順へ戻る。
