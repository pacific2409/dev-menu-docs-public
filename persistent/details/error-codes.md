# **Error Codes**

## **Definition Policy**

- 本Artifactは、Applicationで使用するError Codeの共通定義を管理する。

- Error Codeは`AA0001`または`AAA001`の形式とし、原因Categoryを表す2〜3文字の英字識別子と、その識別子内で一意となる3〜4桁の連番で構成する。

- 英字識別子はErrorの発生箇所ではなく原因を基準に命名し、同一の原因または接続先に起因するErrorでは統一する。

- Error Codeは、同一の識別範囲において一意とし、一つのCodeに複数の異なる意味を持たせない。

- Error Codeの共通的な意味およびClientへ公開する共通Contractを定義し、個々のErrorが発生する具体的な条件、判定Logicおよび処理遷移は、対応するAPI / Batch / UI等のSpecificationを正とする。

- 個別Specificationを正とする発生条件、判定Logic、処理遷移その他の個別仕様は重複して記載しない。

- 同一Error Codeを複数のSpecificationから利用する場合も、本Artifactに定義された意味を変更して使用しない。

- External Service等から受信するError Codeは、それ自体を本ApplicationのError Codeとして公開する場合を除き、本Artifactへ登録しない。External ErrorからApplication ErrorへのMappingは、対応するSpecificationで定義する。

- Communication Error等、ApplicationのError Codeを使用しない事象に対して、形式的にError Codeを新設しない。

- Error Messageに可変値を含める場合は、Error Codeが表す意味を変えない範囲でPlaceholderを使用する。

- Error Codeの追加・変更・廃止時は、そのCodeを利用するSpecificationとの整合性を維持する。


## **Error Code Definition**

| Error Code | HTTP Status | Error Message | Description | Note |
| --- | --- | --- | --- | --- |
| `REQ001` | `400 Bad Request` | `The request format is invalid.` | Request Bodyの欠落または構文不正等により、Requestを所定の構造として解釈できない。 | - |
| `REQ002` | `415 Unsupported Media Type` | `The media type is not supported.` | RequestのMedia TypeがAPIで対応する形式ではない。 | - |
| `REQ003` | `400 Bad Request` | `The path parameter format is invalid.` | Path ParameterがAPIで定義された形式または値の表現範囲を満たさない。 | - |
| `VE0001` | `400 Bad Request` | `The input is invalid.` | 解釈可能なRequestに含まれる一つ以上の入力値が、APIで定義されたValidation Ruleを満たさない。 | Field別の理由は各APIの`validationMessage` Contractを正とする。 |
| `GAI001` | `500 Internal Server Error` | `Failed to generate practice proposals.` | 生成AIの応答を処理できたが、Applicationが要求する成立条件を満たす課題提案を構成できない。 | Request不正または外部Serviceの技術的失敗を表さない。 |
| `BR9999` | `500 Internal Server Error` | `An Amazon Bedrock error occurred.` | Amazon Bedrockとの連携において、通信、Timeout、Service、認証または設定等の技術的失敗が発生した。 | External Errorの詳細はClientへ公開しない。 |
| `DB9999` | `500 Internal Server Error` | `A database error occurred.` | Database接続、Query、Connection PoolまたはTimeout等の技術的失敗が発生した。 | 処理Contractにより内部記録だけに使用し、Clientへ返さない場合がある。 |
| `TEC001` | `404 Not Found` | `The specified technology was not found.` | 指定された代表技術が存在しない、または取得対象ではない。 | - |
| `TEC002` | `-` | `-` | 正規化済み技術名に一致するTechnologyが論理削除済みであるため、成立提案を共有履歴へ保存できない。 | 内部記録専用。Clientへ返さない。 |
| `SH0001` | `404 Not Found` | `The specified shared history was not found.` | 指定された共有履歴が存在しない、または取得対象ではない。 | - |
| `SH0002` | `500 Internal Server Error` | `The shared history data is inconsistent.` | 共有履歴の保存データが整合性条件を満たさない。 | 詳細な不整合内容は内部Logで管理する。 |
| `ER9999` | `500 Internal Server Error` | `Unknown error.` | 個別のError Codeへ安全に分類できない予期しない内部Errorが発生した。 | 原因の内部詳細はLogで管理し、Clientへ公開しない。 |

## **Definition Notes**

- `Error Code`には、Application内でErrorを一意に識別するCodeを記載する。
- `HTTP Status`には、HTTP InterfaceでClientへ公開する場合のStatusを記載する。HTTPを使用しない用途では`-`とする。
- `Error Message`には、Clientまたは利用者へ公開する共通Messageを記載する。
- `Description`には、Error Codeが表す意味を、個別の発生箇所に依存しない粒度で記載する。
- `Note`には、適用範囲、Message Placeholderその他の共通的な補足が必要な場合のみ記載する。
- Validation Message等、個々のFieldや発生箇所によって変化するMessageは、共通Contractとして管理する必要がある場合を除き、本Artifactでは定義しない。
- Errorの発生条件、判定Logic、Retry / Skip / Continue / Abort、画面上のPresentation等は、それぞれ対応するSpecificationを正とする。
