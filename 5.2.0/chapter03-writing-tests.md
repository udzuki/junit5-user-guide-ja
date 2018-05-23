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
|`@Test`|テストメソッドであることを意味します。JUnit 4の`@Test`と異なり、このアノテーションはどのような属性も宣言しません。これは、JUnit Jupiterにおけるテスト拡張が専用のアノテーションを元に作動するからです。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@ParameterizedTest`|[パラメータ化テスト]()であることを意味します。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@RepeatedTest`|[繰り返しテスト]()のテンプレートメソッドであることを意味します。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@TestFactory`|[動的テスト]()のファクトリーメソッドであることを意味します。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@TestInstance`|[テストインスタンス・ライフサイクル]()を設定するためにクラスに付与します。アノテーションは*継承*されます。|
|`@TestTemplate`|[テストケースのテンプレート]()メソッドであることを意味します。テンプレートは登録された[プロバイダ]()によって返される呼び出しコンテキストの数に応じて複数回呼び出されます。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@DisplayName`|テストクラス、もしくはテストメソッドのカスタム表示名を意味します。アノテーションは*継承*されません。|
|`@BeforeEach`|現在のクラス内にある**各テスト**（`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`）が実行される*前*（*before*）に実行されるメソッドを意味します。JUnit 4の`@Before`と類似したものです。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@AfterEach`|現在のクラス内にある**各テスト**（`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`）が実行された*後*（*after*）に実行されるメソッドを意味します。JUnit 4の`@After`と類似したものです。メソッドは*オーバーライド*されない限り、*継承*されます。|
|`@BeforeAll`|現在のクラス内にある**全テスト**（`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`）が実行される*前*（*before*）に実行されるメソッドを意味します。JUnit 4の`@BeforeClass`と類似したものです。メソッドは*隠される*か*オーバーライド*されない限り、*継承*され、`static`でないといけません("クラスごと"[テストインスタンス・ライフサイクル]()を使わない限り)。|
|`@AfterAll`|現在のクラス内にある**全テスト**（`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`）が実行された*後*（*after*）に実行されるメソッドを意味します。JUnit 4の`@AfterClass`と類似したものです。メソッドは*隠される*か*オーバーライド*されない限り、*継承*され、`static`でないといけません("クラスごと"[テストインスタンス・ライフサイクル]()を使わない限り)。|
|`@Nested`|ネストされたnon-staticなテストクラスであることを意味します。`@BeforeAll`と`@AfterAll`メソッドは、"クラスごと"[テストインスタンス・ライフサイクル]()を使わない限り、`@Nested`クラスの中では直接使うことができません。メソッドは*継承*されません。|
|`@Tag`|テストをクラスレベル、もしくはメソッドレベルでフィルタリングするための*タグ*を宣言することができます。TestNGのtest groups、もしくはJUnit 4のCategoriesと類似したものです。アノテーションはクラスレベルでは*継承*されますが、メソッドレベルでは*継承*されません。|
|`@Disabled`|テストクラス、もしくはテストメソッドを*無効化*することができます。JUnit 4の`@Ignore`と類似したものです。アノテーションは*継承*されません。|
|`@ExtendWith`|カスタム[拡張]()を登録することができます。アノテーションは*継承*されます。|

次のアノテーションをつけたメソッドは値を返してはいけません。
(`@Test`, `@TestTemplate`, `@RepeatedTest`, `@BeforeAll`, `@AfterAll`, `@BeforeEach`, `@AfterEach`)

> ⚠️ 注意点：いくつかのアノテーションは現在 *experimental*にあるかもしれません。詳細に関しては、[実験的なAPIs]()をご覧ください。

### 3.1.1. メタアノテーションと組み合わせアノテーション
JUnit Jupiterアノテーションはメタアノテーションとして使うことができます。それはつまり、メタアノテーションの意味を自動で*継承*する独自の*組み合わせアノテーション*を定義することができるということです。

例えば、コードベースに`@Tag("fast")`（[タグ付けとフィルタリング]()をご覧ください。）をコピー＆ペーストする代わりに、あなたは`@Fast`というカスタム*組み合わせアノテーション*を、次のように作ることができます。`@Fast`は`@Tag("fast")`の代替として使うことができます。

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
*テストメソッド*とは、直接もしくはメタ的に`@Test`か`@RepeatedTest`、`@ParamterizedTest`、`@TsetFactory`、もしくは`@TestTemplate`が付与されたインスタンスメソッドです。*テストクラス*とは、最低一つでもテストメソッドを含むトップレベルクラスかstaticなメンバークラスのことです。

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
テストクラスとテストメソッドはカスタムされた表示名 - スペース、特殊文字、さらには絵文字だって使えます - を宣言することができます。それらがテストランナーとテストレポートによって表示されます。

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
JUnit Jupiterには、JUnit 4が持っていたアサーションメソッドの多くを持っています。また、いくつかはJava 8のラムダ式で使うことができます。全てのJUnit Jupiterアサーションは、[`org.junit.jupiter.api.Assertions`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/Assertions.html)クラスの`static`メソッドです。

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

### 3.4.1. サードパーティによるアサーションライブラリ
JUnit Jupiterによって提供されているアサーション機能は、多くのテストシナリオには十分なものだと思いますが、*matchers*のような、より強力で追加的な機能が求められたり、必要であることがあります。そのような場合、JUnitチームは、[AssertJ](http://joel-costigliola.github.io/assertj/)、[Hamcrest](http://hamcrest.org/JavaHamcrest/)、[Truth](http://google.github.io/truth/)などといったサードパーティによるアサーションライブラリの使用をお薦めします。したがって、開発者は自由に選んだアサーションライブラリを使うことができます。

例えば、*matchers*と流暢なAPI（fluent API）の組み合わせは、アサーションをよりわかりやすく、読みやすくするために使うことができます。しかしながら、JUnit Jupiterの[`org.junit.jupiter.Assertions`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/Assertions.html)クラスは、Hamcrestの[`Matcher`](http://junit.org/junit4/javadoc/latest/org/hamcrest/Matcher.html)を受け入れていたJUnit 4の`org.junit.jupiter.Assert`クラスの`assertThat()`メソッドのようなものを提供していません。代わりに、開発者はサードパーティによるアサーションライブラリによって提供されているマッチャー用のサポートを使うことが奨励されています。

次の例は、JUnit Jupiterテストにおいて、どのようにしてHamcrestからの`assertThat()`サポートを使うかを説明しています。Hamcrestライブラリがクラスパスに加えられている限り、`assertThat()`や`is()`、`equalTo()`といったメソッドをstaticにインポートすることができます。また、それらをテストの中で、下に示す`assertWithHamcrestMatcher()`のように使うことができます。

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

当然、JUnit 4のプログラミングモデルに基づいたレガシーコードも`org.junit.Assert#assertThat`を使い続けることができます。

## 3.5. アサンプション
JUnit Jupiterには、JUnit 4が持っていたアサンプションメソッドのサブセットを持っており、いくつかはJava 8のラムダ式で使うことができます。全てのJUnit Jupiterアサンプションは、[`org.junit.jupiter.api.Assumptions`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/Assumptions.html)クラスの`static`メソッドです。

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
テストクラス全体、もしくは各テストメソッドは[`@Disabled`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/Disabled.html)アノテーションか、[条件付きテスト実行]()で話されているアノテーションの一つ、もしくはカスタム[`実行条件`]()によって*無効化*することができます。

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

## 3.7. 条件的なテスト実行
JUnit Jupiterの[実行条件]()拡張APIによって、開発者は、*プログラム的に*コンテナ、またはある条件に基づいたテストを*有効*にしたり*無効*にすることができます。そのような条件の最も単純な例は、`@Disabled`をサポートしているビルトインの[`DisabledCondition`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/DisabledCondition.java)です（[テストを不可能にする]()をご覧ください）。`@Disabled`に加えて、JUnit Jupiterは、`org.junit.jupiter.api.condition`パッケージ内の他のいくつかのアノテーションベースの条件もサポートしています。`org.junit.jupiter.api.condition`パッケージによって、開発者は*宣言的に*コンテナやテストを*有効*にしたり*無効*にすることができます。詳細については、次のセクションをご覧ください。

> 💡 組み合わせアノテーションのヒント：次のセクションで挙げられている条件アノテーションはどれも、組み合わせアノテーションを作るためのメタアノテーションとして使えるかもしれない。例えば、[@EnabledOnOsのデモ]()にある`@TestOnMac`アノテーションは、どのようにして`@Test`と`@EnableOnOs`を一つの再利用できるアノテーションで結合できるかを示しています。



> ⚠️ 次のセクションで挙げられている*条件*アノテーションはそれぞれ、テストインターフェイス、テストクラス、またはテストメソッドに一度だけ宣言することができます。もし条件アノテーションが直接的か間接的、またはメタ的に、ある要素に複数存在していた場合、JUnitによって発見された初めのアノテーションが使われます；他の追加的なアノテーションは静かに無視されます。しかしながら、`org.junit.jupiter.api.condition`パッケージでは、各条件アノテーションは他の条件アノテーションと共に使われているかもしれません。

### 3.7.1. オペレーティングシステムの条件
[`@EnabledOnOs`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledOnOs.html)と[`@DisabledOnOs`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledOnOs.html)を使うことで、特定のオペーティングシステム上でコンテナかテストを、有効にしたり無効にすることができます。

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

### 3.7.2. Java実行時条件
[`@EnabledOnJre`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledOnJre.html)と[`@DisabledOnJre`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledOnJre.html)を使うことで、特定のバージョンのJava実行環境（JRE）でコンテナかテストを、有効にしたり無効にすることができます。

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

### 3.7.3. システムプロパティ条件
[`@EnabledIfSystemProperty`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledIfSystemProperty.html)と[`@DisabledIfSystemProperty`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledIfSystemProperty.html)を使うことで、`named`で指定したJVMシステムプロパティの値に応じてコンテナかテストを、有効にしたり無効にすることができます。`matches`属性を使うことで、値は正規表現として解釈されます。

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

### 3.7.4. 環境変数条件
[`@EnabledIfEnvironmentVariable`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledIfEnvironmentVariable.html)と[`@DisabledIfEnvironmentVariable`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledIfEnvironmentVariable.html)を使うことで、`named`で指定したOSの環境変数の値に応じてコンテナかテストを、有効にしたり無効にすることができます。`matches`属性を使うことで、値は正規表現として解釈されます。

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
JUnit Jupiterは、[`@EnabledIf`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledIf.html)と[`@DisabledIf`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledIf.html)を使うことで、設定されたスクリプトの評価値に応じてコンテナかテストを、有効にしたり無効にすることができる機能を提供しています。スクリプトは、JavaScriptかGroovy、またはJSR 223で定義されたJava Scripting APIをサポートしているスクリプト言語であれば何でも、書くことができます。

> ⚠️  [`@EnabledIf`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/EnabledIf.html)と[`@DisabledIf`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/condition/DisabledIf.html)を使った条件付きテスト実行は、現在*実験的な*機能です。詳細については、[*実験的な* APIs]()をご覧ください。

> 💡 もしスクリプトのロジックが、今使っているOSか、JREのバージョン、またはあるJVMシステムプロパティや環境変数に依存している場合、その目的に合ったビルトインアノテーションを使うといいかもしれません。詳細については、この章の前のセクションをご覧ください。

> ℹ️ もし同じスクリプトベースの条件を多数使っている場合、より速く、型安全で、メンテナンスのしやすい条件を実装するために、それに合った[実行条件]()拡張を書くことを考えてみてください。

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
次の名前は、各スクリプト条件とバインドされています。そのため、スクリプトで使用可能です。*accessor*は、単純な `String get(String name)`メソッドを通して、マップライク（map-like）な構造へのアクセスを提供します。

|Name|Type|Description|
|---|---|---|
|systemEnvironment|*accessor*|OS環境変数アクセサ|
|systemProperty|*accessor*|JVMシステムプロパティアクセサ|
|junitConfigurationParameter|*accessor*|Configurationパラメータアクセサ|
|junitDisplayName|String|テストかコンテナの表示名|
|junitTags|Set\<String\>|テストかコンテナに振られている全てのタグ|
|junitUniqueId|String|テストかコンテナのユニークなID|

## 3.8. タグとフィルタリング
テストクラスとメソッドは`@Tag`を用いてタグ付けすることができます。それらのタグは、後で[テスト発見と実行]()をフィルタリングするのに使われます。

### 3.8.1. タグのシンタックスルール
- タグは`null`か*空*であってはならない。
- *トリミングされた*タグは空白文字を含んではならない。
- *トリミングされた*タグはISO制御文字を含んではならない。
- *トリミングされた*タグは次の*予約語*のいずれも含んではならない。
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

> ℹ️ `@Disabled`や`@DisabledOnOs`といった条件によって*無効化*された*テストメソッド*であっても、テストクラスはインスタンス化されることに注意してください。これは例え"メソッドごと"テストインスタンス・ライフサイクルが有効である時でも同様です。

もしJUnit Jupiterに全テストメソッドを同じテストインスタンスで実行してほしい場合は、単にテストクラスに`@TestInstance(Lifecycle.PER_CLASS)`アノテーションを付与するだけで実現可能です。このモードを使用する場合、新しいテストインスタンスは一度だけ生成されます。これによって、もしテストメソッドがインスタンス変数に保存された状態に依存する場合は、`@BeforeEach`か`AfterEach`メソッドによって状態をリセットする必要があるかもしれません。

”クラスごと”モードは、デフォルトの"メソッドごと"モードに比べていくつかの付加的な便益があります。特に"クラスごと"モードを使うと、インターフェイスの`default`メソッドと同様に、`@BeforeAll`と`@AfterAll`メソッドをnon-staticメソッドとして宣言することが可能になります。そのため、"クラスごと"モードでは、`@Nested`テストクラス内で`@BeforeAll`と`@AfterAll`メソッドを使うことができます。

Kotlinでテストを書いている場合、”クラスごと”テストインスタンス・ライフサイクルモードに切り替えることで、`@BeforeAll`と`@AfterAll`メソッドの実装がより容易になるかもしれません。

### 3.9.1. デフォルトのテストインスタンス・ライフサイクルを変更する
テストクラスかテストインターフェイスに`@TestInstance`が付与されていない場合、JUnit Jupiterは*デフォルト*のライフサイクルモードを使います。
標準的な*デフォルト*モードは`PER_METHOD`です；しかしながら、テスト計画全体の*デフォルト*を変更することが可能です。デフォルトのテストインスタンス・ライフサイクルを変更するには、単に`junit.jupiter.testinstance.lifecycle.default`*設定パラメータ*に、`TestInstance.Lifecycle`で定義されたenum定数の値を、大文字・小文字を無視してセットするだけです。これは、JVMシステムプロパティとして渡すか、`Launcher`に渡される`LauncherDiscoveryRequest`内の*設定パラメータ*として渡すか、JUnit Platform設定ファイル(詳細については、[パラメータの設定]()をご覧ください。)を通して渡します。

例えば、デフォルトのテストインスタンス・ライフサイクルを`LifeCycle.PER_CLASS`に設定するには、JVMを次のシステムプロパティで起動することです。

`-Djunit.jupiter.testinstance.lifecycle.default=per_class`

しかしながら、JUnit Platform設定ファイルを通してデフォルトのテストインスタンス・ライフサイクルを設定する場合は、次の内容を含んだ`junit-platform.properties`という名前のファイルをクラスパスのルート（例えば、`src/test/resources`）に生成する必要があることに注意してください。

`junit.jupiter.testinstance.lifecycle.default = per_class`

> ⚠️  *デフォルト*のテストインスタンス・ライフサイクルを変更することは、一貫性を持って適用しないと、予測不可能な結果と壊れやすいビルドを導く恐れがあります。例えば、デフォルトとして”クラスごと”をビルド設定していながら、IDE上で"メソッドごと"でテスト実行されてしまった場合、ビルドサーバに表れるエラーをデバッグすることは困難になる恐れがあります。そのため、JVMシステムプロパティの代わりに、JUnit Platform設定ファイルを使ってデフォルトモードを変更することをお薦めします。

## 3.10. ネストされたテスト
ネストされたテストは、テスト開発者が様々なグループのテスト間の関係を表現することを可能にします。これがその美しい例です。

*スタックをテストするためのネストされたテストです。*
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

> ℹ️ *non-staticなネストされたクラス（つまり、インナークラス）のみ*が`@Nested`テストクラスとなります。ネストは任意に深くすることができ、それらのインナークラスは一つの例外を除いて、テストクラスの完全なメンバーとして考えられます。例外とは、`@BeforeAll`と`@AfterAll`が*デフォルト*では作動しないことです。その理由は、Javaがインナークラスに`static`なメンバーを許さないためです。しかしながら、この制限は`@Nested`テストクラスに`@TestInstance(Lifecycle.PER_CLASS)`を付与することで回避することができます（[テストインスタンス・ライフサイクル]()をご覧ください）。

## 3.11. コンストラクタとメソッドへの依存性注入
全てのJUnitの前バージョンでは、テストコンストラクタかメソッドは、パラメータを持つことが許されていませんでした（少なくとも標準的な`Runner`実装の下では）。JUnit Jupiterでの大きな変化の一つとして、テストコンストラクタとテストメソッドのどちらもパラメータを持つことが許されたことがあります。このことは、大きな柔軟性をもたらし、コンストラクタとメソッドに*依存性を注入*することが可能になりました。

[`ParameterResolver`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ParameterResolver.html)は、実行時に*動的に*パラメータを解決することを望むテスト拡張のためのAPIを定義しています。もしテストコンストラクタか、`@Test`、`@TestFactory`、`@BeforeEach`、`@AfterEach`、`@BeforeAll`、`@AfterAll`メソッドがパラメータを許容する時は、パラメータは登録された`ParameterResolver`によって実行時に解決されないといけません。

現在は、3つのビルトイン・リゾルバが自動的に登録されています。

- [`TestInfoParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestInfoParameterResolver.java)：もしメソッドパラメータが[`TestInfo`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/TestInfo.html)型であった場合、`TestInfoParameterResolver`は、現在のテストに応じた`TestInfo`のインスタンスをパラメータの値として供給します。`TestInfo`は、テストの表示名、テストクラス、テストメソッド、付いているタグ名といった現在のテストに関する情報を集めるのに使うことができます。表示名は、テストクラス名かテストメソッド名といった技術的な名前か、`@DisplayedName`で設定されたカスタム名のどちらかです。[`TestInfo`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/TestInfo.html)は、JUnit 4の`TestName`の代替のようなものです。次のコードは、テストコンストラクタ、`@BeforeEach`、`@Test`メソッドにどのように`TestInfo`を注入させるかを示しています。

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


- [`RepetitionInfoParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/RepetitionInfoParameterResolver.java)：もし`@RepeatedTest`、`@BeforeEach`、`@AfterEach`メソッドのパラメータが[`RepetitionInfo`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/RepetitionInfo.html)型であった場合、`RepetitionInfoParameterResolver`は、`RepetitionInfo`のインスタンスを供給します。`RepetitionInfo`は、現在の繰り返しと対応する`@RepeatedTest`の繰り返しの総回数に関する情報を集めるのに使うことができます。しかしながら、`RepetitionInfoParameterResolver`は、`@RepeatedTest`以外の文脈以外では登録されていないことに注意してください。[繰り返しテストの例]()をご覧ください。
- [`TestReporterParameterResolver`](https://github.com/junit-team/junit5/tree/r5.2.0/junit-jupiter-engine/src/main/java/org/junit/jupiter/engine/extension/TestReporterParameterResolver.java)：もしメソッドパラメータが[`TestReporter`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/TestReporter.html)型であった場合、`TestReporterParameterResolver`は`TestReporter`のインスタンスを供給します。`TestReporter`は、現在のテスト実行に関する付加的な情報を公開することに使うことができます。データは、[`TestExecutionListener.reportingEntryPublished()`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/TestExecutionListener.html)を通して消費され、それによってIDEから閲覧できたりレポートに含まれます。JUnit Jupiterでは、JUnit 4で `stdout`や`stderr`を使って情報を出力していた箇所に`TestReporter`を使うことができます。`@RunWith(JUnitPlatform.class)`を使うと、全てのレポートされたエントリを`stdout`に出力します。

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

> ℹ️ 他のパラメータリゾルバは、`@ExtendedWith`を用いた適切な[拡張]()を登録することによって明示的に有効化する必要があります。

カスタム[`ParameterResolver`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/ParameterResolver.html)の例のために、[`RandomParametersExtension`](https://github.com/junit-team/junit5-samples/tree/r5.2.0/junit5-jupiter-extensions/src/main/java/com/example/random/RandomParametersExtension.java)を確認しましょう。リリース可能なものではありませんが、拡張モデルとパラメータ解決プロセス両方の単純さと表現性を例示しています。`MyRandomParametersTest`は、どのように`@BeforeEach`と`@Test`メソッド内にMockitoモックを注入するかを示しています。

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

現実的なユースケースとしては、[`MockitoExtension`](https://github.com/mockito/mockito/blob/release/2.x/subprojects/junit-jupiter/src/main/java/org/mockito/junit/jupiter/MockitoExtension.java)と[`SpringExtension`](https://github.com/spring-projects/spring-framework/tree/master/spring-test/src/main/java/org/springframework/test/context/junit/jupiter/SpringExtension.java)のソースコードを確認してください。

## 3.12. テストインターフェイスとデフォルトメソッド
JUnit Jupiterは、`@Test`、`@RepeatedTest`、`@ParameterizedTest`、`@TestFactory`、`@TestTemplate`、`@BeforeEach`、`@AfterEach`にインターフェイスデフォルトメソッドを宣言できるようにしています。`@BeforeAll`と`@AfrterAll`はテストインターフェイス内で`static`メソッドを宣言するか、*もし*テストクラスに`@TestInstance(Lifecycle.PER_CLASS)`が付与されている場合はインターフェイスデフォルトメソッドを宣言することができます（[テストインスタンス・ライフサイクル]()をご覧ください）。いくつかの例を示します。

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

`@ExtenedWoth`と `@Tag`はテストインターフェイスとして宣言することができるため、インターフェイスを実装したクラスは自動的にタグと拡張を継承します。[`タイミング拡張`]()のソースコードを見るには、[`BeforeとAfterのテスト実行コールバック`]()をご覧ください。

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

`TestInterfaceDemo`の実行結果の出力は、次のものと似たようなものになります。

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

この機能の他のあり得る適用としては、インターフェイス契約のためにテストを書くことです。例えば、`Object.equals`か`Comparable.compareTo`の実装がどう振る舞うべきかのテストを、次のように書くことができます。

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

テストクラスは、両方の契約インターフェイスを実装することができ、それによって対応するテストを継承します。もちろん、抽象メソッドを実装する必要があります。

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
JUnit Jupiterは、`@RepeatedTest`を付与し、繰り返してほしい回数を設定するだけで、特定回数テストを繰り返す機能を提供しています。繰り返しテストの各呼び出しは、通常の`@Test`メソッドの実行のように振る舞い、全く同じライフサイクル・コールバックと拡張をサポートしています。

次の例は、どのようにして自動で10回繰り返す`repeatedTest()`の宣言するかを示しています。

```java
@RepeatedTest(10)
void repeatedTest() {
    // ...
}
```

繰り返し回数の設定に加えて、`@RepeatedTest`の`name`属性を用いることで、カスタム表示名も設定することができます。さらに、表示名は、静的テキストと動的プレースホルダの組み合わせのパターンにすることもできます。次のプレースホルダが現在サポートされています。

- `{displayName}`:`@RepeatedTest`メソッドの表示名
- `{currentRepetition}`: 現在の繰り返し回数
- `{totalRepetition}`: 繰り返し回数の合計

ある繰り返し回数時点でのデフォルトの表示名は、次のパターンに基づいて生成されます：`"repetition {currentRepetition} of {totalRepetitions}"`。そのため、先ほどの例の各繰り返し回数における表示名は次のようになります：`repetition 1 of 10`、`repetition 2 of 10`、などなど。もし`@RepeatedTest`の表示名を、各繰り返し回数での表示名に含めたい場合は、独自のカスタムパターンを定義するか、事前定義された`RepeatedTest.LONG_DISPLAY_NAME`パターンを使うことができます。後者は、`"{displayName} :: repetition {currentRepetition} of {totalRepetitions}"`と等しいもので、結果は`repeatedTest() :: repetition 1 of 10`、`repeatedTest() :: repetition 2 of 10`となります。

現在の繰り返し回数と繰り返しの合計数の情報をプログラムから集めるためには、開発者は`@RepeatedTest`か`@BeforeEach`、もしくは`@AfterEach`に`RepetitionInfo`インスタンスを注入することができます。

### 3.13.1. 繰り返しテストの例
このセクションの最後にある`RepeatedTestsDemo`クラスは、繰り返しテストのいくつかの例を示しています。

`repeatedTest()`メソッドは、前のセクションからの例です。一方、`repeatedTestWithRepetitionInfo()`は、繰り返しの合計数と現在の繰り返し回数を得るために、どのようにして`RepetitionInfo`インスタンスをテストに注入すればいいかを示しています。

その次の2つのメソッドは、どのようにして`@RepeatedTest`のカスタム`@DisplayName`を各繰り返しの表示名内に含ませるかを示しています。`customDisplayName()`は、カスタム表示名とカスタムパターンを組み合わせており、`TestInfo`を使って生成された表示名のフォーマットを検証しています。`Repeat!`は`@DisplayName`から来る`{displayName}`で、`1/1`は`{currentRepetition}/{totalRepetitions}`から来ています。対照的に、`customDisplayNameWithLongPattern()`は、先ほど説明した事前定義の`RepeatedTest.LONG_DISPLAY_NAME`パターンを使っています。

`repeatedTestInGerman()`は、繰り返しテストの表示名を他国言語（この場合はドイツ語です）に翻訳する機能を示しています。その結果、各繰り返しにおける名前は、`Wiederholung 1 von 5`、`Wiederholung 2 von 5`のようになります。

`beforeEach()`メソッドは`@BeforeEach`が付与されているため、各繰り返しテストの各繰り返し前に実行されます。`TestInfo`と`RepetitionInfo`をこのメソッドに注入することで、現在実行されている繰り返しテストに関する情報を得ることができます。`INFO`ログレベルで、`RepeatedTestsDemo`を実行すると出力は次のようになります。

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

`ConsoleLauncher`か、unicodeテーマを有効化した`junitPlatformTest`Gradleプラグインを使うと、`RepeatedTestsDemo`の実行結果は次のようなコンソール出力を行います。

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
パラメータ化テストを使うと、テストを異なる引数で複数回実行できるようになります。パラメータ化テストは、通常の`@Test`の代わりに[`@ParameterizedTest`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/params/ParameterizedTest.html)を付与するだけで宣言することができます。さらに、各呼び出して供給され、テストで消費される引数として、最低でも1つの*source*を宣言する必要があります。

次の例は、パラメータ化テストを示していて、`@ValueSource`を使って`String`配列を引数のソースに設定しています。

```
@ParameterizedTest
@ValueSource(strings = { "racecar", "radar", "able was I ere I saw elba" })
void palindromes(String candidate) {
    assertTrue(isPalindrome(candidate));
}
```

上記のパラメータ化テストメソッドを実行すると、各呼び出しは別々に報告されます。例えば、`ConsoleLauncher`は次のようなものを出力します。

```
palindromes(String) ✔
├─ [1] racecar ✔
├─ [2] radar ✔
└─ [3] able was I ere I saw elba ✔
```

> ⚠️  パラメータ化テストは、現在*実験的な*機能です。詳細については、[実験的なAPIs]()をご覧ください。

### 3.14.1. 必要なセットアップ
パラメータ化テストを使うためには、`junit-jupiter-params`を依存関係に加える必要があります。詳細については、[依存性のメタデータ]()をご覧ください。

## 3.15. 引数の消費
典型的なパラメータ化テストメソッドは、引数ソースインデックスとメソッドパラメータインデックス間の1対1相関（[`@CsvSource`]()の例をご覧ください）に従って、設定されたソース（[引数のソース]()をご覧ください。）から直接、引数を*消費*します。しかしながら、パラメータ化テストメソッドは、ソースから得た引数をひとつのオブジェクトに*集約*して、メソッドに渡すこともできます（[引数集約]()をご覧ください）。追加的な引数もまた、`ParameterResolver`によって提供されます（例えば、`TestInfo`や`TestReporter`のインスタンスなど）。特に、パラメータ化テストメソッドは、次のルールに従って形式的なパラメータを宣言する必要があります。

- 0個以上の*インデックスされた引数*が最初に宣言されないといけない。
- 0個以上の*集約器*が次に宣言されないといけない。
- 0個以上の`ParameterResolver`によって供給される引数が最後に宣言されないといけない。

この文脈で、*インデックスされた引数*というのは、`ArgumentsProvider`によって提供される`Arguments`内で与えられたインデックスに対応する引数です。*インデックスされた引数*は、引数として、メソッドの形式パラメータリストと同じインデックスの箇所に渡されます。*集約器*は、`ArgumentsAccessor`型の全てのパラメータか、`@AggregateWith`の付与された全てのパラメータです。

### 3.15.1. 引数のソース
すぐに使えるように、JUnit Jupiterはかなりの数の*ソース*アノテーションを提供しています。次の各セクションはそれぞれ、簡潔な概要とそれぞれの例を提供しています。追加的な情報に関しては、[`org.junit.jupiter.params.provider`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/params/provider/package-summary.html)パッケージのJavaDocを参照してください。

#### `@ValueSource`
`@ValueSource`は最も単純なソースの一つです。リテラル値の配列を1つ設定することができ、パラメータ化テスト呼び出しにつき、1つのパラメータを提供することができます。

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
`@EnumSource`は、`Enum`定数に対して便利な機能を提供します。`@EnumSource`は、使われる定数を特定するために、任意の`names`パラメータを提供します。もし指定されていない場合は、次の例のように全ての定数が使われます。

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

`@EnumSource`はまた、テストメソッドに渡すパラメータを細かく制御するために、任意の`mode`パラメータを提供します。例えば、次の例では、enum定数プールからnamesを取り除いたり、正規表現を設定しています。

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
`@MethodSource`では、テストクラスか外部クラスの*ファクトリー*メソッドを1つ以上使うことができます。ファクトリーメソッドは、`Stream`、`Iterable`、`Iterator`、または引き通の配列を返す必要があります。さらに、ファクトリーメソッドは、引数を受けてはいけません。テストクラス内のファクトリーメソッドは、テストクラスに`@TestInstance(Lifecycle.PER_CLASS)`が付与されていない限り、`static`である必要があります；一方、外部クラスのファクトリーメソッドは常に`static`である必要があります。

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

`@MethodSource`を通して明示的にファクトリーメソッドの名前を提供しない場合、JUnit Jupiterは、慣習で現在の`@ParameterizedTest`と同じ名前を持つ*ファクトリー*メソッドを探します。これを次の例で示します。

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

`DoubleStream`、`IntStream`、`LongStream`といったプリミティブ型のStreamもまた、次の例のようにサポートされています。

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

テストメソッドが複数のパラメータを宣言している場合、下に示すように`Arguments`のコレクションかストリームを返す必要があります。`Arguments.of(Object...)`は、`Arguments`インターフェイスで定義されている`static`なファクトリーメソッドです。

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
`@CsvSource`は、引数リストをCSVとして表現できるようにします（つまり、`String`リテラルです）。

```java
@ParameterizedTest
@CsvSource({ "foo, 1", "bar, 2", "'baz, qux', 3" })
void testWithCsvSource(String first, int second) {
    assertNotNull(first);
    assertNotEquals(0, second);
}
```

`@CsvSource`は、シングルクォーテーション`'`を引用文字として使います。上の例と下の表の`'baz, qux'`の値をご覧ください。引用された空の値`''`は、空の`String`となります；一方、完全に*空*の値は`null`参照として解釈されます。`null`参照のターゲット値がプリミティブ型の場合、`ArgumentConversionException`が投げられます。

|Example Input|Resulting Argument List|
|---|---|
|`@CsvSource({ "foo, bar" })`|`"foo"`, `"bar"`|
|`@CsvSource({ "foo, 'baz, qux'" })`|`"foo"`, `"baz, qux"`|
|`@CsvSource({ "foo, ''" })`|`"foo"`, `""`|
|`@CsvSource({ "foo, " })`|`"foo"`, `null`|

#### `@CsvFileSource`
`@CsvFileSource`は、CSVファイルをクラスパスから使えるようにします。CSVファイルの各行がパラメータテストの1呼び出しに相当しています。

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

> ℹ️ `@CsvSource`で使われているシンタックスとは対照的に、`@CsvFileSource`では引用文字としてダブルクォーテーション`"`を使います。上記の例の`"United States of America"`をご覧ください。引用された空の値`””`は、空の`String`となります；一方、完全に*空*の値は`null`参照として解釈されます。`null`参照のターゲット値がプリミティブ型の場合、`ArgumentConversionException`が投げられます。

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
#### 広げる変換
JUnit Jupiterは`@ParamterizedTest`に供給する引数のために、[広げるプリミティブ変換](https://docs.oracle.com/javase/specs/jls/se8/html/jls-5.html#jls-5.1.2)をサポートしています。例えば、`@ValueSource(ints = { 1, 2, 3 })`が付与されたパラメータ化テストは、`int`型のみならず、`long`、`float`、`double`型の引数も受けることができます。

#### 暗示的な変換
`@CsvSource`のようなユースケースをサポートするために、JUnit Jupiterは多くの暗示的なビルトイン型変換を提供しています。変換プロセスは、各メソッドパラメータの宣言された型に依存します。

例えば、もし`@ParameterizedTest`が`TimeUnit`型のパラメータを宣言していて、ソースから供給された実際の型が`String`であった場合、`String`は自動的に対応する`TimeUnit`enum定数に変換されます。

```java
@ParameterizedTest
@ValueSource(strings = "SECONDS")
void testWithImplicitArgumentConversion(TimeUnit argument) {
    assertNotNull(argument.name());
}
```

`String`インスタンスは、現在次のターゲット型に暗示的に変換されます。

|ターゲット型|例|
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

#### String-to-Object変換のフォールバック
Stringから上のリストで挙げたターゲット型への暗示的な変換に加えて、JUnit Jupiterでは、`String`をある型への自動変換に対するフォールバック機能を提供します。それは目標とする、ターゲット型が、下の定義にまさに正確に合致した*ファクトリーメソッド*か*ファクトリーコンストラクタ*を宣言するときです。

- *ファクトリーメソッド*：ターゲットタイプの中で宣言されているnon-private、かつ`static`なメソッドで、1つの`String`引数を取り、ターゲット型のインスタンスを返すもの。メソッド名は任意であり、いかなる慣習にも従う必要はありません。
- *ファクトリーコンストラクタ*：ターゲット型のnon-privateなコンストラクタで、1つの`String`引数を取るもの。

> ℹ️ もし複数の*ファクトリーメソッド*が見つかった場合、それらは無視されます。もし*ファクトリーメソッド*と*ファクトリーコンストラクタ*が見つかった場合、ファクトリーメソッドが、コンストラクタの代わりに使われます。

例えば、次の`@ParameterizedTest`メソッドの中で、引数`Book`は`Book.fromTitle(String)`ファクトリーメソッドが呼び出されることで生成され、`"42 Cats"`が本のタイトルとして渡されます。

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
暗黙的な引数変換の代わりに、次の例のように`@ConvertWith`を使うことで、あるパラメータに対して明示的に`ArgumentConverter`を特定することができます。

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

明示的な引数変換は、テストと拡張の著者らによって実装される必要があります。そのため、`junit-jupiter-params`では、レファレンス実装として使える明示的な引数変換器：`JavaTimeArgumentConverter`を提供しています。結合アノテーション`JavaTimeConversionPattern`アノテーションを通して使うことができます。

```java
@ParameterizedTest
@ValueSource(strings = { "01.01.2017", "31.12.2017" })
void testWithExplicitJavaTimeConverter(
        @JavaTimeConversionPattern("dd.MM.yyyy") LocalDate argument) {

    assertEquals(2017, argument.getYear());
}
```

### 3.15.3. 引数集約
デフォルトでは、`@ParameterizedTest`メソッドに渡される各`引数`は、1つのメソッドパラメータに対応しています。その結果として、大量の引数を供給することが期待される引数ソースは、巨大なメソッドシグネチャーを導くことがあり得ます。

そのような場合、[`ArgumentAccessor`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/params/aggregator/ArgumentsAccessor.html)は、複数のパラメータの代わりに使うことができます。このAPIを使うことで、テストメソッドに渡された1つのパラメータを通して提供された引数にアクセスすることができます。さらに、[`暗黙的な変換`]()で述べた型変換もサポートしています。

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

`ArgumentsAccessor`*のインスタンスは、自動的に全ての*`ArgumentsAccessor`*型のパラメータに注入されます*。

#### カスタム集約
`ArgumentsAccessor`を用いた`@ParameterizedTest`メソッドの引数への直接アクセスとは別に、JUnit Jupiterはカスタムで再利用可能な*集約器*の使用もサポートしています。

カスタム集約器を使うためには、単に[`ArgumentAggregator`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/params/aggregator/ArgumentsAggregator.html)インターフェイスを実装し、`@ParameterizedTest`メソッド内で互換可能なパラメータに対して、`@AggregateWith`を付与して登録するだけです。集約の結果は、パラメータ化テストが呼び出された時に、対応するパラメータへの引数として提供されます。

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

もし複数のパラメータ化テストに対して繰り返し`@AggregateWith(MyTypeAggregator.class)`を宣言している場合、`@AggregateWith(MyTypeAggregator.class)`のメタアノテーションとして`@CsvToMyType`のようなカスタム*結合アノテーション*を作ることができます。次の例は、カスタム`@CsvToPerson`アノテーションを用いた例を示しています。

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
デフォルトでは、パラメータ化テスト呼び出しの表示名は、呼び出しインデックスと呼び出しに対する全ての引数の`String`表現を含んでいます。しかしながら、次の例のように`@ParameterizedTest`アノテーションの`name`属性によって呼び出し表示名をカスタマイズすることができます。

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

`@ParameterizedTest`メソッドと合わせて`ParameterResolver`を使うことができます。しかしながら、引数ソースによって解決されたパラメータは、引数リストの最初に来る必要があります。テストクラスは様々なパラメータリストを持つパラメータ化テストと同様に通常のテストクラスを含んでいることもあるので、引数ソースからの値は、`@BeforeEach`といったライフサイクル・メソッドやテストクラスコンストラクタは解決されません。

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
[`@TestTemplate`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/TestTemplate.html)メソッドは、通常のテストケースではなく、むしろテストケースのためのテンプレートです。したがって、`@TestTemplate`は、登録された提供器によって返される呼び出し文脈の数に応じて複数回呼び出されるものとして設計されています。そのため、登録された[`TestTemplateInvocationContextProvider `](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/api/extension/TestTemplateInvocationContextProvider.html)拡張と併せて使われる必要があります。テストテンプレートメソッドの各呼び出しは、通常の`@Test`メソッドの実行と同じように振る舞い、全く同じライフサイクル・コールバックと拡張が完全にサポートされています。用法例については、[テストテンプレートに対する呼び出し文脈の提供]()を参照ください。

## 3.17. 動的テスト
[アノテーション]()で説明したJUnit Jupiterの`@Test`アノテーションは、JUnit 4の`@Test`アノテーションに非常に似通っています。どちらもテストケースを実装したメソッドを描写します。これらのテストケースはコンパイル時に完全に決定するという意味では静的であり、それらの振る舞いは実行時に変更することはできません。*アサンプションは、意図的にかなり表現性に制限のあるものですが、動的な振る舞いの基本的な形式を提供します*。

これらの標準的なテストに加えて、全く新しい種類のテストプログラミング・モデルがJUnit Jupiterでは導入されました。この新しいテストとは、*動的テスト*です。動的テストは、`@TestFactory`が付与されたファクトリーメソッドによって、実行時に生成されます。

`@Test`メソッドとは対照的に、`@TestFactory`メソッド自身はテストケースではなく、むしろテストケースのためのファクトリーです。そのため、動的テストはファクトリーの産出物となります。技術的なことを言うと、`@TestFactory`メソッドは、`DynamicNode`インスタンスの`Stream`、`Collection`、`Iterable`、もしくは`Iterator`を返さなければなりません。`DynamicNode`のインスタンス化可能なサブクラスは`DynamicContainer`と`DynamicTest`です。`DynamicContainer`インスタンスは、*表示名*と動的子ノードリストで構成されており、動的ノードの任意なネスト階層を生成します。`DynamicTest`インスタンスは、遅れて実行され、テストケースを動的で非決定的に生成します。

`@TestFactory`によって返される`Stream`はどれも、`stream.close()`を呼ぶことによって適切に閉じられます。これによって、`Files.lines()`のような資源を安全に使うことができます。

`@Test`メソッドと同様に、`@TestFactory`メソッドも`private`や`static`でいることは**できず**、任意に`ParameterResolvers`で解決されるであろうパラメータを宣言することができます。

`DynamicTest`は実行時に生成されるテストケースで、*表示名*`と`Executable`で構成されています。`Executable`は`@FunctionalInterface`で、このインターフェイスは、*ラムダ表現*と*メソッド参照*として提供されることができる動的テストの実装であることを意味します。

> ⚠️  *動的テスト・ライフサイクル*：動的テストの実行ライフサイクルは、通常の`@Test`ケースとは全く異なります。特に、各動的テストに対してのライフサイクル・コールバックはありません。このことは、`@BeforeEach`と`@AfterEach`、それに対応した拡張コールバックは`@TestFactory`メソッドに対して実行され、各*動的テスト*には実行されないことを意味します。つまり、動的テストに対してラムダ表現でテストインスタンスからフィールドにアクセスしても、それらのフィールドは、同じ`@TestFactory`メソッドで生成された動的テストの実行中は、コールバックメソッドやその拡張によってリセットされません。

JUnit Jupiter 5.2.0に関しては、動的テストは常にファクトリーメソッドによって生成される必要があります；しかしながら、これは後のリリースの登録機能によって補完されるかもしれません。

> ⚠️  動的テストは現在*実験的な*機能です。詳細に関しては、[実験的なAPIs]()をご覧ください。

### 3.17.1. 動的テストの例
次の`DynamicTestsDemo`クラスは、テストファクトリーと動的テストのいくつかの例を示しています。

最初のメソッドは不正な型を返しています。不正な返却型はコンパイル時に検出することができないため、実行時に検出され`JUnitException`が投げられます。

次の5つもメソッドは、`DynamicTest`インスタンスの`Collection`、`Iterable`、`Iterator`、`Stream`を生成する非常に単純な例です。これらの例のほとんどは、実際に動的振る舞いを示しておらず、単に原則的にサポートされている返却型を示しています。しかしながら、`dynamicTestsFromStream()`と`dynamicTestsFromIntStream()`は、与えられた`String`のセットと入力された数の範囲に対する動的テストの生成が、いかに簡単かを示しています。

次のメソッドは、性質上、真に動的なものです。`generateRandomNumberOfTests()`は、ランダム数を生成する`Iterator`と表示名生成器、テスト実行器を実装しており、その3つを`DynamicTest.stream()`に提供しています。`generateRandomNumberOfTests()`の非決定的な振る舞いは、もちろんテスト反復可能性に抵触しており、注意深く取り扱われるべきではありますが、動的テストの表現性と力を示しています。

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
