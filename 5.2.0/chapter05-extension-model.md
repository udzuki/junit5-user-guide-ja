# 5. 拡張モデル
## 5.1. 概要
JUnit 4における`Runner`と`@Rule`、`@ClassRule`の矛盾する拡張ポイントとは対照的に、JUnit Jupiterの拡張モデルは1つの包括的なコンセプト：`Extension` APIからなっています。しかしながら、`Extension`自体はただのマーカ・インターフェイスであることに注意してください。

## 5.2. 拡張の登録
拡張は、[`@ExtendWith`]()を通して*宣言的に*か、[`@RegisterExtension`]()を通して*プログラム的に*、もしくはJavaの[`ServiceLoader`]()メカニズムを通して*自動的に*登録されます。

### 5.2.1. 宣言的な拡張登録
開発者は、テストインターフェイスまたはテストクラス、テストメソッド、カスタム[組み合わせアノテーション]()に`@ExtendWith(…​)`を付与し、登録される拡張のクラス・レファレンスを供給することで、1つかそれ以上の拡張を*宣言的に*登録することができます。

例えば、特定のテストメソッドのためにカスタム`RandomParametersExtension`を登録するには、次のようにテストメソッドにアノテーションを付与します。

```java
@ExtendWith(RandomParametersExtension.class)
@Test
void test(@Random int i) {
    // ...
}
```

特定のクラスとそのサブクラスの全てのテストにカスタム`RandomParametersExtension`を登録するには次のようにテストクラスにアノテーションを付与します。

```java
@ExtendWith(RandomParametersExtension.class)
class MyTests {
    // ...
}
```

複数の拡張はこのように一緒に登録することができます。

```java
@ExtendWith({ FooExtension.class, BarExtension.class })
class MyFirstTests {
    // ...
}
```

他には、複数の拡張はこのように分けて登録することもできます。

```java
@ExtendWith(FooExtension.class)
@ExtendWith(BarExtension.class)
class MySecondTests {
    // ...
}
```

> :information_source: *拡張登録の順番* `@ExtendWith`を通した宣言的に登録された拡張は、ソースコード内で宣言された順番に実行されます。例えば、`MyFirstTests`と`MySecondTests`のテストの実行は、`FooExtension`と`BarExtension`によって、*正確にこの順番によって*、拡張されます。

### 5.2.2. プログラム的な拡張登録
開発者は、テストクラスのフィールドに[`@RegisterExtension`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/RegisterExtension.html)を付与することで、*プログラム的に*拡張を登録することができます。

[`@ExtendWith`]()を通して*宣言的に*拡張を登録した場合、概してアノテーション通りにのみ設定できます。それに対して、`@RegisterExtension`を通して拡張を登録した場合、*プログラム的に*拡張することができますー例えば、拡張のコンストラクタやstaticなファクトリメソッド、ビルダーAPIに複数を渡すために使うことができます。

> :information_source: `@RegisterExtension`フィールドは、`private`や`null`（評価時）であってはなりませんが、`static`であってもnon-staticであっても構いません。

#### Staticフィールド
もし`@RegisterExtension`フィールドが`static`である場合、拡張は`@ExtendWith`を通したクラスレベルの拡張が登録された後に登録されます。そのような`static拡張`は、拡張API内で実装できることを制限するものではありません。staticフィールドを通して登録された拡張はそのため、`BeforeEachCallback`のようなメソッドレベルの拡張APIと同様に、`BeforeAllCallback`や`AfterAllCallback`、`TestInstancePostProcessor`のようなクラスレベルとインスタンスレベルの拡張APIの実装します。

次の例では、テストクラスの`server`フィールドが、`WebServerExtension`によってサポートされているビルダーパターンを用いてプログラム的に初期化されます。設定された`WebServerExtension`は、クラスレベルの拡張として自動的に登録されますー例えば、クラスの全てのテストを実行前にサーバを起動し、実行完了後にサーバを停止するためです。さらに、`@BeforeEach`や`@AfterEach`、`@Test`メソッドだけでなく、`@BeforeAll`や`@AfterAll`が付与されたstaticなライフサイクル・メソッドは、必要であれば`server`フィールドを通して拡張のインスタンスにアクセスすることができます。

*staticフィールドを通して登録された拡張*
```java
class WebServerDemo {

    @RegisterExtension
    static WebServerExtension server = WebServerExtension.builder()
        .enableSecurity(false)
        .build();

    @Test
    void getProductList() {
        WebClient webClient = new WebClient();
        String serverUrl = server.getServerUrl();
        // Use WebClient to connect to web server using serverUrl and verify response
        assertEquals(200, webClient.get(serverUrl + "/products").getResponseStatus());
    }
}
```

#### インスタンスフィールド
もし`@RegisterExtension`フィールドがnon-static（つまりインスタンスフィールド）である場合、拡張はテストクラスが初期化された後に登録され、それぞれが登録された後、`TestInstancePostProcessor`によってテストインスタンスがポストプロセスされます（潜在的に使われる拡張のインスタンスがアノテーション付与されたフィールドに注入されます）。そのため、もしそのような*インスタンス拡張*が`BeforeAllCallback`や`AfterAllCallback`、または`TestInstancePostProcessor`のようなクラスレベル、もしくはインスタンスレベルの拡張APIを実装している場合、それらのAPIは評価されません。デフォルトでは、インスタンス拡張は、`@ExtendWith`を通したメソッドレベルで登録された拡張の*後に*登録されます；しかしながら、もしテストクラスが`@TestInstance(Lifecycle.PER_CLASS)`と設定されている場合、インスタンス拡張は、`@ExtendWith`を通したメソッドレベルの拡張が登録される*前に*登録されます。

次の例では、テストクラスの`docs`フィールドが、カスタム`lookUpDocsDir()`メソッドを呼び出し、その結果を`DocumentationExtension`のstaticな`forPath()`ファクトリメソッドに供給することで、プログラム的に初期化されています。設定された`DocumentationExtension`はメソッドレベルの拡張として自動的に登録されます。さらに、`@BeforeEach`や`@AfterEach`、`@Test`メソッドは、もし必要であれば`docs`フィールドを通して拡張のインスタンスにアクセスすることができます。

*インスタンスフィールドを通して登録された拡張*

```java
class DocumentationDemo {

    static Path lookUpDocsDir() {
        // return path to docs dir
    }

    @RegisterExtension
    DocumentationExtension docs = DocumentationExtension.forPath(lookUpDocsDir());

    @Test
    void generateDocumentation() {
        // use this.docs ...
    }
}
```

### 5.2.3. 自動的な拡張登録
アノテーションを用いた[宣言的な拡張登録]()と[プログラム的な拡張登録]()へのサポートに加えて、JUnit Jupiterは、Javaの`java.util.ServiceLoader`メカニズムを通じた*グローバル拡張登録*もサポートしており、サードパーティによる拡張を自動で検出し、クラスパス上で利用可能なものを元に自動で登録します。

特にカスタム拡張は、閉じたJARファイルの中で`/META-INF/services`フォルダ内の`org.junit.jupiter.api.extension.Extension`という名前のファイルで完全修飾クラス名を供給することによって、登録することができます。

#### 自動拡張検出を有効にする
自動検出は先進的な特徴で、そのためデフォルトでは有効化されていません。有効化するには、単に`junit.jupiter.extensions.autodetection.enabled`*設定パラメータ*に`true`をセットするだけです。これは、JVMシステムプロパティや`Launcher`に渡される`LauncherDiscoveryRequest`内の*設定パラメータ*として、またはJUnit Platform設定ファイル（詳細は[設定パラメータ]()をご覧ください）を通しても供給することが可能です。

例えば、拡張の自動検出を有効にするには、JVMを次のシステムプロパティで起動します。

`-Djunit.jupiter.extensions.autodetection.enabled=true`

自動検出が有効の時、`ServiceLoader`メカニズムを通して発見された拡張は、JUnit JUpiterのグローバル拡張（つまり`TestInfo`や`TestReporter`などへのサポート）の後に拡張登録に追加されます。

### 5.2.4. 拡張の継承
登録された拡張は、トップダウンに従ってテストクラス階層の中で継承されます。同様に、クラスレベルで登録された拡張は、メソッドレベルで継承されます。さらに、特定の拡張実装は、ある拡張コンテキストと親コンテキストに対して1度だけ登録されます。その結果として、重複する拡張実装の登録は無視されます。

## 5.3. 条件テスト実行
[ExecutionCondition](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ExecutionCondition.html)は、プログラム的な*条件テスト実行*のための`Extension` APIを定義しています。

`ExecutionCondition`は、各コンテナ（つまりテストクラス）で含まれている全てのテストが供給された`ExecutionContext`に則って実行されるか決定するために*評価*されます。同じように`ExecutionCondition`は、各テストごとに供給された`ExecutionContext`に則って実行されるか決定するために*評価*されます。

複数の`ExecutionCondition`が登録されている場合、コンテナもしくはテストは、条件のうち1つでも*disabled*を返した瞬間に無効化されます。そのため、条件が評価されるかどうかの保証はありません。なぜなら、他の条件が既にコンテナもしくはテストを無効化しているかもしれないからです。つまり、評価はブーリアン（boolean）のショートカットORオペレータのように作動します。

具体的な例については、[`DisabledCondition`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/DisabledCondition.java)と[`@Disabled`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/Disabled.html)のソースコードをご覧ください。

### 5.3.1. 条件の無効化
時には、いくつかの条件を有効に*しないで*テストスイートを実行することが役に立ちます。例えば、`@Disabled`がたとえ付与されていたとしても、それらが*壊れていないか*確認するために実行したいかもしれません。このためには、単に`junit.jupiter.conditions.deactivate`*設定パラメータ*に現在のテスト実行でどの条件を無効化（つまり評価しない）するかを指定するパターンを提供するだけです。パターンは、JVMシステムプロパティや、`Launcher`に渡される`LauncherDiscoveryRequest`内の*設定パラメータ*として、もしくはJUNit Platform設定ファイル（詳細は[設定パラメータ]()をご覧ください）を通して供給することができます。

例えば、JUnitの`@Disabled`条件を無効化するには、次のシステムプロパティでJVMを起動します。

`-Djunit.jupiter.conditions.deactivate=org.junit.*DisabledCondition`

#### パターンマッチング・シンタックス
もし`junit.jupiter.conditions.deactivate`パターンが、アスタリスク（`*`）のみで構成されている場合、全ての条件が無効化されます。そうでない場合、パターンは、各登録された条件の完全修飾クラス名（*FQCN*）とマッチしているかの判断に使われます。パターン内の全てのドット（`.`）は、FQCN内のドット（`.`）かドル（`$`）とマッチします。全てのアスタリスク（`*`）は、FQCN内の1つ以上の文字とマッチします。パターン内の他の全ての文字は、FQCNと1対1でマッチします。

例：
- `*`：全ての条件を無効化します。
- `org.junit.*`：`org.junit`パッケージとサブパッケージ以下の全ての条件を無効化します。
- `*.MycCondition`：クラス名が`MyCondition`のものを無効化します。
- `*System*`：クラス名に`System`を含むものを無効化します。
- `org.example.MyCondition`：FQCNが`org.example.MyCondition`のものを無効化します。

## 5.4. テストインスタンス・ポストプロセス
[`TestInstancePostProcessor`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/TestInstancePostProcessor.html)は、テストインスタンスを*ポストプロセス*したい`Extensions`のAPIを定義しています。

一般的なユースケースには、テストインスタンスに依存関係を注入したり、テストインスタンス上でカスタム初期化メソッドを呼び出すことなどが含まれます。

具体的な例については、[`MockitoExtension`](https://github.com/mockito/mockito/blob/release/2.x/subprojects/junit-jupiter/src/main/java/org/mockito/junit/jupiter/MockitoExtension.java)と[`SpringExtension`](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java)のソースコードをご覧ください。

## 5.5. パラメータ解決
[`ParameterResolver`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ParameterResolver.html)は、動的に実行時にパラメータを解決するための`Extension`APIを定義しています。

もしテストコンストラクタか、`@Test`や`@RepeatedTest`、`@ParameterizedTest`、`@TestFactory`、`@BeforeEach`、`@AfterEach`、`@BeforeAll`、`@AfterAll`メソッドがパラメータを受け入れている場合、パラメータは、`ParameterResolver`によって実行時に*解決される*必要があります。`ParameterResolver`は、ビルトイン（[`TestInfoParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestInfoParameterResolver.java)をご覧ください）のものか、[ユーザが登録したもの]()を使うことができます。一般的に言って、パラメータは、*名前*や*型*、*アノテーション*、もしくはそれらの組み合わせで解決されます。具体的な例については、[`CustomTypeParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/test/java/org/junit/jupiter/engine/execution/injection/sample/CustomTypeParameterResolver.java)と[`CustomAnnotationParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/test/java/org/junit/jupiter/engine/execution/injection/sample/CustomAnnotationParameterResolver.java)のソースコードをご覧ください。

> ⚠️ JDK 9より前のバージョンのJDKの`javac`によって生成されるバイトコード内のバグによって、コアの`java.lang.reflect.Parameter`APIを通した直接パラメータに対するアノテーションの評価は、*インナークラス*のコンストラクタ（例えば、`@Nested`テストクラス内のコントラクタ）は常に失敗します。
>
> `ParameterResolver`実装に供給されている[`ParameterContext`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ParameterContext.html)APIは、そのため、パラメータ上のアノテーションの正しい評価のために、次の便利なメソッドを含んでいます。拡張著者（Extension Authors）は、JDK内のこのバグを避けるために`java.lang.reflect.Parameter`内で提供されているメソッドの代わりに、これらのメソッドを使うことが強く奨励されています。
- `boolean isAnnotated(Class<? extends Annotation> annotationType)`
- `Optional<A> findAnnotation(Class<A> annotationType)`
- `List<A> findRepeatableAnnotations(Class<A> annotationType)`

## 5.6. テストライフサイクル・コールバック
