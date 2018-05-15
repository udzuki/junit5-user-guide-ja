# 8. APIの進化
JUnit 5の大きな目標の一つは、多くのプロジェクトに使われているにも関わらず、JUnit 5を進化させるために保守運用者ができることを改善させることです。JUnit 4では、元は内側の構成物として加えられていた多くのものが、外部の拡張者やツールビルダーに使われました。そのことが、JUnit 4を特に難しく、時には不可能なものにしました。

そんなわけで、JUnit 5は全てのpublicに利用可能なインターフェイス、クラス、メソッドに対して、あらかじめ定義したライフサイクルを導入します。

## 8.1. APIバージョンとステータス
全ての公開された生成物は、バージョン番号 `<メジャー>.<マイナー>.<パッチ>`を持っており、全てのpublicに利用可能なインターフェイス、クラス、メソッドは、[`@API Guardian`](https://github.com/apiguardian-team/apiguardian)プロジェクトから[`@API`](https://apiguardian-team.github.io/apiguardian/docs/current/api/)を付与されています。アノテーションの`status`属性は、次の値のうち、1つを割り当てることができます。

|ステータス|説明|
|:--:|:--:|
|`INTERNAL`|JUnit自身以外のコードから使われてはなりません。事前通知なしに削除されるかもしれません。|
|`DEPRECATED`|もう使うべきではありません；次のマイナー・リリースで消滅するかもしれません。|
|`EXPERIMENTAL`|フィードバックを求めている新しい実験的な機能を意図しています。この要素は注意を持って使ってください；`MAINTAINED`か`STABLE`に将来、昇格するかもしれませんが、事前通知なしにパッチ内でも削除されるかもしれません。|
|`MAINTAINED`|**少なくとも**現在のメジャーバージョンにおける次のマイナー・リリースまでは後方非互換に変更されることはない機能を意図しています。もし削除される予定になった場合は、最初に`DEPRECATED`に降格されます。|
|`STABLE`|現在のメジャーバージョン（`5.*`）では後方非互換に変更されることはないことを意図しています。|

もし`@API`アノテーションがある型に存在していた場合、その型の全てのpublicなメンバーにも同様に適用されると考えられます。メンバーは、低い安定性の`status`値を宣言することが許されています。

## 8.2. 実験的なAPIs
次の表は、現在のところ、*実験的*（`@API(status = EXPERIMENTAL)`を通して）と指定されているAPIを示しています。これらのAPIsを使うときは注意してください。

|パッケージ名|クラス名|タイプ|
|:--:|:--:|:--:|
|`org.junit.jupiter.api`|`AssertionsKt`|`クラス`|
|`org.junit.jupiter.api`|`DynamicContainer`|`クラス`|
|`org.junit.jupiter.api`|`DynamicNode`|`クラス`|
|`org.junit.jupiter.api`|`DynamicTest`|`クラス`|
|`org.junit.jupiter.api`|`TestFactory`|`アノテーション`|
|`org.junit.jupiter.api.condition`|`DisabledIf`|`アノテーション`|
|`org.junit.jupiter.api.condition`|`EabledIf`|`アノテーション`|
|`org.junit.jupiter.api.extension`|`ScriptEvaluationException`|`クラス`|
|`org.junit.jupiter.params`|`ParameterizedTest`|`アノテーション`|
|`org.junit.jupiter.params.aggregator`|`AggregateWith`|`アノテーション`|
|`org.junit.jupiter.params.aggregator`|`ArgumentAccessException`|`クラス`|
|`org.junit.jupiter.params.aggregator`|`ArgumentsAccessor`|`インターフェイス`|
|`org.junit.jupiter.params.aggregator`|`ArgumentsAggregationException`|`クラス`|
|`org.junit.jupiter.params.aggregator`|`ArgumentsAggregator`|`インターフェイス`|
|`org.junit.jupiter.params.converter`|`ArgumentConversionException`|`クラス`|
|`org.junit.jupiter.params.converter`|`ArgumentConverter`|`インターフェイス`|
|`org.junit.jupiter.params.converter`|`ConvertWith`|`アノテーション`|
|`org.junit.jupiter.params.converter`|`JavaTimeConversionPattern`|`アノテーション`|
|`org.junit.jupiter.params.converter`|`SimpleArgumentConverter`|`クラス`|
|`org.junit.jupiter.params.provider`|`Arguments`|`インターフェイス`|
|`org.junit.jupiter.params.provider`|`ArgumentsProvider`|`インターフェイス`|
|`org.junit.jupiter.params.provider`|`ArgumentsSource`|`アノテーション`|
|`org.junit.jupiter.params.provider`|`ArgumentsSources`|`アノテーション`|
|`org.junit.jupiter.params.provider`|`CsvFileSource`|`アノテーション`|
|`org.junit.jupiter.params.provider`|`CsvSource`|`アノテーション`|
|`org.junit.jupiter.params.provider`|`EnumSource`|`アノテーション`|
|`org.junit.jupiter.params.provider`|`MethodSource`|`アノテーション`|
|`org.junit.jupiter.params.provider`|`ValueSource`|`アノテーション`|
|`org.junit.jupiter.params.support`|`AnnotationConsumer`|`インターフェイス`|
|`org.junit.platform.engine.discovery`|`ModuleSelector`|`クラス`|

## 8.3. @API ツールサポート
[`@API Guardian`](https://github.com/apiguardian-team/apiguardian)は、[`@API`](https://apiguardian-team.github.io/apiguardian/docs/current/api/)の付与されたAPIの開発者と利用者のためのツールサポートを提供することを計画しています。例えば、ツールサポートは、JUnit APIが`@API`アノテーションの宣言に応じて使われているかを確認する手段を提供しそうです。
