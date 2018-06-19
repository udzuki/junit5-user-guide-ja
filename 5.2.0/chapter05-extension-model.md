# 5. 拡張モデル
## 5.1. 概要
JUnit 4において競合している`Runner`と`@Rule`、`@ClassRule`の拡張ポイントとは対照的に、JUnit Jupiterの拡張モデルは単一の包括的なコンセプト：`Extension` APIからなっています。しかしながら、`Extension`自体はただのマーカーインターフェイスであることに注意してください。

## 5.2. 拡張の登録
拡張は、[`@ExtendWith`]()を通して*宣言的に*、または[`@RegisterExtension`]()を通して*プログラム的に*、Javaの[`ServiceLoader`]()機構を通して*自動的に*登録されます。

### 5.2.1. 宣言的な拡張登録
テストインターフェイスまたはテストクラス、テストメソッド、カスタム[合成アノテーション]()に`@ExtendWith(…​)`を付与し、登録する拡張のクラス参照を供給することで、1つ以上の拡張を*宣言的に*登録できます。

例えば、特定のテストメソッドのためにカスタム`RandomParametersExtension`を登録するには、次のようにテストメソッドにアノテーションを付与します。

```java
@ExtendWith(RandomParametersExtension.class)
@Test
void test(@Random int i) {
    // ...
}
```

特定のクラスとそのサブクラスの全てのテストにカスタム`RandomParametersExtension`を登録するには、次のようにテストクラスにアノテーションを付与します。

```java
@ExtendWith(RandomParametersExtension.class)
class MyTests {
    // ...
}
```

複数の拡張はこのように一緒に登録できます。

```java
@ExtendWith({ FooExtension.class, BarExtension.class })
class MyFirstTests {
    // ...
}
```

他には、複数の拡張はこのように分けて登録できます。

```java
@ExtendWith(FooExtension.class)
@ExtendWith(BarExtension.class)
class MySecondTests {
    // ...
}
```

> ℹ️ *拡張登録の順序* `@ExtendWith`を通した宣言的に登録された拡張は、ソースコード内で宣言された順番で実行されます。例えば、`MyFirstTests`と`MySecondTests`のテストの実行は、`FooExtension`と`BarExtension`によって、*正確にこの順番で*拡張されます。

### 5.2.2. プログラム的な拡張登録
テストクラスのフィールドに[`@RegisterExtension`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/RegisterExtension.html)を付与することで、*プログラム的に*拡張を登録できます。

[`@ExtendWith`]()を通して*宣言的に*拡張を登録した場合、概してアノテーション通りにのみ設定できます。それに対して、`@RegisterExtension`を通して拡張を登録した場合、*プログラム的に*拡張することができます。例えば、拡張のコンストラクタや静的なファクトリメソッド、ビルダーAPIに引数を渡すために使うことができます。

> ℹ️ `@RegisterExtension`フィールドは、`private`や`null`（評価時）であってはなりませんが、`static`であっても非静的であっても構いません。

#### 静的なフィールド
`@RegisterExtension`フィールドが`static`である場合、`@ExtendWith`を通したクラスレベルでの拡張が登録された後に、拡張は登録されます。そのような`静的な拡張`は、拡張API内で実装できることを制限するものではありません。静的なフィールドを通して登録された拡張はそのため、`BeforeEachCallback`のようなメソッドレベルの拡張APIと同様に、`BeforeAllCallback`や`AfterAllCallback`、`TestInstancePostProcessor`のようなクラスレベルとインスタンスレベルの拡張APIを実装します。

次の例では、テストクラスの`server`フィールドが、`WebServerExtension`によってサポートされているビルダーパターンを用いてプログラム的に初期化されます。設定された`WebServerExtension`は、クラスレベルの拡張として自動的に登録されます。例えば、クラスの全てのテストの実行前にサーバを起動し、実行完了後にサーバを停止するためです。さらに、`@BeforeEach`や`@AfterEach`、`@Test`メソッドだけでなく、`@BeforeAll`や`@AfterAll`が付与された静的なライフサイクルメソッドは、必要であれば、`server`フィールドを通して拡張のインスタンスにアクセスできます。

*静的なフィールドを通して登録された拡張*
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
`@RegisterExtension`フィールドが非静的（つまり、インスタンスフィールド）である場合、テストクラスが初期化された後、さらに、登録された各`TestInstancePostProcessor`がテストインスタンスを事後処理されてから（使われる拡張のインスタンスをアノテーションが付与されたフィールドに潜在的に挿入します）、拡張は登録されます。そのため、そのような*インスタンス拡張*が`BeforeAllCallback`や`AfterAllCallback`、`TestInstancePostProcessor`のようなクラスレベル、もしくはインスタンスレベルの拡張APIを実装している場合、それらのAPIは評価されません。デフォルトでは、`@ExtendWith`を通したメソッドレベルで登録された拡張の*後に*、インスタンス拡張は登録されます。しかしながら、テストクラスが`@TestInstance(Lifecycle.PER_CLASS)`セマンティックで設定されている場合、`@ExtendWith`を通したメソッドレベルの拡張が登録される*前に*、インスタンス拡張は登録されます。

次の例では、テストクラスの`docs`フィールドが、カスタム`lookUpDocsDir()`メソッドを呼び出し、その結果を`DocumentationExtension`の静的な`forPath()`ファクトリメソッドに供給することで、プログラム的に初期化されます。設定された`DocumentationExtension`は、メソッドレベルの拡張として自動的に登録されます。さらに、`@BeforeEach`や`@AfterEach`、`@Test`メソッドは、必要であれば、`docs`フィールドを通して拡張のインスタンスにアクセスできます。

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
アノテーションを用いた[宣言的な拡張登録]()と[プログラム的な拡張登録]()のサポートに加えて、JUnit JupiterはJavaの`java.util.ServiceLoader`機構を通じた*グローバル拡張登録*もサポートしており、サードパーティによる拡張を自動で検出し、クラスパス上で利用可能なものを元に自動で登録します。

特に、カスタム拡張は、同封するJARファイルの中で`/META-INF/services`フォルダ内の`org.junit.jupiter.api.extension.Extension`という名前のファイルで完全修飾クラス名を供給することによって、登録することができます。

#### 自動拡張検出の有効化
自動検出は先進的な特徴で、そのためデフォルトでは有効化されていません。有効化するには、単に`junit.jupiter.extensions.autodetection.enabled`*設定パラメータ*に`true`をセットするだけです。これは、JVMシステムプロパティや`Launcher`に渡される`LauncherDiscoveryRequest`内の*設定パラメータ*として、またはJUnit Platform設定ファイル（詳細は[設定パラメータ]()をご覧ください）を通しても供給することが可能です。

例えば、拡張の自動検出を有効にするには、JVMを次のシステムプロパティで起動します。

`-Djunit.jupiter.extensions.autodetection.enabled=true`

自動検出が有効化されている場合、JUnit JUpiterのグローバル拡張（つまり`TestInfo`や`TestReporter`などへのサポート）の後に、`ServiceLoader`機構を通して発見された拡張は拡張レジストリに追加されます。

### 5.2.4. 拡張の継承
登録された拡張は、トップダウンに従ってテストクラス階層の中で継承されます。同様に、クラスレベルで登録された拡張は、メソッドレベルで継承されます。さらに、特定の拡張実装は、ある拡張コンテキストと親コンテキストに対して1度だけ登録されます。その結果として、重複する拡張実装の登録は無視されます。

## 5.3. 条件付きテスト実行
[ExecutionCondition](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ExecutionCondition.html)は、プログラム的な*条件付きテスト実行*のための`Extension` APIを定義しています。

`ExecutionCondition`は、各コンテナ（つまり、テストクラス）が含む全てのテストを実行すべきか、供給された`ExecutionContext`に則って決定するために*評価*されます。同じように`ExecutionCondition`は、各テストごとに供給された`ExecutionContext`に則って実行されるか決定するために*評価*されます。

複数の`ExecutionCondition`が登録されている場合、コンテナもしくはテストは、条件のうち1つでも*disabled*を返した瞬間に無効化されます。そのため、条件が評価されるかどうかの保証はありません。なぜなら、他の条件が既にコンテナもしくはテストを無効化している可能性があるからです。つまり、評価は短絡ブーリアンORオペレータのように作動します。

具体的な例については、[`DisabledCondition`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/DisabledCondition.java)と[`@Disabled`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/Disabled.html)のソースコードをご覧ください。

### 5.3.1. 条件の無効化
時には、いくつかの条件を有効に*しないで*テストスイートを実行することが役に立ちます。例えば、`@Disabled`がたとえ付与されていたとしても、それらが*壊れていないか*確認するために実行したいかもしれません。このためには、単に`junit.jupiter.conditions.deactivate`*設定パラメータ*に現在のテスト実行でどの条件を無効化する（つまり、評価しない）かを指定するパターンを提供するだけです。パターンは、JVMシステムプロパティや、`Launcher`に渡される`LauncherDiscoveryRequest`内の*設定パラメータ*として、もしくはJUnit Platform設定ファイル（詳細は[設定パラメータ]()をご覧ください）を通して供給することができます。

例えば、JUnitの`@Disabled`条件を無効化するには、次のシステムプロパティでJVMを起動します。

`-Djunit.jupiter.conditions.deactivate=org.junit.*DisabledCondition`

#### パターンマッチングの構文
`junit.jupiter.conditions.deactivate`パターンが、アスタリスク（`*`）のみで構成されている場合、全ての条件が無効化されます。そうでない場合、パターンは、各登録された条件の完全修飾クラス名（*FQCN*）とマッチしているかの判断に使われます。パターン内の全てのドット（`.`）は、FQCN内のドット（`.`）またはドル（`$`）とマッチします。全てのアスタリスク（`*`）は、FQCN内の1つ以上の文字とマッチします。パターン内の他の全ての文字は、FQCNと1対1でマッチします。

例：
- `*`：全ての条件を無効化します。
- `org.junit.*`：`org.junit`パッケージとサブパッケージ以下の全ての条件を無効化します。
- `*.MycCondition`：クラス名が`MyCondition`のものを無効化します。
- `*System*`：クラス名に`System`を含むものを無効化します。
- `org.example.MyCondition`：FQCNが`org.example.MyCondition`のものを無効化します。

## 5.4. テストインスタンスの事後処理
[`TestInstancePostProcessor`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/TestInstancePostProcessor.html)は、テストインスタンスを*事後処理*したい`Extensions`のAPIを定義しています。

一般的なユースケースには、テストインスタンスに依存関係を挿入したり、テストインスタンス上でカスタム初期化メソッドを呼び出すことなどが含まれます。

具体的な例については、[`MockitoExtension`](https://github.com/mockito/mockito/blob/release/2.x/subprojects/junit-jupiter/src/main/java/org/mockito/junit/jupiter/MockitoExtension.java)と[`SpringExtension`](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java)のソースコードをご覧ください。

## 5.5. パラメータの解決
[`ParameterResolver`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ParameterResolver.html)は、実行時にパラメータを動的に解決するための`Extension`APIを定義しています。

テストコンストラクタまたは`@Test`や`@RepeatedTest`、`@ParameterizedTest`、`@TestFactory`、`@BeforeEach`、`@AfterEach`、`@BeforeAll`、`@AfterAll`メソッドがパラメータを受け入れている場合、パラメータは`ParameterResolver`によって実行時に*解決される*必要があります。`ParameterResolver`は、組み込みもの（[`TestInfoParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestInfoParameterResolver.java)をご覧ください）まはた[ユーザが登録したもの]()を使うことができます。一般的には、パラメータは*名前*や*型*、*アノテーション*、それらの組み合わせで解決されます。具体的な例については、[`CustomTypeParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/test/java/org/junit/jupiter/engine/execution/injection/sample/CustomTypeParameterResolver.java)と[`CustomAnnotationParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/test/java/org/junit/jupiter/engine/execution/injection/sample/CustomAnnotationParameterResolver.java)のソースコードをご覧ください。

> ⚠️ JDK 9より前のバージョンのJDKの`javac`によって生成されるバイトコード内のバグによって、コアの`java.lang.reflect.Parameter`APIを通したパラメータに対するアノテーションの直接の探索は、*内部クラス*のコンストラクタ（例えば、`@Nested`テストクラス内のコントラクタ）は常に失敗します。
>
> `ParameterResolver`実装に供給されている[`ParameterContext`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ParameterContext.html)APIは、そのため、パラメータ上のアノテーションの正しい探索のために、次の便利なメソッドを含んでいます。拡張に関する開発者（Extension Authors）は、JDK内のこのバグを避けるために、`java.lang.reflect.Parameter`内で提供されているメソッドの代わりに、これらのメソッドを使うことが強く奨励されています。
- `boolean isAnnotated(Class<? extends Annotation> annotationType)`
- `Optional<A> findAnnotation(Class<A> annotationType)`
- `List<A> findRepeatableAnnotations(Class<A> annotationType)`

## 5.6. テストライフサイクルのコールバック
次のインターフェイスは、テスト実行ライフサイクルにおいて様々なポイントでテストを拡張するためのAPIを定義しています。実例に関しては次の章を、さらなる詳細については[`org.junit.jupiter.api.extension`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/package-summary.html)パッケージ内の各インターフェイスのJavadocをご覧ください。

- [`BeforeAllCallback`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/BeforeAllCallback.html)
  - [`BeforeEachCallback`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/BeforeEachCallback.html)
    - [`BeforeTestExecutionCallback`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/BeforeTestExecutionCallback.html)
    - [`AfterTestExecutionCallback`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/AfterTestExecutionCallback.html)
  - [`AfterEachCallback`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/AfterEachCallback.html)
- [`AfterAllCallback`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/AfterAllCallback.html)

> ℹ️ *複数の拡張APIsの実装*  拡張に関する開発者は、これらのインターフェイスのうちいくつかを1つの拡張内に実装できます。具体的な例については、[`SpringExtension`](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java)のソースコードをご覧ください。

### 5.6.1. BeforeとAfterのテスト実行コールバック
[`BeforeAllCallback`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/BeforeAllCallback.html)と[`AfterAllCallback`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/AfterAllCallback.html)はそれぞれ、テストメソッドが実行される*直前*と*直後*に実行される振る舞いを追加するための`Extension`APIを定義しています。そのように、これらのコールバックは、タイミングよく跡を追う似たようなユースケースによく適しています。`@BeforeEach`や`@AfterEach`メソッドの*周り*で呼び出されるコールバックを実装する必要がある場合、代わりに`BeforeEachCallback`と`AfterEachCallback`を実装してください。

次の例は、これらのコールバックを使ってテストメソッドの実行時間を計算しログする方法を示しています。`TimingExtension`は、テスト実行の時間計測とログのために`BeforeTestExecutionCallback`と`AfterTestExecutionCallback`を実装しています。

*テストメソッドの実行を時間計測してログする拡張*
```java
import java.lang.reflect.Method;
import java.util.logging.Logger;

import org.junit.jupiter.api.extension.AfterTestExecutionCallback;
import org.junit.jupiter.api.extension.BeforeTestExecutionCallback;
import org.junit.jupiter.api.extension.ExtensionContext;
import org.junit.jupiter.api.extension.ExtensionContext.Namespace;
import org.junit.jupiter.api.extension.ExtensionContext.Store;

public class TimingExtension implements BeforeTestExecutionCallback, AfterTestExecutionCallback {

    private static final Logger logger = Logger.getLogger(TimingExtension.class.getName());

    private static final String START_TIME = "start time";

    @Override
    public void beforeTestExecution(ExtensionContext context) throws Exception {
        getStore(context).put(START_TIME, System.currentTimeMillis());
    }

    @Override
    public void afterTestExecution(ExtensionContext context) throws Exception {
        Method testMethod = context.getRequiredTestMethod();
        long startTime = getStore(context).remove(START_TIME, long.class);
        long duration = System.currentTimeMillis() - startTime;

        logger.info(() -> String.format("Method [%s] took %s ms.", testMethod.getName(), duration));
    }

    private Store getStore(ExtensionContext context) {
        return context.getStore(Namespace.create(getClass(), context.getRequiredTestMethod()));
    }

}
```
`TimingExtensionTests`クラスは、`@ExtendWith`を通して`TimingExtension`を登録しているので、そのテストは実行時にこの時間計測が適用されます。

*TimingExtensionを用いるテストクラス*
```java
@ExtendWith(TimingExtension.class)
class TimingExtensionTests {

    @Test
    void sleep20ms() throws Exception {
        Thread.sleep(20);
    }

    @Test
    void sleep50ms() throws Exception {
        Thread.sleep(50);
    }

}
```

次にあるのは、`TimingExtensionTests`が実行された時に生成されたログの例です。

```
INFO: Method [sleep20ms] took 24 ms.
INFO: Method [sleep50ms] took 53 ms.
```

## 5.7. 例外処理
[`TestExecutionExceptionHandler`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/TestExecutionExceptionHandler.html)は、テスト実行中に投げられた例外を処理する`Extensions`のためのAPIを提供しています。

次の例は、`IOException`の全インスタンスを飲み込み、他の型の例外を投げ直す拡張を示しています。

*例外を扱う拡張*
```java
public class IgnoreIOExceptionExtension implements TestExecutionExceptionHandler {

    @Override
    public void handleTestExecutionException(ExtensionContext context, Throwable throwable)
            throws Throwable {

        if (throwable instanceof IOException) {
            return;
        }
        throw throwable;
    }
}
```
# 5.8. テストテンプレートに関する呼び出しコンテキストの提供
[`@TestTemplate`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/TestTemplate.html)メソッドは、少なくとも1つの[`TestTemplateInvocationContextProvider`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html)が登録されているときにだけ実行されます。各プロバイダは、[`TestTemplateInvocationContext`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/TestTemplateInvocationContext.html)インスタンスの`Stream`を提供する責務を担っています。各コンテキストは、カスタム表示名と[`@TestTemplate`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/TestTemplate.html)メソッドの次の呼び出しのために使われる追加的な拡張のリストを特定します。

次の例は、テストテンプレートの書き方だけでなく、[`TestTemplateInvocationContextProvider`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html)の登録・実装の方法も示しています。

*付随する拡張を持つテストテンプレート*

```java
@TestTemplate
@ExtendWith(MyTestTemplateInvocationContextProvider.class)
void testTemplate(String parameter) {
    assertEquals(3, parameter.length());
}

public class MyTestTemplateInvocationContextProvider implements TestTemplateInvocationContextProvider {
    @Override
    public boolean supportsTestTemplate(ExtensionContext context) {
        return true;
    }

    @Override
    public Stream<TestTemplateInvocationContext> provideTestTemplateInvocationContexts(ExtensionContext context) {
        return Stream.of(invocationContext("foo"), invocationContext("bar"));
    }

    private TestTemplateInvocationContext invocationContext(String parameter) {
        return new TestTemplateInvocationContext() {
            @Override
            public String getDisplayName(int invocationIndex) {
                return parameter;
            }

            @Override
            public List<Extension> getAdditionalExtensions() {
                return Collections.singletonList(new ParameterResolver() {
                    @Override
                    public boolean supportsParameter(ParameterContext parameterContext,
                            ExtensionContext extensionContext) {
                        return parameterContext.getParameter().getType().equals(String.class);
                    }

                    @Override
                    public Object resolveParameter(ParameterContext parameterContext,
                            ExtensionContext extensionContext) {
                        return parameter;
                    }
                });
            }
        };
    }
}
```

この例では、テストテンプレートは2回呼び出されます。呼び出しの表示名は、呼び出しコンテキストによって決められたように"foo"と"bar"になります。各呼び出しは、メソッドパラメータを解決するためのカスタム[`ParameterResolver`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ParameterResolver.html)を登録します。`ConsoleLauncher`を使った時の出力は次のようになります。

```
└─ testTemplate(String) ✔
   ├─ foo ✔
   └─ bar ✔
```

[`TestTemplateInvocationContextProvider`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html)拡張APIは、異なるコンテキストでテストのようなメソッドを繰り返し呼び出すことを当てにしている別種のテストを実装することを第一に意図しています。例えば、異なるパラメータを用いたり、テストクラスインスタンスを異なる方法で準備したり、コンテキストを修正せずに複数回行うなどです。機能に提供ためにこの拡張ポイントを使う[`繰り返しテスト`]()または[`パラメータ化テスト`]()の実装を参照してください。

## 5.9. 拡張での状態の保持
通常、拡張は一度だけインスタンス化されます。そのため、関連する質問が挙がります：拡張のある呼び出しからの状態は、どのようにして次の呼び出しに保持するのか？`ExtensionContext`APIは、まさにこの目的のための`Store`を提供します。拡張は、後からの収集のためにストアに値を入れます。メソッドレベルのスコープでの`Store`の使用例は、[`タイミング拡張`]()をご覧ください。テスト実行中に`ExtensionContext`に貯蔵された値は、周囲の`ExtensionContext`では利用できないことに注意してください。`ExtensionContext`はネストされているかもしれないので、内部コンテキストのスコープもまた限定的となっています。[`Store`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ExtensionContext.Store.html)を通した値の貯蔵と収集に利用可能なメソッドの詳細については、対応するJavaDocをご覧ください。

> ℹ️ `ExtensionContext.Store.CloseableResource` 拡張コンテキストのストアは、その拡張コンテキストのライフサイクルにバインドされています。拡張コンテキストのライフサイクルが終了するとき、関連するストアも閉じます。全ての貯蔵された値は、`CloseableResource`のインスタンスで、`close()`メソッドの呼び出しによって通知されます。

## 5.10. 拡張でサポートしているユーティリティ
`junit-platform-commons`アーティファクトは、アノテーションやクラス、リフレクション、タスクをスキャンするクラスパスに作動する*保守された*ユーティリティメソッドを含む[`org.junit.platform.commons.support`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/commons/support/package-summary.html)という名前のパッケージを公開しています。`TestEngine`と`Extension`の開発者は、JUnit Platformの動作に合わせるため、これらサポートされたメソッドを利用することが推奨されています。

### 5.10.1. アノテーションのサポート
`AnnotationSupport`は、静的なユーティリティメソッドを提供しており、アノテーションの付与された要素（つまり、パッケージやアノテーション、クラス、インターフェイス、メソッド、フィールド）を操作できます。これらは、ある要素に特定のアノテーションが付与またはメタ付与されているか確認したり、特定のアノテーションを検索したり、クラスやインターフェイス内でアノテーション付与されているメソッドとフィールドを探し出すためのメソッドが含まれています。これらのメソッドのいくつかは、実装されたインターフェイスとクラス階層内をアノテーションを見つけるために捜索します。さらなる詳細については、[`AnnotationSupport`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/commons/support/AnnotationSupport.html)のJavaDocをご覧ください。

### 5.10.2. クラスのサポート
`ClassSupport`は、クラス（つまり、`java.lang.Class`のインスタンス）に作動する静的なユーティリティメソッドを提供しています。さらなる詳細については、[`ClassSupport`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/commons/support/ClassSupport.html)のJavaDocをご覧ください。

### 5.10.3. リフレクションのサポート
`ReflectionSupport`は、標準のJDKリフレクションとクラス読み込み機構を増強するための静的なユーティリティメソッドを提供しています。これらは、特定の述語にマッチするクラスを探すためにクラスパスをスキャンしたり、クラスの新しいインスタンスを読み込んで生成したり、メソッドを見つけて呼び出すためのメソッドを含みます。これらのメソッドのいつくかは、マッチするメソッドの位置を特定するためにクラス階層を走査します。さらなる詳細については、[`ReflectionSupport`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/commons/support/ReflectionSupport.html)のJavaDocをご覧ください。

## 5.11. ユーザコードと拡張の相対実行順序
1つ以上のテストメソッドを含むテストクラスを実行する時、ユーザの提供するテストとライフサイクルに関するメソッドに加えて、いくつかの拡張コールバックが呼び出されます。次の図は、ユーザ提供のコードと拡張コードの相対順序を示しています。

![](https://junit.org/junit5/docs/5.2.0/user-guide/images/extensions_lifecycle.png)

*ユーザコードと拡張コード*

ユーザ提供のテストとライフサイクルに関するメソッドはオレンジで示され、拡張によって提供されるコールバックは青で示されています。灰色のボックスは、1つのテストメソッドの実行を意味しており、テストクラスの全てのテストメソッドに対して繰り返されます。

次の表は、[ユーザコードと拡張コード]()の図式内の12のステップについて、より詳しく説明をしています。

|ステップ|インターフェイス/アノテーション|説明|
|:-:|:-:|:-:|
|1|インターフェイス `org.junit.jupiter.api.extension.BeforeAllCallback`|コンテナの全てのテストが実行される前に実行される拡張コード|
|2|アノテーション `org.junit.jupiter.api.BeforeAll`|コンテナの全てのテストが実行される前に実行されるユーザコード|
|3|インターフェイス `org.junit.jupiter.api.extension.BeforeEachCallback`|各テストが実行される前に実行される拡張コード|
|4|アノテーション `org.junit.jupiter.api.BeforeEach`|各テストが実行される前に実行されるユーザコード|
|5|インターフェイス `org.junit.jupiter.api.extension.BeforeTestExecutionCallback`|テストが実行される直前に実行される拡張コード|
|6|アノテーション `org.junit.jupiter.api.Test`|実際のテストメソッドとなるユーザコード|
|7|インターフェイス `org.junit.jupiter.api.extension.TestExecutionExceptionHandler`|テスト中に投げられる例外を扱う拡張コード|
|8|インターフェイス `org.junit.jupiter.api.extension.AfterTestExecutionCallback`|テストと対応する例外ハンドラが実行された直後に実行される拡張コード|
|9|アノテーション `org.junit.jupiter.api.AfterEach`|各テストが実行された後に実行されるユーザコード|
|10|インターフェイス `org.junit.jupiter.api.extension.AfterEachCallback`|各テストが実行された後に実行される拡張コード|
|11|アノテーション `org.junit.jupiter.api.AfterAll`|コンテナの全てのテストが実行された後に実行されるユーザコード|
|12|インターフェイス `org.junit.jupiter.api.extension.AfterAllCallback`|コンテナの全てのテストが実行された後に実行される拡張コード|

最も単純なケースでは、実際のテストメソッドのみが実行されます（ステップ 6）。他の全てのステップはオプションで、対応するライフサイクルコールバックをサポートするユーザコードまたは拡張コードの存在に依存します。様々なライフサイクルコールバックのさらなる詳細については、各アノテーションと拡張それぞれのJavaDocをご覧ください。
