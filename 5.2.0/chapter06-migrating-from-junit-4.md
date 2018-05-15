# JUnit 4からの移行
JUnit JUpiterのプログラミングモデルと拡張モデルは、JUnit 4の`Rules`や`Runners`といった特徴を元々サポートしていませんが、ソースコード保守運用者が彼らの全ての既存のテスト、テスト拡張、そしてカスタムビルド・テストインフラをJUnit JUpiterに移行するためにアップデートする必要があることは期待していません。

その代わりに、JUnitは、JUnit Platformのインフラを用いてJUnit 3とJUnit 4ベースのテストを実行できる*JUnit Vintageテストエンジン*を通して、穏やかな移行経路を提供しています。JUnit JUpiterに特化したクラスとアノテーションは全て新しい`org.junit.jupiter`ベースパッケージ以下にあるので、JUnit 4とJUnit Jupiterをクラスパス上に両方持つことはいかなるコンフリクトも起こしません。そのため、JUnit JUpiterテストと一緒に既存のJUnit 4のテストを保守することは安全です。さらに、JUnitチームはJUnit 4.xのための保守とバグ修正リリースを提供し続けるので、開発者は、スケジュール上にJUnit Jupiter移行のための時間が多くあります。

## 6.1. JUnit Platform上でのJUnit 4テストの実行
`junit-vintage-engine`が実行時クラスパスにあるか確認するだけです。そのような場合、JUnit 3とJUnit 4のテストが、自動的にJUnit Platformラウンチャーによって拾われます。

GradleとMavenでどう行うかは、[`jnit5-samples`](https://github.com/junit-team/junit5-samples)内のプロジェクト例をご覧ください。

### 6.1.1. カテゴリのサポート
`@Category`が付与されたテストクラスとテストメソッドに関して、*JUnit Vintageテストエンジン*はカテゴリの完全修飾クラス名を対応するテスト識別子のタグとして扱います。例えば、もしテストメソッドに`@Category(Example.class)`が付与されている場合、`"com.acme.Example"`のタグが付与されます。JUnit 4の`Categories`ランナーと同じように、この情報は発見されたテストが実行される前にフィルタリングされるのに使われます（詳細については[テストを実行する]()をご覧ください）。

## 移行のためのTips
次の内容は、既存のJUnit 4のテストをJUnit Jupiterに移行する際に気をつけるべきことです。

- `org.junit.jupiter.api`パッケージ内のアノテーション
- `org.junit.jupiter.api.Assertions`内のアサーション
- `org.junit.jupiter.api.Assumptions`内のアサンプション
- `@Before`と`@After`はもうありません；代わりに`@BeforeEach`と`@AfterEach`を使ってください。
- `@BeforeClass`と`@AfterClass`はもうありません；代わりに`@BeforeAll`と`@AfterAll`を使ってください。
- `@Ignore`はもうありません；代わりに`@Disabled`を使ってください。
- `@Category`はもうありません；代わりに`@Tag`を使ってください。
- `@RunWith`はもうありません；代わりに`@ExtendWith`を使ってください。
- `@Rule`と`@ClassRule`はもうありません；`@ExtendWith`によって置換されました；個別のルールのサポートについては次のセクションをご覧ください。

## 6.3. 限定的なJUnit 4のルールサポート
上で述べたように、JUnit Jupiterは、JUnit 4のルールを元々サポートしていませんし、これからもしません。しかしながら、JUnitチームは、多くの組織、特に大きい組織、がカスタムルールを利用した巨大なJUnit 4のコードを持っていることを理解しています。これらの組織に役目を果たし、段階的な移行を可能にするために、JUnitチームは、いくつかの選ばれたJUnit 4のルールを言葉通りJUnit Jupiter内でサポートすることを決めました。このサポートは、アダプタベースで行われ、JUnit Jupiterの拡張モデルと意味的に互換なルール、つまりテスト全体の実行フローを完全には変更しないもの、に限定されています。

JUnit Jupiterの`junit-jupiter-migrationsupport`モジュールは、現在のところ、次の3つの`Rule`型とそれらのサブクラスをサポートしています。

- `org.junit.rules.ExternalResource`（`org.junit.rules.TemporaryFolder`を含む）
- `org.junit.rules.Verifier`（`org.junit.rules.ErrorCollector`を含む）
- `org.junit.rules.ExpectedException`

JUnit 4と同様に、Ruleが付与されたフィールドもメソッド同様にサポートされています。これらを使うことで、レガシーコード内の`Rule`実装のようなテストクラスに対するクラスレベルの拡張を、JUnit 4のルールのインポート文含めて*変更なしで残す*ことができます。

この限られた形での`Rule`サポートは、クラスレベル・アノテーション`org.junit.jupiter.migrationsupport.rules.EnableRuleMigrationSupport`によって切り替えることができます。このアノテーションは、組み合わせアノテーションで、全ての意向サポート拡張を有効にします：`VerifierSupport`と`ExternalResourceSupport`、そして`ExpectedExceptionSupport`。

しかしながら、JUnit 5に対する新たな拡張を開発する際には、JUnit 4のルールベースのモデルの代わりに、JUnit Jupiterの新しい拡張モデルを使ってください。

> ⚠️ Junit Jupiter内におけるJUnit 4の`Rule`サポートは現在のところ、*実験的な*特徴です。詳細については、[実験的なAPI]()をご覧ください。
