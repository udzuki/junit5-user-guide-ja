# 3. テストを書く
*最初のテストケース*

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;

class FirstJUnit5Tests {

    @Test
    void myFirstTest() {
        assertEquals(2, 1 + 1);
    }

}
```

## 3.1. アノテーション
JUnit Jupiterは、テストの設定やフレームワークの拡張のために、次のアノテーションをサポートしています。

全てのコアなアノテーションは、`junit-jupiter-api`モジュール内の[`org.junit.jupiter.api`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/package-summary.html)パッケージにあります。

|アノテーション|説明|
|---|---|
|`@Test`|テストメソッドであることを意味します。JUnit 4の`@Test`と異なり、このアノテーションはいかなる属性も宣言しません。これは、JUnit Jupiterにおけるテスト拡張が専用のアノテーションを元に作動するからです。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@ParameterizedTest`|[パラメータ化テスト]()であることを意味します。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@RepeatedTest`|[繰り返しテスト]()のテンプレートメソッドであることを意味します。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@TestFactory`|[動的テスト]()のファクトリーメソッドであることを意味します。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@TestInstance`|アノテーションが付与されたテストクラスの[テストインスタンス・ライフサイクル]()を設定するために使用されます。アノテーションは*継承*されます。|
|`@TestTemplate`|[テストケースのテンプレート]()メソッドであることを意味します。テンプレートは登録された[プロバイダ]()によって返される呼び出しコンテキストの数に応じて複数回呼び出されます。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@DisplayName`|テストクラスもしくはテストメソッドのカスタム表示名を宣言します。アノテーションは*継承*されません。|
|`@BeforeEach`|現在のクラス内にある**各テスト**（`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`）が実行される*前*（*before*）に実行されるメソッドを意味します。JUnit 4の`@Before`と類似したものです。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@AfterEach`|現在のクラス内にある**各テスト**（`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`）が実行された*後*（*after*）に実行されるメソッドを意味します。JUnit 4の`@After`と類似したものです。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@BeforeAll`|現在のクラス内にある**全テスト**（`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`）が実行される*前*（*before*）に実行されるメソッドを意味します。JUnit 4の`@BeforeClass`と類似したものです。メソッドは*隠蔽*または*オーバーライド*されない限り、*継承*され、（"クラス毎に"[テストインスタンス・ライフサイクル]()を使わない限り）`static`でないといけません。|
|`@AfterAll`|現在のクラス内にある**全テスト**（`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`）が実行された*後*（*after*）に実行されるメソッドを意味します。JUnit 4の`@AfterClass`と類似したものです。メソッドは*隠蔽*または*オーバーライド*されない限り、*継承*され、（"クラス毎に"[テストインスタンス・ライフサイクル]()を使わない限り）`static`でないといけません。|
|`@Nested`|ネストされた非staticなテストクラスであることを意味します。`@BeforeAll`と`@AfterAll`メソッドは、"クラス毎に"[テストインスタンス・ライフサイクル]()を使わない限り、`@Nested`テストクラスの中では直接使うことができません。メソッドは*継承*されません。|
|`@Tag`|クラスもしくはメソッドレベルでテストをフィルタリングするための*タグ*を宣言できます。TestNGのtest groupsもしくはJUnit 4のCategoriesと類似したものです。アノテーションはクラスレベルでは*継承*されますが、メソッドレベルでは*継承*されません。|
|`@Disabled`|テストクラスもしくはテストメソッドを*無効化*できます。JUnit 4の`@Ignore`と類似したものです。アノテーションは*継承*されません。|
|`@ExtendWith`|カスタム[拡張]()を登録できます。アノテーションは*継承*されます。|

次のアノテーションをつけたメソッドは値を返してはいけません。
(`@Test`, `@TestTemplate`, `@RepeatedTest`, `@BeforeAll`, `@AfterAll`, `@BeforeEach`, `@AfterEach`)

> ⚠️ 注意点：いくつかのアノテーションは現在*実験段階*である恐れがあります。詳細に関しては、[実験的なAPIs]()をご覧ください。

### 3.1.1. メタアノテーションと合成アノテーション
JUnit Jupiterアノテーションはメタアノテーションとして使うことができます。つまり、メタアノテーションのセマンティックを自動で*継承*する独自の*合成アノテーション*を定義できます。

例えば、コードベースに`@Tag("fast")`（[タグ付けとフィルタリング]()をご覧ください。）をコピー＆ペーストする代わりに、次のように`@Fast`というカスタム*合成アノテーション*を作成できます。`@Fast`は`@Tag("fast")`の代替として利用できます。

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.junit.jupiter.api.Tag;

@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Tag("fast")
public @interface Fast {
}
```

## 3.2. テストクラスとメソッド
*テストメソッド*とは、直接もしくはメタ的に`@Test`または`@RepeatedTest`、`@ParamterizedTest`、`@TsetFactory`、`@TestTemplate`が付与されたインスタンスメソッドです。*テストクラス*とは、少なくとも1つのテストメソッドを含むトップレベルまたは静的なメンバークラスです。

*標準的なテストケース*

```java
import static org.junit.jupiter.api.Assertions.fail;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

class StandardTests {

    @BeforeAll
    static void initAll() {
    }

    @BeforeEach
    void init() {
    }

    @Test
    void succeedingTest() {
    }

    @Test
    void failingTest() {
        fail("a failing test");
    }

    @Test
    @Disabled("for demonstration purposes")
    void skippedTest() {
        // not executed
    }

    @AfterEach
    void tearDown() {
    }

    @AfterAll
    static void tearDownAll() {
    }

}
```

> ℹ️ テストクラスもテストメソッドも `public`である必要はありません。

## 3.3. 表示名
テストクラスとテストメソッドはカスタム表示名（スペースや特殊文字、絵文字さえ使用可能） を宣言できます。それらがテストランナーとテストレポートによって表示されます。

```java
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

@DisplayName("A special test case")
class DisplayNameDemo {

    @Test
    @DisplayName("Custom test name containing spaces")
    void testWithDisplayNameContainingSpaces() {
    }

    @Test
    @DisplayName("╯°□°）╯")
    void testWithDisplayNameContainingSpecialCharacters() {
    }

    @Test
    @DisplayName("😱")
    void testWithDisplayNameContainingEmoji() {
    }

}
```

## 3.4. アサーション
JUnit Jupiterには、JUnit 4のアサーションメソッドの多くを備えています。また、いくつかはJava 8のラムダ式で使うことができます。全てのJUnit Jupiterアサーションは、[`org.junit.jupiter.api.Assertions`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/Assertions.html)クラスの`static`メソッドです。

```java
import static java.time.Duration.ofMillis;
import static java.time.Duration.ofMinutes;
import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTimeout;
import static org.junit.jupiter.api.Assertions.assertTimeoutPreemptively;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.Test;

class AssertionsDemo {

    @Test
    void standardAssertions() {
        assertEquals(2, 2);
        assertEquals(4, 4, "The optional assertion message is now the last parameter.");
        assertTrue('a' < 'b', () -> "Assertion messages can be lazily evaluated -- "
                + "to avoid constructing complex messages unnecessarily.");
    }

    @Test
    void groupedAssertions() {
        // In a grouped assertion all assertions are executed, and any
        // failures will be reported together.
        assertAll("person",
            () -> assertEquals("John", person.getFirstName()),
            () -> assertEquals("Doe", person.getLastName())
        );
    }

    @Test
    void dependentAssertions() {
        // Within a code block, if an assertion fails the
        // subsequent code in the same block will be skipped.
        assertAll("properties",
            () -> {
                String firstName = person.getFirstName();
                assertNotNull(firstName);

                // Executed only if the previous assertion is valid.
                assertAll("first name",
                    () -> assertTrue(firstName.startsWith("J")),
                    () -> assertTrue(firstName.endsWith("n"))
                );
            },
            () -> {
                // Grouped assertion, so processed independently
                // of results of first name assertions.
                String lastName = person.getLastName();
                assertNotNull(lastName);

                // Executed only if the previous assertion is valid.
                assertAll("last name",
                    () -> assertTrue(lastName.startsWith("D")),
                    () -> assertTrue(lastName.endsWith("e"))
                );
            }
        );
    }

    @Test
    void exceptionTesting() {
        Throwable exception = assertThrows(IllegalArgumentException.class, () -> {
            throw new IllegalArgumentException("a message");
        });
        assertEquals("a message", exception.getMessage());
    }

    @Test
    void timeoutNotExceeded() {
        // The following assertion succeeds.
        assertTimeout(ofMinutes(2), () -> {
            // Perform task that takes less than 2 minutes.
        });
    }

    @Test
    void timeoutNotExceededWithResult() {
        // The following assertion succeeds, and returns the supplied object.
        String actualResult = assertTimeout(ofMinutes(2), () -> {
            return "a result";
        });
        assertEquals("a result", actualResult);
    }

    @Test
    void timeoutNotExceededWithMethod() {
        // The following assertion invokes a method reference and returns an object.
        String actualGreeting = assertTimeout(ofMinutes(2), AssertionsDemo::greeting);
        assertEquals("Hello, World!", actualGreeting);
    }

    @Test
    void timeoutExceeded() {
        // The following assertion fails with an error message similar to:
        // execution exceeded timeout of 10 ms by 91 ms
        assertTimeout(ofMillis(10), () -> {
            // Simulate task that takes more than 10 ms.
            Thread.sleep(100);
        });
    }

    @Test
    void timeoutExceededWithPreemptiveTermination() {
        // The following assertion fails with an error message similar to:
        // execution timed out after 10 ms
        assertTimeoutPreemptively(ofMillis(10), () -> {
            // Simulate task that takes more than 10 ms.
            Thread.sleep(100);
        });
    }

    private static String greeting() {
        return "Hello, World!";
    }

}
```

また、JUnit Jupiterのいくつかのアサーションメソッドは[Kotlin](https://kotlinlang.org/)で使うことができます。全てのJUnit Jupiter Kotlinアサーションは、`org.junit.jupiter.api`パッケージのトップレベル関数です。

```kotlin
import org.junit.jupiter.api.Test
import org.junit.jupiter.api.assertAll
import org.junit.jupiter.api.Assertions.assertEquals
import org.junit.jupiter.api.Assertions.assertTrue
import org.junit.jupiter.api.assertThrows

class AssertionsKotlinDemo {

    @Test
    fun `grouped assertions`() {
        assertAll("person",
            { assertEquals("John", person.firstName) },
            { assertEquals("Doe", person.lastName) }
        )
    }

    @Test
    fun `exception testing`() {
        val exception = assertThrows<IllegalArgumentException> ("Should throw an exception") {
            throw IllegalArgumentException("a message")
        }
        assertEquals("a message", exception.message)
    }

    @Test
    fun `assertions from a stream`() {
        assertAll(
            "people with name starting with J",
            people
                .stream()
                .map {
                    // This mapping returns Stream<() -> Unit>
                    { assertTrue(it.firstName.startsWith("J")) }
                }
        )
    }

    @Test
    fun `assertions from a collection`() {
        assertAll(
            "people with last name of Doe",
            people.map { { assertEquals("Doe", it.lastName) } }
        )
    }

}
```

### 3.4.1. サードパーティのアサーションライブラリ
JUnit Jupiterによって提供されているアサーション機能は多くのテストシナリオで十分ですが、*matchers*といったより強力で追加的な機能が求められたり必要な場合があります。そのような場合、JUnitチームは、[AssertJ](http://joel-costigliola.github.io/assertj/)や[Hamcrest](http://hamcrest.org/JavaHamcrest/)、[Truth](http://google.github.io/truth/)などといったサードパーティのアサーションライブラリの使用をお薦めします。したがって、開発者は自由に選んだアサーションライブラリを使うことができます。

例えば、*matchers*と流暢なAPI（fluent API）の組み合わせは、アサーションをよりわかりやすく、読みやすくするために使うことができます。しかしながら、JUnit Jupiterの[`org.junit.jupiter.Assertions`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/Assertions.html)クラスは、Hamcrestの[`Matcher`](http://junit.org/junit4/javadoc/latest/org/hamcrest/Matcher.html)を許容しているJUnit 4の`org.junit.jupiter.Assert`クラスにあるような`assertThat()`メソッドを提供していません。代わりに、開発者はサードパーティのアサーションライブラリによって提供されているマッチャー用の組み込みサポートを使うことが奨励されています。

次の例は、JUnit JupiterのテストにおいてHamcrestからの`assertThat()`サポートを使い方を説明しています。Hamcrestライブラリがクラスパスに加えられている限り、`assertThat()`や`is()`、`equalTo()`といったメソッドを静的にインポートできます。また、それらをテストの中で、下に示す`assertWithHamcrestMatcher()`のように使うことができます。

```java
import static org.hamcrest.CoreMatchers.equalTo;
import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

import org.junit.jupiter.api.Test;

class HamcrestAssertionDemo {

    @Test
    void assertWithHamcrestMatcher() {
        assertThat(2 + 1, is(equalTo(3)));
    }

}
```

当然、JUnit 4のプログラミングモデルに基づいたレガシーテストも`org.junit.Assert#assertThat`を用いて継続して利用可能です。

## 3.5. アサンプション
JUnit Jupiterには、JUnit 4のアサンプションメソッドのサブセットを備えています。また、いくつかはJava 8のラムダ式で使うことができます。全てのJUnit Jupiterアサンプションは、[`org.junit.jupiter.api.Assumptions`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/Assumptions.html)クラスの`static`メソッドです。

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assumptions.assumingThat;

import org.junit.jupiter.api.Test;

class AssumptionsDemo {

    @Test
    void testOnlyOnCiServer() {
        assumeTrue("CI".equals(System.getenv("ENV")));
        // remainder of test
    }

    @Test
    void testOnlyOnDeveloperWorkstation() {
        assumeTrue("DEV".equals(System.getenv("ENV")),
            () -> "Aborting test: not on developer workstation");
        // remainder of test
    }

    @Test
    void testInAllEnvironments() {
        assumingThat("CI".equals(System.getenv("ENV")),
            () -> {
                // perform these assertions only on the CI server
                assertEquals(2, 2);
            });

        // perform these assertions in all environments
        assertEquals("a string", "a string");
    }

}
```

## 3.6. テストの無効化
テストクラス全体もしくは各テストメソッドは、[`@Disabled`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/Disabled.html)アノテーションまたは[条件付きテスト実行]()で議論されているアノテーションの1つ、カスタム[`ExecutionCondition`]()によって*無効化*できます。

これは`@Disabled`テストクラスです。

```java
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

@Disabled
class DisabledClassDemo {
    @Test
    void testWillBeSkipped() {
    }
}
```


そして、これは`@Disabled`テストメソッドを含むテストクラスです。

```java
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

class DisabledTestsDemo {

    @Disabled
    @Test
    void testWillBeSkipped() {
    }

    @Test
    void testWillBeExecuted() {
    }
}
```

## 3.7. 条件付きテスト実行
JUnit Jupiterの[ExecutionCondition]()拡張APIを用いて、ある条件に基づいたコンテナまたはテストを*プログラム的に* *有効*または*無効*にできます。そのような条件の最も単純な例は、`@Disabled`をサポートしている組み込みの[`DisabledCondition`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/DisabledCondition.java)です（[テストの無効化]()をご覧ください）。`@Disabled`に加えて、JUnit Jupiterは、`org.junit.jupiter.api.condition`パッケージに他のいくつかのアノテーションベースの条件もサポートしており、コンテナやテストを*宣言的に* *有効*または*無効*にできます。詳細については、次章をご覧ください。

> 💡 合成アノテーション：次章に列挙する条件アノテーションはいずれも、カスタム合成アノテーションを作るためにメタアノテーションとしても使える可能性があります。例えば、[@EnabledOnOsのデモ]()にある`@TestOnMac`アノテーションは、`@Test`と`@EnableOnOs`を単一で再利用可能なアノテーションに組み合わせる方法を示しています。



> ⚠️ 次章に列挙する*条件*アノテーションはそれぞれ、テストインターフェイスまたはテストクラス、テストメソッドに一度だけ宣言できます。もし条件アノテーションがある要素に直接的か間接的、またはメタ的に複数存在する場合、JUnitによって発見された最初のアノテーションのみ使われます（いかなる追加的なアノテーションも静かに無視されます）。しかしながら、`org.junit.jupiter.api.condition`パッケージでは、各条件アノテーションは他の条件アノテーションと共に使われる可能性があります。

### 3.7.1. オペレーティングシステムに関する条件
[`@EnabledOnOs`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledOnOs.html)と[`@DisabledOnOs`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledOnOs.html)アノテーションを使うことで、特定のオペーティングシステム上でコンテナまたはテストを有効にしたり無効にできます。

```java
@Test
@EnabledOnOs(MAC)
void onlyOnMacOs() {
    // ...
}

@TestOnMac
void testOnMac() {
    // ...
}

@Test
@EnabledOnOs({ LINUX, MAC })
void onLinuxOrMac() {
    // ...
}

@Test
@DisabledOnOs(WINDOWS)
void notOnWindows() {
    // ...
}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Test
@EnabledOnOs(MAC)
@interface TestOnMac {
}
```

### 3.7.2. Java実行環境に関する条件
[`@EnabledOnJre`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledOnJre.html)と[`@DisabledOnJre`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledOnJre.html)アノテーションを使うことで、特定のバージョンのJava実行環境（JRE）上でコンテナまたはテストを有効にしたり無効にできます。

```java
@Test
@EnabledOnJre(JAVA_8)
void onlyOnJava8() {
    // ...
}

@Test
@EnabledOnJre({ JAVA_9, JAVA_10 })
void onJava9Or10() {
    // ...
}

@Test
@DisabledOnJre(JAVA_9)
void notOnJava9() {
    // ...
}
```

### 3.7.3. システムプロパティに関する条件
[`@EnabledIfSystemProperty`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledIfSystemProperty.html)と[`@DisabledIfSystemProperty`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledIfSystemProperty.html)アノテーションを使うことで、`named`で指定したJVMシステムプロパティの値に応じて、コンテナまたはテストを有効にしたり無効にできます。`matches`属性を使うことで、値は正規表現として解釈されます。

```java
@Test
@EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
void onlyOn64BitArchitectures() {
    // ...
}

@Test
@DisabledIfSystemProperty(named = "ci-server", matches = "true")
void notOnCiServer() {
    // ...
}
```

### 3.7.4. 環境変数に関する条件
[`@EnabledIfEnvironmentVariable`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledIfEnvironmentVariable.html)と[`@DisabledIfEnvironmentVariable`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledIfEnvironmentVariable.html)アノテーションを使うことで、基礎となるオペレーティングシステムからの`named`で指定した環境変数の値に応じて、コンテナまたはテストを有効にしたり無効にできます。`matches`属性を使うことで、値は正規表現として解釈されます。

```java
@Test
@EnabledIfEnvironmentVariable(named = "ENV", matches = "staging-server")
void onlyOnStagingServer() {
    // ...
}

@Test
@DisabledIfEnvironmentVariable(named = "ENV", matches = ".*development.*")
void notOnDeveloperWorkstation() {
    // ...
}
```

### 3.7.5. スクリプトベースの条件
JUnit Jupiterは、[`@EnabledIf`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledIf.html)と[`@DisabledIf`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledIf.html)アノテーションを使うことで、設定されたスクリプトの評価値に応じて、コンテナまたはテストを有効にしたり無効にできる機能を提供しています。スクリプトは、JavaScriptまたはGroovy、JSR 223で定義されているJava Scripting APIをサポートしているスクリプト言語であれば記述できます。

> ⚠️  [`@EnabledIf`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledIf.html)と[`@DisabledIf`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledIf.html)を使った条件付きテストテスト実行は、現在*実験的な*機能です。詳細については、[*実験的な* APIs]()をご覧ください。

> 💡 スクリプトのロジックが、現オペレーティングまたは現Java実行環境のバージョン、特定のJVMシステムプロパティ、特定の環境変数にのみ依存している場合、その目的に合った組み込みのアノテーションを使うことを考慮すべきです。さらなる詳細については、前章をご覧ください。

> ℹ️ 同じスクリプトベースの条件を多数使っている場合、より速く、型安全で、メンテナンスのしやすい方法で条件を実装するために、それに合った[実行条件]()拡張を書くことを考えてみてください。

```java
@Test // Static JavaScript expression.
@EnabledIf("2 * 3 == 6")
void willBeExecuted() {
    // ...
}

@RepeatedTest(10) // Dynamic JavaScript expression.
@DisabledIf("Math.random() < 0.314159")
void mightNotBeExecuted() {
    // ...
}

@Test // Regular expression testing bound system property.
@DisabledIf("/32/.test(systemProperty.get('os.arch'))")
void disabledOn32BitArchitectures() {
    assertFalse(System.getProperty("os.arch").contains("32"));
}

@Test
@EnabledIf("'CI' == systemEnvironment.get('ENV')")
void onlyOnCiServer() {
    assertTrue("CI".equals(System.getenv("ENV")));
}

@Test // Multi-line script, custom engine name and custom reason.
@EnabledIf(value = {
                "load('nashorn:mozilla_compat.js')",
                "importPackage(java.time)",
                "",
                "var today = LocalDate.now()",
                "var tomorrow = today.plusDays(1)",
                "tomorrow.isAfter(today)"
            },
            engine = "nashorn",
            reason = "Self-fulfilling: {result}")
void theDayAfterTomorrow() {
    LocalDate today = LocalDate.now();
    LocalDate tomorrow = today.plusDays(1);
    assertTrue(tomorrow.isAfter(today));
}
```

#### スクリプトバインディング
次の名前は、各スクリプトコンテキストでバインドされているめ、スクリプト内で使用可能です。*accessor*は、単純な `String get(String name)`メソッドを介してマップライク（map-like）な構造へのアクセスを提供します。

|Name|Type|Description|
|---|---|---|
|systemEnvironment|*accessor*|オペレーティングシステム環境変数のアクセサ|
|systemProperty|*accessor*|JVMシステムプロパティのアクセサ|
|junitConfigurationParameter|*accessor*|設定パラメータのアクセサ|
|junitDisplayName|String|テストまたはコンテナの表示名|
|junitTags|Set\<String\>|テストまたはコンテナに振られている全てのタグ|
|junitUniqueId|String|テストまたはコンテナのユニークなID|

## 3.8. タグとフィルタリング
テストクラスとメソッドは`@Tag`アノテーションを用いてタグ付けできます。それらのタグは後に[テスト発見と実行]()をフィルタリングするために使われます。

### 3.8.1. タグの構文規則
- タグは`null`か*空*であってはならない。
- *トリミングされた*タグは空白文字を含んではならない。
- *トリミングされた*タグはISO制御文字を含んではならない。
- *トリミングされた*タグは次の*予約語*のいずれも含んではならない。
    - `,`：カンマ
    - `(`：左カッコ
    - `)`：右カッコ
    - `&`：アンパサンド
    - `|`：縦棒
    - `!`：エクスクラメーション

> ℹ️ 上の文章で、*トリミングされた*というのは、語頭と語尾の空白文字を取り除いたということを意味します。

```java
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;

@Tag("fast")
@Tag("model")
class TaggingDemo {

    @Test
    @Tag("taxes")
    void testingTaxCalculation() {
    }

}
```

## 3.9. テストインスタンス・ライフサイクル
各テストメソッドの独立した実行と、変化可能なテストインスタンスの状態による予期せぬ副作用を避けるため、JUnitは各*テストメソッド*を実行する前に、各テストクラスの新しいインスタンスを生成します（[テストクラスとメソッド]()をご覧ください）。この"メソッドごと"のテストインスタンス・ライフサイクルはJUnit Jupiterではデフォルトの動作で、以前の全てのバージョンのJUnitと類似したものになっています。

> ℹ️ `@Disabled`や`@DisabledOnOs`といった[条件]()によって*無効化*された*テストメソッド*であっても、テストクラスはインスタンス化されることに注意してください。これは"メソッドごと"テストインスタンス・ライフサイクルモードが有効である時でも同様です。

JUnit Jupiterに全テストメソッドを同じテストインスタンス上で実行してほしい場合は、単にテストクラスに`@TestInstance(Lifecycle.PER_CLASS)`アノテーションを付与するだけで実現可能です。このモードを使用する場合、テストクラス毎に新しいテストインスタンスが一度だけ生成されます。これによって、テストメソッドがインスタンス変数に保存された状態に依存する場合は、`@BeforeEach`または`AfterEach`メソッドでその状態をリセットする必要かもしれません。

”クラスごと”のモードは、デフォルトの"メソッドごと"モードに比べていくつかの追加的な利点があります。特に、"クラスごと"モードを使うと、インターフェイスの`default`メソッドと同様に、`@BeforeAll`と`@AfterAll`メソッドを非静的メソッドとして宣言することが可能になります。そのため、"クラスごと"モードでは、`@Nested`テストクラス内で`@BeforeAll`と`@AfterAll`メソッドを使うことができます。

Kotlinプログラミング言語でテストを書いている場合、”クラスごと”テストインスタンス・ライフサイクルモードに切り替えることで、`@BeforeAll`と`@AfterAll`メソッドの実装がより容易になるかもしれません。

### 3.9.1. デフォルトのテストインスタンス・ライフサイクルの変更
テストクラスまたはテストインターフェイスに`@TestInstance`が付与されていない場合、JUnit Jupiterは*デフォルト*のライフサイクルモードを使います。
標準的な*デフォルト*モードは`PER_METHOD`ですが、テスト計画全体を実行するための*デフォルト*を変更することが可能です。デフォルトのテストインスタンス・ライフサイクルモードを変更するには、単に`junit.jupiter.testinstance.lifecycle.default`*設定パラメータ*に`TestInstance.Lifecycle`に定義されているenum定数名を（大文字・小文字を無視して）設定するだけです。これは、JVMシステムプロパティとして渡すか、`Launcher`に渡される`LauncherDiscoveryRequest`内の*設定パラメータ*として渡すか、JUnit Platformの設定ファイル（詳細については、[設定パラメータ]()をご覧ください。）を通して渡します。

例えば、デフォルトのテストインスタンス・ライフサイクルモードを`LifeCycle.PER_CLASS`に設定するには、JVMを次のシステムプロパティで起動してください。

`-Djunit.jupiter.testinstance.lifecycle.default=per_class`

しかしながら、JUnit Platformの設定ファイルを通してデフォルトのテストインスタンス・ライフサイクルモードを設定する方が、より堅牢なソリューションです。設定ファイルはプロジェクトのバージョン管理システムに取り込め、自身のIDEやビルドソフトウェアで利用できます。

JUnit Platformの設定ファイルを通してデフォルトのテストインスタンス・ライフサイクルモードを設定ためには、次の内容を含んだ`junit-platform.properties`という名前のファイルをクラスパス（例えば、`src/test/resources`）のルートに生成してください。

`junit.jupiter.testinstance.lifecycle.default = per_class`

> ⚠️  *デフォルト*のテストインスタンス・ライフサイクルモードを変更することは、一貫性を持って適用しないと、予測不可能な結果と壊れやすいビルドにつながる恐れがあります。例えば、ビルドではデフォルトとして”クラスごと”のセマンティックを設定していながら、IDEでのテストでは"メソッドごと"で実行していた場合、ビルドサーバで起きるエラーをデバッグすることは困難になる恐れがあります。そのため、JVMシステムプロパティの代わりに、JUnit Platformの設定ファイルを使ってデフォルトを変更することをお薦めします。

## 3.10. ネストされたテスト
ネストされたテストは、テスト開発者が様々なグループのテスト間の関係を表現することを可能にします。これがその美しい例です。

*スタックをテストするためのネストされたテスト*
```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.util.EmptyStackException;
import java.util.Stack;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;

@DisplayName("A stack")
class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, () -> stack.pop());
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, () -> stack.peek());
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```

> ℹ️ *非静的なネストされたクラス（つまり、内部クラス）のみ*が`@Nested`テストクラスとなります。ネストは任意に深くすることができ、それら内部クラスは一つの例外を除いて、テストクラスの完全なメンバーとして考えられます。例外は`@BeforeAll`と`@AfterAll`で、*デフォルト*では動作しません。その理由は、Javaが内部クラスに`static`なメンバーを許さないためです。しかしながら、この制限は`@Nested`テストクラスに`@TestInstance(Lifecycle.PER_CLASS)`を付与することで回避できます（[テストインスタンス・ライフサイクル]()をご覧ください）。

## 3.11. コンストラクタとメソッドへの依存性注入
JUnitの前バージョン全てにおいて、テストコンストラクタまたはメソッドは（少なくとも標準的な`Runner`実装を用いる場合は）パラメータを持つことが許されていませんでした。JUnit Jupiterでの大きな変更の1つとして、テストコンストラクタとメソッドどちらもパラメータを持てるようになりました。このことは、大きな柔軟性をもたらし、コンストラクタとメソッドに*依存性の注入*が可能になりました。

[`ParameterResolver`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ParameterResolver.html)は、実行時に*動的に*パラメータを解決することを望むテスト拡張のためのAPIを定義しています。テストコンストラクタまたは`@Test`、`@TestFactory`、`@BeforeEach`、`@AfterEach`、`@BeforeAll`、`@AfterAll`メソッドがパラメータを許容する場合は、そのパラメータは登録された`ParameterResolver`によって実行時に解決されなければなりません。

現在は、3つの組み込みリゾルバが自動的に登録されます。

- [`TestInfoParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestInfoParameterResolver.java)：メソッドパラメータが[`TestInfo`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/TestInfo.html)型の場合、`TestInfoParameterResolver`はパラメータの値として現在のテストに応じた`TestInfo`のインスタンスを供給します。`TestInfo`は、テストの表示名、テストクラス、テストメソッド、関連付けられたタグ名といった現在のテストに関する情報を集めるのに使うことができます。表示名は、テストクラスまたはテストメソッドの名前といった技術的な名前か、`@DisplayedName`で設定されたカスタム名のどちらかです。
[`TestInfo`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/TestInfo.html)は、JUnit 4の`TestName`規則の代替として動作します。次のコードは、テストコンストラクタと`@BeforeEach`メソッド、`@Test`メソッドに`TestInfo`を注入させる方法を示しています。

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;

@DisplayName("TestInfo Demo")
class TestInfoDemo {

    TestInfoDemo(TestInfo testInfo) {
        assertEquals("TestInfo Demo", testInfo.getDisplayName());
    }

    @BeforeEach
    void init(TestInfo testInfo) {
        String displayName = testInfo.getDisplayName();
        assertTrue(displayName.equals("TEST 1") || displayName.equals("test2()"));
    }

    @Test
    @DisplayName("TEST 1")
    @Tag("my-tag")
    void test1(TestInfo testInfo) {
        assertEquals("TEST 1", testInfo.getDisplayName());
        assertTrue(testInfo.getTags().contains("my-tag"));
    }

    @Test
    void test2() {
    }

}
```


- [`RepetitionInfoParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/RepetitionInfoParameterResolver.java)：`@RepeatedTest`または`@BeforeEach`、`@AfterEach`メソッドにおけるメソッドパラメータが[`RepetitionInfo`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/RepetitionInfo.html)型の場合、`RepetitionInfoParameterResolver`は`RepetitionInfo`のインスタンスを供給します。`RepetitionInfo`は、現在の繰り返しと対応する`@RepeatedTest`の繰り返しの総数に関する情報を集めるために利用できます。しかしながら、`RepetitionInfoParameterResolver`は、`@RepeatedTest`の文脈外では登録されていないことに注意してください。[繰り返しテストの例]()をご覧ください。
- [`TestReporterParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestReporterParameterResolver.java)：メソッドパラメータが[`TestReporter`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/TestReporter.html)型の場合、`TestReporterParameterResolver`は`TestReporter`のインスタンスを供給します。`TestReporter`は、現在のテスト実行に関する追加情報を公開するために利用できます。そのデータは、[`TestExecutionListener.reportingEntryPublished()`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/TestExecutionListener.html)を通して消費され、IDEに表示またはレポートに含まれます。JUnit Jupiterでは、JUnit 4で `stdout`や`stderr`に情報を出力していた箇所に`TestReporter`を使うことができます。`@RunWith(JUnitPlatform.class)`を使うと、全てのレポートされたエントリを`stdout`に出力します。

```java
import java.util.HashMap;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestReporter;

class TestReporterDemo {

    @Test
    void reportSingleValue(TestReporter testReporter) {
        testReporter.publishEntry("a key", "a value");
    }

    @Test
    void reportSeveralValues(TestReporter testReporter) {
        HashMap<String, String> values = new HashMap<>();
        values.put("user name", "dk38");
        values.put("award year", "1974");

        testReporter.publishEntry(values);
    }

}
```

> ℹ️ 他のパラメータリゾルバは、`@ExtendWith`を用いた適切な[拡張]()を登録することによって明示的に有効化する必要があります。

カスタム[`ParameterResolver`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ParameterResolver.html)の例に関して[`RandomParametersExtension`](https://github.com/junit-team/junit5-samples/tree/r5.2.0/junit5-jupiter-extensions/src/main/java/com/example/random/RandomParametersExtension.java)を確認しましょう。リリース可能なものではありませんが、拡張モデルとパラメータ解決プロセス両方の単純さと表現性を例示しています。`MyRandomParametersTest`は、`@Test`メソッドへのランダム値の挿入方法をを示しています。

```java
@ExtendWith(RandomParametersExtension.class)
class MyRandomParametersTest {

    @Test
    void injectsInteger(@Random int i, @Random int j) {
        assertNotEquals(i, j);
    }

    @Test
    void injectsDouble(@Random double d) {
        assertEquals(0.0, d, 1.0);
    }

}
```

現実的なユースケースとして、[`MockitoExtension`](https://github.com/mockito/mockito/blob/release/2.x/subprojects/junit-jupiter/src/main/java/org/mockito/junit/jupiter/MockitoExtension.java)と[`SpringExtension`](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java)のソースコードを確認してください。

## 3.12. テストインターフェイスとデフォルトメソッド
JUnit Jupiterは、`@Test`と`@RepeatedTest`、`@ParameterizedTest`、`@TestFactory`、`@TestTemplate`、`@BeforeEach`、`@AfterEach`にインターフェイスに`default`メソッドを宣言できるようにしています。`@BeforeAll`と`@AfrterAll`はテストインターフェイス内で`static`メソッドを宣言するか、*もし*テストインターフェイスまたはテストクラスに`@TestInstance(Lifecycle.PER_CLASS)`が付与されている場合はインターフェイス`default`メソッドを宣言することができます（[テストインスタンス・ライフサイクル]()をご覧ください）。いくつかの例を示します。

```java
@TestInstance(Lifecycle.PER_CLASS)
interface TestLifecycleLogger {

    static final Logger LOG = Logger.getLogger(TestLifecycleLogger.class.getName());

    @BeforeAll
    default void beforeAllTests() {
        LOG.info("Before all tests");
    }

    @AfterAll
    default void afterAllTests() {
        LOG.info("After all tests");
    }

    @BeforeEach
    default void beforeEachTest(TestInfo testInfo) {
        LOG.info(() -> String.format("About to execute [%s]",
            testInfo.getDisplayName()));
    }

    @AfterEach
    default void afterEachTest(TestInfo testInfo) {
        LOG.info(() -> String.format("Finished executing [%s]",
            testInfo.getDisplayName()));
    }

}
```

```java
interface TestInterfaceDynamicTestsDemo {

    @TestFactory
    default Collection<DynamicTest> dynamicTestsFromCollection() {
        return Arrays.asList(
            dynamicTest("1st dynamic test in test interface", () -> assertTrue(true)),
            dynamicTest("2nd dynamic test in test interface", () -> assertEquals(4, 2 * 2))
        );
    }

}
```

`@ExtenWith`と `@Tag`はテストインターフェイスとして宣言することができるため、インターフェイスを実装したクラスは自動的にタグと拡張を継承します。[`TimingExtension`]()のソースコードを見るには、[`BeforeとAfterのテスト実行コールバック`]()をご覧ください。

```java
@Tag("timed")
@ExtendWith(TimingExtension.class)
interface TimeExecutionLogger {
}
```

テストクラスでは、これらのテストインターフェイスを実装することで適用することができます。

```java
class TestInterfaceDemo implements TestLifecycleLogger,
        TimeExecutionLogger, TestInterfaceDynamicTestsDemo {

    @Test
    void isEqualValue() {
        assertEquals(1, 1, "is always equal");
    }

}
```

`TestInterfaceDemo`を実行すると、次と同様の出力が得られます。

```
:junitPlatformTest
INFO  example.TestLifecycleLogger - Before all tests
INFO  example.TestLifecycleLogger - About to execute [dynamicTestsFromCollection()]
INFO  example.TimingExtension - Method [dynamicTestsFromCollection] took 13 ms.
INFO  example.TestLifecycleLogger - Finished executing [dynamicTestsFromCollection()]
INFO  example.TestLifecycleLogger - About to execute [isEqualValue()]
INFO  example.TimingExtension - Method [isEqualValue] took 1 ms.
INFO  example.TestLifecycleLogger - Finished executing [isEqualValue()]
INFO  example.TestLifecycleLogger - After all tests

Test run finished after 190 ms
[         3 containers found      ]
[         0 containers skipped    ]
[         3 containers started    ]
[         0 containers aborted    ]
[         3 containers successful ]
[         0 containers failed     ]
[         3 tests found           ]
[         0 tests skipped         ]
[         3 tests started         ]
[         0 tests aborted         ]
[         3 tests successful      ]
[         0 tests failed          ]

BUILD SUCCESSFUL
```

この機能の他のあり得る適用としては、インターフェイス契約のためにテストを書くことです。例えば、`Object.equals`または`Comparable.compareTo`の実装がどう振る舞うべきかのテストを、次のように書くことができます。

```java
public interface Testable<T> {

    T createValue();

}
```

```java
public interface EqualsContract<T> extends Testable<T> {

    T createNotEqualValue();

    @Test
    default void valueEqualsItself() {
        T value = createValue();
        assertEquals(value, value);
    }

    @Test
    default void valueDoesNotEqualNull() {
        T value = createValue();
        assertFalse(value.equals(null));
    }

    @Test
    default void valueDoesNotEqualDifferentValue() {
        T value = createValue();
        T differentValue = createNotEqualValue();
        assertNotEquals(value, differentValue);
        assertNotEquals(differentValue, value);
    }

}
```

```java
public interface ComparableContract<T extends Comparable<T>> extends Testable<T> {

    T createSmallerValue();

    @Test
    default void returnsZeroWhenComparedToItself() {
        T value = createValue();
        assertEquals(0, value.compareTo(value));
    }

    @Test
    default void returnsPositiveNumberComparedToSmallerValue() {
        T value = createValue();
        T smallerValue = createSmallerValue();
        assertTrue(value.compareTo(smallerValue) > 0);
    }

    @Test
    default void returnsNegativeNumberComparedToSmallerValue() {
        T value = createValue();
        T smallerValue = createSmallerValue();
        assertTrue(smallerValue.compareTo(value) < 0);
    }

}
```

テストクラスでは、2つ契約インターフェイスを実装することで、対応するテストを継承します。もちろん、抽象メソッドを実装する必要があります。

```java
class StringTests implements ComparableContract<String>, EqualsContract<String> {

    @Override
    public String createValue() {
        return "foo";
    }

    @Override
    public String createSmallerValue() {
        return "bar"; // 'b' < 'f' in "foo"
    }

    @Override
    public String createNotEqualValue() {
        return "baz";
    }

}
```

> ℹ️ 上記のテストは、単に例であって、完全ではありません。

## 3.13. 繰り返しテスト
JUnit Jupiterは、`@RepeatedTest`を付与し、繰り返してほしい回数を設定するだけで、特定回数テストを繰り返す機能を提供しています。繰り返しテストの各呼び出しは、通常の`@Test`メソッドの実行のように振る舞い、同じライフサイクル・コールバックと拡張を完全にサポートしています。

次の例は、自動で10回繰り返す`repeatedTest()`という名前のテストの宣言方法を示しています。

```java
@RepeatedTest(10)
void repeatedTest() {
    // ...
}
```

繰り返し回数の設定に加えて、`@RepeatedTest`アノテーションの`name`属性を用いることでカスタム表示名も設定できます。さらに、表示名は、静的なテキストと動的なプレースホルダの組み合わせで構成されるパターンにすることもできます。次のプレースホルダが現在サポートされています。

- `{displayName}`:`@RepeatedTest`メソッドの表示名
- `{currentRepetition}`: 現在の繰り返し回数
- `{totalRepetition}`: 繰り返し回数の合計

ある繰り返し回数時点でのデフォルトの表示名は、次のパターンに基づいて生成されます：`"repetition {currentRepetition} of {totalRepetitions}"`。そのため、先ほどの例の各繰り返し回数における表示名は次のようになります：`repetition 1 of 10`や`repetition 2 of 10`など。`@RepeatedTest`メソッドの表示名に各繰り返しの名前を含めたい場合は、独自のカスタムパターンを定義するか、事前定義された`RepeatedTest.LONG_DISPLAY_NAME`パターンを使うことができます。後者は、`"{displayName} :: repetition {currentRepetition} of {totalRepetitions}"`と等しいもので、各繰り返しの表示名は`repeatedTest() :: repetition 1 of 10`や`repeatedTest() :: repetition 2 of 10`などとなります。

現在の繰り返し回数と繰り返しの合計数の情報をプログラム的に集めるために、`@RepeatedTest`または`@BeforeEach`、`@AfterEach`に`RepetitionInfo`インスタンスを挿入することができます。

### 3.13.1. 繰り返しテストの例
この章の最後にある`RepeatedTestsDemo`クラスは、繰り返しテストのいくつかの例を示しています。

`repeatedTest()`メソッドは、前章からの例です。一方、`repeatedTestWithRepetitionInfo()`は、現在繰り返されているテストの繰り返し合計数を得るために`RepetitionInfo`インスタンスをテストに注入する方法を示しています。

その次の2つのメソッドは、`@RepeatedTest`のカスタム`@DisplayName`を各繰り返しの表示名内に含ませる方法を示しています。`customDisplayName()`はカスタム表示名とカスタムパターンを組み合わせており、`TestInfo`を使って生成された表示名のフォーマットを検証しています。`Repeat!`は`@DisplayName`宣言から来る`{displayName}`で、`1/1`は`{currentRepetition}/{totalRepetitions}`から来ています。対照的に、`customDisplayNameWithLongPattern()`は、先ほど説明した事前定義の`RepeatedTest.LONG_DISPLAY_NAME`パターンを使っています。

`repeatedTestInGerman()`は、繰り返しテストの表示名を他国言語（この場合はドイツ語です）に翻訳する機能を示しています。その結果、各繰り返しにおける名前は、`Wiederholung 1 von 5`や`Wiederholung 2 von 5`などのようになります。

`beforeEach()`メソッドは`@BeforeEach`が付与されているため、各繰り返しテストの各繰り返し前に実行されます。`TestInfo`と`RepetitionInfo`をこのメソッドに注入することで、現在実行されている繰り返しテストに関する情報を得ることができます。`INFO`ログレベルで`RepeatedTestsDemo`を実行すると出力は次のようになります。

```
INFO: About to execute repetition 1 of 10 for repeatedTest
INFO: About to execute repetition 2 of 10 for repeatedTest
INFO: About to execute repetition 3 of 10 for repeatedTest
INFO: About to execute repetition 4 of 10 for repeatedTest
INFO: About to execute repetition 5 of 10 for repeatedTest
INFO: About to execute repetition 6 of 10 for repeatedTest
INFO: About to execute repetition 7 of 10 for repeatedTest
INFO: About to execute repetition 8 of 10 for repeatedTest
INFO: About to execute repetition 9 of 10 for repeatedTest
INFO: About to execute repetition 10 of 10 for repeatedTest
INFO: About to execute repetition 1 of 5 for repeatedTestWithRepetitionInfo
INFO: About to execute repetition 2 of 5 for repeatedTestWithRepetitionInfo
INFO: About to execute repetition 3 of 5 for repeatedTestWithRepetitionInfo
INFO: About to execute repetition 4 of 5 for repeatedTestWithRepetitionInfo
INFO: About to execute repetition 5 of 5 for repeatedTestWithRepetitionInfo
INFO: About to execute repetition 1 of 1 for customDisplayName
INFO: About to execute repetition 1 of 1 for customDisplayNameWithLongPattern
INFO: About to execute repetition 1 of 5 for repeatedTestInGerman
INFO: About to execute repetition 2 of 5 for repeatedTestInGerman
INFO: About to execute repetition 3 of 5 for repeatedTestInGerman
INFO: About to execute repetition 4 of 5 for repeatedTestInGerman
INFO: About to execute repetition 5 of 5 for repeatedTestInGerman
```

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.logging.Logger;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.RepetitionInfo;
import org.junit.jupiter.api.TestInfo;

class RepeatedTestsDemo {

    private Logger logger = // ...

    @BeforeEach
    void beforeEach(TestInfo testInfo, RepetitionInfo repetitionInfo) {
        int currentRepetition = repetitionInfo.getCurrentRepetition();
        int totalRepetitions = repetitionInfo.getTotalRepetitions();
        String methodName = testInfo.getTestMethod().get().getName();
        logger.info(String.format("About to execute repetition %d of %d for %s", //
            currentRepetition, totalRepetitions, methodName));
    }

    @RepeatedTest(10)
    void repeatedTest() {
        // ...
    }

    @RepeatedTest(5)
    void repeatedTestWithRepetitionInfo(RepetitionInfo repetitionInfo) {
        assertEquals(5, repetitionInfo.getTotalRepetitions());
    }

    @RepeatedTest(value = 1, name = "{displayName} {currentRepetition}/{totalRepetitions}")
    @DisplayName("Repeat!")
    void customDisplayName(TestInfo testInfo) {
        assertEquals(testInfo.getDisplayName(), "Repeat! 1/1");
    }

    @RepeatedTest(value = 1, name = RepeatedTest.LONG_DISPLAY_NAME)
    @DisplayName("Details...")
    void customDisplayNameWithLongPattern(TestInfo testInfo) {
        assertEquals(testInfo.getDisplayName(), "Details... :: repetition 1 of 1");
    }

    @RepeatedTest(value = 5, name = "Wiederholung {currentRepetition} von {totalRepetitions}")
    void repeatedTestInGerman() {
        // ...
    }

}
```

`ConsoleLauncher`またはunicodeテーマを有効化した`junitPlatformTest`Gradleプラグインを使うと、`RepeatedTestsDemo`の実行結果は次のようなコンソール出力を行います。

```
├─ RepeatedTestsDemo ✔
│  ├─ repeatedTest() ✔
│  │  ├─ repetition 1 of 10 ✔
│  │  ├─ repetition 2 of 10 ✔
│  │  ├─ repetition 3 of 10 ✔
│  │  ├─ repetition 4 of 10 ✔
│  │  ├─ repetition 5 of 10 ✔
│  │  ├─ repetition 6 of 10 ✔
│  │  ├─ repetition 7 of 10 ✔
│  │  ├─ repetition 8 of 10 ✔
│  │  ├─ repetition 9 of 10 ✔
│  │  └─ repetition 10 of 10 ✔
│  ├─ repeatedTestWithRepetitionInfo(RepetitionInfo) ✔
│  │  ├─ repetition 1 of 5 ✔
│  │  ├─ repetition 2 of 5 ✔
│  │  ├─ repetition 3 of 5 ✔
│  │  ├─ repetition 4 of 5 ✔
│  │  └─ repetition 5 of 5 ✔
│  ├─ Repeat! ✔
│  │  └─ Repeat! 1/1 ✔
│  ├─ Details... ✔
│  │  └─ Details... :: repetition 1 of 1 ✔
│  └─ repeatedTestInGerman() ✔
│     ├─ Wiederholung 1 von 5 ✔
│     ├─ Wiederholung 2 von 5 ✔
│     ├─ Wiederholung 3 von 5 ✔
│     ├─ Wiederholung 4 von 5 ✔
│     └─ Wiederholung 5 von 5 ✔
```

## 3.14. パラメータ化テスト
パラメータ化テストを使うと、テストを異なる引数で複数回実行できるようになります。パラメータ化テストは、通常の`@Test`メソッドの代わりに[`@ParameterizedTest`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/params/ParameterizedTest.html)アノテーションを付与するだけで宣言することができます。さらに、各呼び出して供給されテストで*消費される*引数として、少なくとも1つの*source*を宣言する必要があります。

次の例は、パラメータ化テストを示していて、`@ValueSource`アノテーションを使って引数のソースとして`String`配列を指定しています。

```
@ParameterizedTest
@ValueSource(strings = { "racecar", "radar", "able was I ere I saw elba" })
void palindromes(String candidate) {
    assertTrue(isPalindrome(candidate));
}
```

上記のパラメータ化テストメソッドを実行すると、各呼び出しは別々にレポートされます。例えば、`ConsoleLauncher`は次のようなものを出力します。

```
palindromes(String) ✔
├─ [1] racecar ✔
├─ [2] radar ✔
└─ [3] able was I ere I saw elba ✔
```

> ⚠️  パラメータ化テストは、現在*実験的な*機能です。詳細については、[実験的なAPIs]()をご覧ください。

### 3.14.1. 必要なセットアップ
パラメータ化テストを使うためには、`junit-jupiter-params`アーティファクトを依存関係に加える必要があります。詳細については、[依存関係のメタデータ]()をご覧ください。

## 3.15. 引数の消費
パラメータ化テストメソッドは典型的に、設定されたソース（[引数のソース]()をご覧ください。）から直接、引数を*消費*します。引数ソースとメソッドパラメータのインデックスは1対1の相関関係に従います（[`@CsvSource`]()の例をご覧ください）。しかしながら、パラメータ化テストメソッドは、ソースから得た引数をひとつのオブジェクトに*集約*して、メソッドに渡すこともできます（[引数集約]()をご覧ください）。追加的な引数もまた（例えば、`TestInfo`や`TestReporter`などのインスタンスを獲得するために）`ParameterResolver`によって提供されます。特に、パラメータ化テストメソッドは、次のルールに従って形式的なパラメータを宣言する必要があります。

- まず、0個以上の*インデックスされた引数*を宣言する。
- 次に、0個以上の*アグリゲータ*を宣言する。
- 最後に、0個以上の`ParameterResolver`によって供給される引数を宣言する。

この文脈で、*インデックスされた引数*とは、`ArgumentsProvider`によって提供される`Arguments`内で与えられたインデックスに対応する引数です。`ArgumentsProvider`は、パラメータ化メソッドが保持する形式的なパラメータリストにおいて同じインデックスにあるメソッドに引数として渡されます。*アグリゲータ*は、`ArgumentsAccessor`型または`@AggregateWith`の付与されたパラメータです。

### 3.15.1. 引数のソース
すぐに使えるように、JUnit Jupiterはごく少数の*ソース*アノテーションを提供しています。次の各章はそれぞれ、簡潔な概要とそれぞれの例を提供しています。さらなる情報に関しては、[`org.junit.jupiter.params.provider`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/params/provider/package-summary.html)パッケージのJavaDocを参照してください。

#### `@ValueSource`
`@ValueSource`は最も単純なソースの1つです。リテラル値の配列を1つ設定することができ、パラメータ化テスト呼び出しにつき、1つのパラメータを提供できます。

次のリテラル値の型が`@ValueSource`にサポートされています。

- `short`
- `byte`
- `int`
- `long`
- `float`
- `double`
- `char`
- `java.lang.String`
- `java.lang.Class`

例えば、次の`@ParameterizedTest`メソッドはそれぞれ`1`、`2`、`3`の値とともに3回呼び出されます。

```java
@ParameterizedTest
@ValueSource(ints = { 1, 2, 3 })
void testWithValueSource(int argument) {
    assertTrue(argument > 0 && argument < 4);
}
```

#### `@EnumSource`
`@EnumSource`は、`Enum`定数に対して便利な機能を提供します。このアノテーションは、使われる定数を特定するために、オプションで`names`パラメータを提供します。省略する場合は、次の例のように全ての定数が使われます。

```java
@ParameterizedTest
@EnumSource(TimeUnit.class)
void testWithEnumSource(TimeUnit timeUnit) {
    assertNotNull(timeUnit);
}
```

```java

@ParameterizedTest
@EnumSource(value = TimeUnit.class, names = { "DAYS", "HOURS" })
void testWithEnumSourceInclude(TimeUnit timeUnit) {
    assertTrue(EnumSet.of(TimeUnit.DAYS, TimeUnit.HOURS).contains(timeUnit));
}
```

`@EnumSource`アノテーションはまた、テストメソッドに渡すパラメータを細かく制御するために、オプションで`mode`パラメータを提供します。例えば、次の例では、enum定数プールからnamesを取り除いたり、正規表現を設定しています。

```java
@ParameterizedTest
@EnumSource(value = TimeUnit.class, mode = EXCLUDE, names = { "DAYS", "HOURS" })
void testWithEnumSourceExclude(TimeUnit timeUnit) {
    assertFalse(EnumSet.of(TimeUnit.DAYS, TimeUnit.HOURS).contains(timeUnit));
    assertTrue(timeUnit.name().length() > 5);
}
```

```java
@ParameterizedTest
@EnumSource(value = TimeUnit.class, mode = MATCH_ALL, names = "^(M|N).+SECONDS$")
void testWithEnumSourceRegex(TimeUnit timeUnit) {
    String name = timeUnit.name();
    assertTrue(name.startsWith("M") || name.startsWith("N"));
    assertTrue(name.endsWith("SECONDS"));
}
```

#### `@MethodSource`
`@MethodSource`では、テストクラスまたは外部クラスの*ファクトリー*メソッドを1つ以上使うことができます。ファクトリーメソッドは、`Stream`または`Iterable`、`Iterator`、引数の配列を返す必要があります。さらに、ファクトリーメソッドには引数を与えられません。テストクラス内のファクトリーメソッドは、テストクラスに`@TestInstance(Lifecycle.PER_CLASS)`が付与されていない限り、`static`である必要があります。一方、外部クラスのファクトリーメソッドは常に`static`である必要があります。

パラメータが1つだけ必要な場合は、次の例が示しているように、パラメータの型のインスタンスの`Stream`を返すことができます。

```java
@ParameterizedTest
@MethodSource("stringProvider")
void testWithSimpleMethodSource(String argument) {
    assertNotNull(argument);
}

static Stream<String> stringProvider() {
    return Stream.of("foo", "bar");
}
```

`@MethodSource`を通して明示的にファクトリーメソッドの名前を提供しない場合、JUnit Jupiterは、慣例にならって現在の`@ParameterizedTest`と同じ名前を持つ*ファクトリー*メソッドを探します。これを次の例で示します。

```java
@ParameterizedTest
@MethodSource
void testWithSimpleMethodSourceHavingNoValue(String argument) {
    assertNotNull(argument);
}

static Stream<String> testWithSimpleMethodSourceHavingNoValue() {
    return Stream.of("foo", "bar");
}
```

`DoubleStream`や`IntStream`、`LongStream`といったプリミティブ型のStreamもまた、次の例のようにサポートされています。

```java
@ParameterizedTest
@MethodSource("range")
void testWithRangeMethodSource(int argument) {
    assertNotEquals(9, argument);
}

static IntStream range() {
    return IntStream.range(0, 20).skip(10);
}
```

テストメソッドが複数のパラメータを宣言している場合、下に示すように`Arguments`インスタンスのコレクションまたはストリームを返す必要があります。`Arguments.of(Object...)`は、`Arguments`インターフェイスで定義されている静的なファクトリーメソッドです。

```java
@ParameterizedTest
@MethodSource("stringIntAndListProvider")
void testWithMultiArgMethodSource(String str, int num, List<String> list) {
    assertEquals(3, str.length());
    assertTrue(num >=1 && num <=2);
    assertEquals(2, list.size());
}

static Stream<Arguments> stringIntAndListProvider() {
    return Stream.of(
        Arguments.of("foo", 1, Arrays.asList("a", "b")),
        Arguments.of("bar", 2, Arrays.asList("x", "y"))
    );
}
```

外部の`static`*ファクトリー*メソッドは、次の例で示すように*完全修飾メソッド名*によって参照されます。

```java
package example;

import java.util.stream.Stream;

import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;

class ExternalMethodSourceDemo {

    @ParameterizedTest
    @MethodSource("example.StringsProviders#blankStrings")
    void testWithExternalMethodSource(String blankString) {
        // test with blank string
    }
}

class StringsProviders {

    static Stream<String> blankStrings() {
        return Stream.of("", " ", " \n ");
    }
}
```

#### `@CsvSource`
`@CsvSource`は、引数リストをコンマ区切りの値（つまり、`String`リテラル）として表現できるようにします。

```java
@ParameterizedTest
@CsvSource({ "foo, 1", "bar, 2", "'baz, qux', 3" })
void testWithCsvSource(String first, int second) {
    assertNotNull(first);
    assertNotEquals(0, second);
}
```

`@CsvSource`は、シングルクォーテーション`'`を引用文字として使います。上の例と下の表の`'baz, qux'`の値をご覧ください。引用された空の値`''`は、空の`String`となります。一方、完全に*空*の値は`null`参照として解釈されます。`null`参照が対象とする型がプリミティブ型の場合、`ArgumentConversionException`が投げられます。

|入力例|引数リストの結果|
|---|---|
|`@CsvSource({ "foo, bar" })`|`"foo"`, `"bar"`|
|`@CsvSource({ "foo, 'baz, qux'" })`|`"foo"`, `"baz, qux"`|
|`@CsvSource({ "foo, ''" })`|`"foo"`, `""`|
|`@CsvSource({ "foo, " })`|`"foo"`, `null`|

#### `@CsvFileSource`
`@CsvFileSource`は、CSVファイルをクラスパスから使えるようにします。パラメータ化テストが1回呼び出される度に、CSVファイルの各行が読み込まれます。

```java
@ParameterizedTest
@CsvFileSource(resources = "/two-column.csv", numLinesToSkip = 1)
void testWithCsvFileSource(String first, int second) {
    assertNotNull(first);
    assertNotEquals(0, second);
}
```

```:two-column.csv
Country, reference
Sweden, 1
Poland, 2
"United States of America", 3
```

> ℹ️ `@CsvSource`で使われている構文とは対照的に、`@CsvFileSource`では引用文字としてダブルクォーテーション`"`を使います。上記の例の`"United States of America"`をご覧ください。引用された空の値`””`は、空の`String`となります。一方、完全に*空*の値は`null`参照として解釈されます。`null`参照が対象とする型がプリミティブ型の場合、`ArgumentConversionException`が投げられます。

#### `@ArgumentSource`
`@ArgumentSource`はカスタムの再利用可能な`ArgumentsProvider`を特定するために使うことができます。

```java
@ParameterizedTest
@ArgumentsSource(MyArgumentsProvider.class)
void testWithArgumentsSource(String argument) {
    assertNotNull(argument);
}

public class MyArgumentsProvider implements ArgumentsProvider {

    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
        return Stream.of("foo", "bar").map(Arguments::of);
    }
}
```

### 3.15.2. 引数変換
#### 拡大的な変換
JUnit Jupiterは`@ParamterizedTest`に供給する引数のために、[拡大的なプリミティブ変換](https://docs.oracle.com/javase/specs/jls/se8/html/jls-5.html#jls-5.1.2)をサポートしています。例えば、`@ValueSource(ints = { 1, 2, 3 })`が付与されたパラメータ化テストは、`int`型のみならず、`long`や`float`、`double`型の引数も受けることができます。

#### 暗示的な変換
`@CsvSource`のようなユースケースをサポートするために、JUnit Jupiterは組み込みの暗示的な型変換をいくつか提供しています。変換プロセスは、各メソッドパラメータの宣言された型に依存します。

例えば、`@ParameterizedTest`が`TimeUnit`型のパラメータを宣言していて、ソースから供給された実際の型が`String`であった場合、`String`は自動的に対応する`TimeUnit`enum定数に変換されます。

```java
@ParameterizedTest
@ValueSource(strings = "SECONDS")
void testWithImplicitArgumentConversion(TimeUnit argument) {
    assertNotNull(argument.name());
}
```

`String`インスタンスは現在、次の対象型に暗示的に変換されます。

|対象型|例|
|---|---|
|`boolean`/`Boolean`|`"true"` → `true`|
|`byte`/`Byte`|`"1"` → (byte) `1`|
|`char`/`Character`|`"o"` → `'o'`|
|`short`/`Short`|`"1"` → (short) `1`|
|`int`/`Integer`|`"1"` → `1`|
|`long`/`Long`|`"1"` → `1L`|
|`float`/`Float`|`"1"` → `1.0f`|
|`double`/`Double`|`"1"` → `1.0d`|
|`Enum`サブクラス|`"SECONDS"` → `TimeUnit.SECONDS`|
|`java.io.File`|`"/path/to/file"` → `new File("path/to/file")`|
|`java.math.BigDecimal`|`"123.456e789"` → `new BigDecimal("123.456e789")`|
|`java.math.BigInteger`|`"1234567890123456789"` → `new BigInteger("1234567890123456789")`|
|`java.net.URI`|`"http://junit.org/"` → `URI.create("http://junit.org/")`|
|`java.net.URL`|`"http://junit.org/"` → `new URL("http://junit.org/")`|
|`java.nio.file.Path`|`"/path/to/file"` → `Paths.get("/path/to/file")`|
|`java.time.Instant`|`"1970-01-01T00:00:00Z"` → `Instant.ofEpochMilli(0)`|
|`java.time.LocalDateTime`|`"2017-03-14T12:34:56.789"` → `LocalDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000)`|
|`java.time.LocalDate`|`"2017-03-14"` → `LocalDate.of(2017, 3, 14)`|
|`java.time.LocalTime`|`"12:34:56.789"` → `LocalTime.of(12, 34, 56, 789_000_000)`|
|`java.time.OffsetDateTime`|`"2017-03-14T12:34:56.789Z"` → `OffsetDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000, ZoneOffset.UTC)`|
|`java.time.OffsetTime`|`"12:34:56.789Z"` → `OffsetTime.of(12, 34, 56, 789_000_000, ZoneOffset.UTC)`|
|`java.time.YearMonth`|`"2017-03"` → `YearMonth.of(2017, 3)`|
|`java.time.Year`|`"2017"` → `Year.of(2017)`|
|`java.time.ZonedDateTime`|`"2017-03-14T12:34:56.789Z"` → `ZonedDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000, ZoneOffset.UTC)`|
|`java.util.Currency`|`"JPY"` → `Currency.getInstance("JPY")`|
|`java.util.Locale`|`"en"` → `new Locale("en")`|
|`java.util.UUID`|`"d043e930-7b3b-48e3-bdbe-5a3ccfb833db"` → `UUID.fromString("d043e930-7b3b-48e3-bdbe-5a3ccfb833db")`|

#### StringからObjectへの予備的な変換
Stringから上に列挙されている対象型への暗示的な変換に加えて、JUnit Jupiterでは、対象型が下の定義に合致した*ファクトリーメソッド*または*ファクトリーコンストラクタ*を宣言する場合、`String`をその対象型へ自動変換する予備的な機構を提供します。

- *ファクトリーメソッド*：対象型で宣言されている非プリミティブかつ`static`なメソッドで、1つの`String`引数を取り、対象型のインスタンスを返すもの。メソッド名は任意であり、特定の慣習にも従う必要はありません。
- *ファクトリーコンストラクタ*：対象型のプライベートでないコンストラクタで、1つの`String`引数を取るもの。

> ℹ️ 複数の*ファクトリーメソッド*が見つかった場合、それらは無視されます。*ファクトリーメソッド*と*ファクトリーコンストラクタ*が見つかった場合、ファクトリーメソッドがコンストラクタの代わりに使われます。

例えば、次の`@ParameterizedTest`メソッドの中で、`Book`引数は`Book.fromTitle(String)`ファクトリーメソッドが呼び出されることで生成され、`"42 Cats"`が本のタイトルとして渡されます。

```java
@ParameterizedTest
@ValueSource(strings = "42 Cats")
void testWithImplicitFallbackArgumentConversion(Book book) {
    assertEquals("42 Cats", book.getTitle());
}

public class Book {

    private final String title;

    private Book(String title) {
        this.title = title;
    }

    public static Book fromTitle(String title) {
        return new Book(title);
    }

    public String getTitle() {
        return this.title;
    }
}
```

#### 明示的な変換
暗黙的な引数変換の代わりに、次の例のように`@ConvertWith`アノテーションを使うことで、あるパラメータに対して明示的に`ArgumentConverter`を特定することができます。

```java
@ParameterizedTest
@EnumSource(TimeUnit.class)
void testWithExplicitArgumentConversion(
        @ConvertWith(ToStringArgumentConverter.class) String argument) {

    assertNotNull(TimeUnit.valueOf(argument));
}

public class ToStringArgumentConverter extends SimpleArgumentConverter {

    @Override
    protected Object convert(Object source, Class<?> targetType) {
        assertEquals(String.class, targetType, "Can only convert to String");
        return String.valueOf(source);
    }
}
```

明示的な引数変換は、テストと拡張の開発者によって実装される必要があります。そのため、`junit-jupiter-params`では、参照実装として使える明示的な引数変換器：`JavaTimeArgumentConverter`を提供しています。合成アノテーション`であるJavaTimeConversionPattern`を通して使うことができます。

```java
@ParameterizedTest
@ValueSource(strings = { "01.01.2017", "31.12.2017" })
void testWithExplicitJavaTimeConverter(
        @JavaTimeConversionPattern("dd.MM.yyyy") LocalDate argument) {

    assertEquals(2017, argument.getYear());
}
```

### 3.15.3. 引数集約
デフォルトでは、`@ParameterizedTest`メソッドに渡される各`引数`は、1つのメソッドパラメータに対応しています。その結果として、大量の引数を供給することが期待される引数ソースは、巨大なメソッドシグネチャになる可能性があります。

そのような場合、[`ArgumentAccessor`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/params/aggregator/ArgumentsAccessor.html)を複数のパラメータの代わりに使うことができます。このAPIを使うことで、テストメソッドに渡された1つのパラメータを通して提供された引数にアクセスすることができます。さらに、[`暗黙的な変換`]()で議論している型変換もサポートしています。

```java
@ParameterizedTest
@CsvSource({
    "Jane, Doe, F, 1990-05-20",
    "John, Doe, M, 1990-10-22"
})
void testWithArgumentsAccessor(ArgumentsAccessor arguments) {
    Person person = new Person(arguments.getString(0),
                               arguments.getString(1),
                               arguments.get(2, Gender.class),
                               arguments.get(3, LocalDate.class));

    if (person.getFirstName().equals("Jane")) {
        assertEquals(Gender.F, person.getGender());
    }
    else {
        assertEquals(Gender.M, person.getGender());
    }
    assertEquals("Doe", person.getLastName());
    assertEquals(1990, person.getDateOfBirth().getYear());
}
```

`ArgumentsAccessor`*のインスタンスは、*`ArgumentsAccessor`*型のいかなるパラメータに自動的に挿入されます*。

#### カスタム集約器
`ArgumentsAccessor`を用いた`@ParameterizedTest`メソッドの引数への直接アクセスとは別に、JUnit Jupiterはカスタムで再利用可能な*集約器*の使用もサポートしています。

カスタム集約器を使うためには、単に[`ArgumentAggregator`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/params/aggregator/ArgumentsAggregator.html)インターフェイスを実装し、`@ParameterizedTest`メソッド内で互換可能なパラメータに対して`@AggregateWith`アノテーションを付与して登録するだけです。集約の結果は、パラメータ化テストが呼び出された時に、対応するパラメータへの引数として提供されます。

```java
@ParameterizedTest
@CsvSource({
    "Jane, Doe, F, 1990-05-20",
    "John, Doe, M, 1990-10-22"
})
void testWithArgumentsAggregator(@AggregateWith(PersonAggregator.class) Person person) {
    // perform assertions against person
}

public class PersonAggregator implements ArgumentsAggregator {
    @Override
    public Person aggregateArguments(ArgumentsAccessor arguments, ParameterContext context) {
        return new Person(arguments.getString(0),
                          arguments.getString(1),
                          arguments.get(2, Gender.class),
                          arguments.get(3, LocalDate.class));
    }
}
```

コードベースにまたがって複数のパラメータ化テストに対して繰り返し`@AggregateWith(MyTypeAggregator.class)`を宣言している場合、`@AggregateWith(MyTypeAggregator.class)`のメタアノテーションとして`@CsvToMyType`のようなカスタム*合成アノテーション*を作成できます。次の例は、カスタム`@CsvToPerson`アノテーションを用いた動作例を示しています。

```java
@ParameterizedTest
@CsvSource({
    "Jane, Doe, F, 1990-05-20",
    "John, Doe, M, 1990-10-22"
})
void testWithCustomAggregatorAnnotation(@CsvToPerson Person person) {
    // perform assertions against person
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
@AggregateWith(PersonAggregator.class)
public @interface CsvToPerson {
}
```

### 3.15.4. 表示名のカスタマイズ
デフォルトでは、パラメータ化テスト呼び出しの表示名は、呼び出しインデックスと特定の呼び出しに対する全ての引数の`String`表現を含んでいます。しかしながら、次の例のように`@ParameterizedTest`アノテーションの`name`属性によって呼び出し表示名をカスタマイズできます。

```java
@DisplayName("Display name of container")
@ParameterizedTest(name = "{index} ==> first=''{0}'', second={1}")
@CsvSource({ "foo, 1", "bar, 2", "'baz, qux', 3" })
void testWithCustomDisplayNames(String first, int second) {
}
```

上記のメソッドを`ConsoleLauncher`を使って実行すると、次のような出力が表示されます。

```
Display name of container ✔
├─ 1 ==> first='foo', second=1 ✔
├─ 2 ==> first='bar', second=2 ✔
└─ 3 ==> first='baz, qux', second=3 ✔
```

カスタム表示名では、次のプレースホルダがサポートされています。

|プレースホルダ|説明|
|---|---|
|{index}|現在の呼び出しインデックス（1始まり）|
|{arguments}|完全な引数リスト（CSV形式）|
|{0}, {1}, ...|各引数|

### 3.15.5. ライフサイクルと相互運用性
パラメータ化テストの各呼び出しは、通常の`@Test`メソッドと同じライフサイクルを持っています。例えば、各呼び出し前には`@BeforeEach`メソッドが実行されます。[`動的テスト`]()と同じように、呼び出しはIDEのテストツリーでは一つ一つ表れます。同一のテストクラスに、自由に`@Test`と`@ParameterizedTest`を混ぜることができます。

`@ParameterizedTest`メソッドと合わせて`ParameterResolver`拡張を使うことができます。しかしながら、引数ソースによって解決されたパラメータは、引数リストの最初に来る必要があります。テストクラスは様々なパラメータリストを持つパラメータ化テストと同様に通常のテストを含むこともあるので、引数ソースからの値は`@BeforeEach`といったライフサイクルメソッドやテストクラスコンストラクタは解決されません。

```java
@BeforeEach
void beforeEach(TestInfo testInfo) {
    // ...
}

@ParameterizedTest
@ValueSource(strings = "foo")
void testWithRegularParameterResolver(String argument, TestReporter testReporter) {
    testReporter.publishEntry("argument", argument);
}

@AfterEach
void afterEach(TestInfo testInfo) {
    // ...
}
```

## 3.16. テストテンプレート
[`@TestTemplate`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/TestTemplate.html)メソッドは、通常のテストケースではなく、むしろテストケースのためのテンプレートです。したがって、`@TestTemplate`は、登録されたプロバイダによって返される呼び出し文脈の数に応じて複数回呼び出されるものとして設計されています。そのため、登録された[`TestTemplateInvocationContextProvider `](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html)拡張と併せて使われる必要があります。テストテンプレートメソッドの各呼び出しは、通常の`@Test`メソッドの実行と同じように振る舞い、同じライフサイクルのコールバックと拡張が完全にサポートされています。用法例については、[テストテンプレートに対する呼び出し文脈の提供]()を参照ください。

## 3.17. 動的テスト
[アノテーション]()で説明したJUnit Jupiterの標準的な`@Test`アノテーションは、JUnit 4の`@Test`アノテーションに非常に似通っています。どちらもテストケースを実装したメソッドです。これらのテストケースはコンパイル時に完全に決定するという意味では静的であり、それらの振る舞いは実行時に変更することはできません。*アサンプションは、意図的にかなり表現性に制限のあるものですが、動的な振る舞いの基本的な形式を提供します*。

これらの標準的なテストに加えて、全く新しい種類のテストプログラミングモデルがJUnit Jupiterでは導入されました。この新しいテストとは、*動的テスト*です。動的テストは、`@TestFactory`が付与されたファクトリーメソッドによって、実行時に生成されます。

`@Test`メソッドとは対照的に、`@TestFactory`メソッド自身はテストケースではなく、むしろテストケースのためのファクトリーです。そのため、動的テストはファクトリーの産出物となります。技術的なことを言うと、`@TestFactory`メソッドは、`DynamicNode`インスタンスの`Stream`または`Collection`、`Iterable`、`Iterator`を返さなければなりません。`DynamicNode`のインスタンス化可能なサブクラスは`DynamicContainer`と`DynamicTest`です。`DynamicContainer`インスタンスは、*表示名*と動的な子ノードのリストで構成されており、動的なノードの任意なネスト階層を生成できます。`DynamicTest`インスタンスは、遅延実行され、テストケースを動的で非決定的な生成が可能となります。

`@TestFactory`によって返される`Stream`はいずれも、`stream.close()`を呼ぶことで適切に閉じられます。これによって、`Files.lines()`のような資源を安全に使うことができます。

`@Test`メソッドと同様に、`@TestFactory`メソッドは`private`または`static`は不可で、`ParameterResolvers`で解決されるパラメータをオプションで宣言できます。

`DynamicTest`は実行時に生成されるテストケースで、*表示名*と`Executable`で構成されています。`Executable`は`@FunctionalInterface`で、このインターフェイスは*ラムダ表現*または*メソッド参照*として提供されることができる動的テストの実装であることを意味します。

> ⚠️  *動的テストのライフサイクル*：動的テストの実行ライフサイクルは、標準的な`@Test`ケースとは全く異なります。特に、各動的テストに対してのライフサイクルのコールバックはありません。このことは、`@BeforeEach`と`@AfterEach`、それに対応した拡張コールバックは`@TestFactory`メソッドに対して実行され、各*動的テスト*には実行されないことを意味します。つまり、動的テストに対してラムダ表現でテストインスタンスからフィールドにアクセスしても、それらのフィールドは、同じ`@TestFactory`メソッドで生成された個々の動的テストの実行中は、コールバックメソッドやその拡張によってリセットされません。

JUnit Jupiter 5.2.0時点では、動的テストは常にファクトリーメソッドによって生成される必要があります。しかしながら、これは後のリリースの登録機能によって補完されるかもしれません。

> ⚠️  動的テストは現在*実験的な*機能です。詳細に関しては、[実験的なAPIs]()をご覧ください。

### 3.17.1. 動的テストの例
次の`DynamicTestsDemo`クラスは、テストファクトリーと動的テストのいくつかの例を示しています。

最初のメソッドは不正な型を返しています。不正な返り値の型はコンパイル時に検出することができないため、実行時に検出され`JUnitException`が投げられます。

次の5つもメソッドは、`DynamicTest`インスタンスの`Collection`または`Iterable`、`Iterator`、`Stream`を生成する非常に単純な例です。これらの例のほとんどは実際には動的振る舞いを示しておらず、単に原則的にサポートされている返却型を示しています。しかしながら、`dynamicTestsFromStream()`と`dynamicTestsFromIntStream()`は、`String`のセットや入力値の範囲に対する動的テストの生成がいかに簡単かを示しています。

次のメソッドは、性質上、真に動的なものです。`generateRandomNumberOfTests()`、ランダム数を生成する`Iterator`と表示名生成器、テスト実行器を実装しており、その3つを`DynamicTest.stream()`に提供しています。`generateRandomNumberOfTests()`の非決定的な振る舞いは、もちろんテスト反復可能性に抵触しており、注意深く取り扱われるべきではありますが、動的テストの表現性と能力を示しています。

最後のメソッドは、`DynamicContainer`を使って動的テストのネスト階層を生成しています。

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.DynamicContainer.dynamicContainer;
import static org.junit.jupiter.api.DynamicTest.dynamicTest;

import java.util.Arrays;
import java.util.Collection;
import java.util.Iterator;
import java.util.List;
import java.util.Random;
import java.util.function.Function;
import java.util.stream.IntStream;
import java.util.stream.Stream;

import org.junit.jupiter.api.DynamicNode;
import org.junit.jupiter.api.DynamicTest;
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.TestFactory;
import org.junit.jupiter.api.function.ThrowingConsumer;

class DynamicTestsDemo {

    // This will result in a JUnitException!
    @TestFactory
    List<String> dynamicTestsWithInvalidReturnType() {
        return Arrays.asList("Hello");
    }

    @TestFactory
    Collection<DynamicTest> dynamicTestsFromCollection() {
        return Arrays.asList(
            dynamicTest("1st dynamic test", () -> assertTrue(true)),
            dynamicTest("2nd dynamic test", () -> assertEquals(4, 2 * 2))
        );
    }

    @TestFactory
    Iterable<DynamicTest> dynamicTestsFromIterable() {
        return Arrays.asList(
            dynamicTest("3rd dynamic test", () -> assertTrue(true)),
            dynamicTest("4th dynamic test", () -> assertEquals(4, 2 * 2))
        );
    }

    @TestFactory
    Iterator<DynamicTest> dynamicTestsFromIterator() {
        return Arrays.asList(
            dynamicTest("5th dynamic test", () -> assertTrue(true)),
            dynamicTest("6th dynamic test", () -> assertEquals(4, 2 * 2))
        ).iterator();
    }

    @TestFactory
    Stream<DynamicTest> dynamicTestsFromStream() {
        return Stream.of("A", "B", "C")
            .map(str -> dynamicTest("test" + str, () -> { /* ... */ }));
    }

    @TestFactory
    Stream<DynamicTest> dynamicTestsFromIntStream() {
        // Generates tests for the first 10 even integers.
        return IntStream.iterate(0, n -> n + 2).limit(10)
            .mapToObj(n -> dynamicTest("test" + n, () -> assertTrue(n % 2 == 0)));
    }

    @TestFactory
    Stream<DynamicTest> generateRandomNumberOfTests() {

        // Generates random positive integers between 0 and 100 until
        // a number evenly divisible by 7 is encountered.
        Iterator<Integer> inputGenerator = new Iterator<Integer>() {

            Random random = new Random();
            int current;

            @Override
            public boolean hasNext() {
                current = random.nextInt(100);
                return current % 7 != 0;
            }

            @Override
            public Integer next() {
                return current;
            }
        };

        // Generates display names like: input:5, input:37, input:85, etc.
        Function<Integer, String> displayNameGenerator = (input) -> "input:" + input;

        // Executes tests based on the current input value.
        ThrowingConsumer<Integer> testExecutor = (input) -> assertTrue(input % 7 != 0);

        // Returns a stream of dynamic tests.
        return DynamicTest.stream(inputGenerator, displayNameGenerator, testExecutor);
    }

    @TestFactory
    Stream<DynamicNode> dynamicTestsWithContainers() {
        return Stream.of("A", "B", "C")
            .map(input -> dynamicContainer("Container " + input, Stream.of(
                dynamicTest("not null", () -> assertNotNull(input)),
                dynamicContainer("properties", Stream.of(
                    dynamicTest("length > 0", () -> assertTrue(input.length() > 0)),
                    dynamicTest("not empty", () -> assertFalse(input.isEmpty()))
                ))
            )));
    }

}
```
