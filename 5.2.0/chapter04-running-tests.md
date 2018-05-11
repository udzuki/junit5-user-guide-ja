# 4. テストを実行する
## 4.1. IDEサポート
### 4.1.1. IntelliJ IDEA
IntelliJ IDEAは、バージョン 2016.2から、JUnit Platform上でのテスト実行をサポートしています。詳細については、[IntelliJ IDEAブログの記事](https://blog.jetbrains.com/idea/2016/08/using-junit-5-in-intellij-idea/)をご覧ください。しかしながら、2017.3か、それよりも新しいIDEAを使うことをお薦めします。というのも、これらの新しいバージョンのIDEAは、プロジェクト内で使われているAPIバージョンに応じて次のJARを自動でダウンロードしてくれるからです：`junit-platform-launcher`、`junit-jupiter-engine`、そして`junit-vintage-engine`。

> ⚠️ IDEA 2017.3より前のIntelliJ IDEAは、特定のバージョンのJUnit 5をバンドルします。そのため、新しいバージョンのJUnit JUpiterを使いたい場合、IDE内でのテスト実行はバージョンの衝突によって失敗するかもしれません。そのような場合、ItelliJ IDEAにバンドルされたものではない新しいJUnit 5を使うために下の説明に従ってください。

異なるJUnit 5のバージョン（例えば、5.2.0）を使うには、対応するバージョンの`junit-platform-launcher`と`junit-jupiter-engine`、`junit-vintage-engine`のJARをクラスパスに含める必要があるかもしれません。

#### 付加的なGradle依存関係

```groovy
// Only needed to run tests in a version of IntelliJ IDEA that bundles older versions
testRuntime("org.junit.platform:junit-platform-launcher:1.2.0")
testRuntime("org.junit.jupiter:junit-jupiter-engine:5.2.0")
testRuntime("org.junit.vintage:junit-vintage-engine:5.2.0")
```

#### 付加的なMaven依存関係

```xml
<!-- Only needed to run tests in a version of IntelliJ IDEA that bundles older versions -->
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <version>1.2.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.2.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>5.2.0</version>
    <scope>test</scope>
</dependency>
```

### 4.1.2. Eclipse
Eclipse IDEは、Eclipse Oxygen.1a (4.7.1a)からJUnit Platformサポートを提供しています。

EclipseでJUnit 5を使うためのさらなる情報については、[Eclipse Project Oxygen.1a (4.7.1a) - New and Noteworthy](https://www.eclipse.org/eclipse/news/4.7.1a/#junit-5-support)の公式*Eclipse support for JUnit 5*のセクションをご覧ください。

### 4.1.3. 他のIDE
この記事を書いている時点では、IDE内でJUnit Platform上のテスト実行を直接サポートしているIDEは、IntelliJ IDEAとEclipse以外にありません。しかしながら、JUnitチームは2つの即座な解決法を提供しているため、今日にでもJUnitを試すことができます。[コンソール・ラウンチャー]()を手動で使うか、[JUnit 4ベースのランナー]()でテストを実行することができます。

## 4.2. ビルドサポート
### 4.2.1. Gradle
[バージョン4.6](https://docs.gradle.org/4.6/release-notes.html)から、GradleはJUnit Platform上でのテスト実行を[ネイティブ・サポート](https://docs.gradle.org/current/userguide/java_testing.html#using_junit5)しています。それを有効化するためには、`build.gradle`内の`test`タスク宣言で`useJUnitPlatform()`を指定する必要があります。

```Gradle
test {
    useJUnitPlatform()
}
```

タグやエンジンによるフィルタリングもサポートされています：

```gradle
test {
    useJUnitPlatform {
        includeTags 'fast', 'smoke & feature-a'
        // excludeTags 'slow', 'ci'
        includeEngines 'junit-jupiter'
        // excludeEngines 'junit-vintage'
    }
}
```

オプションの包括的なリストについては、[Gradleの公式ドキュメント](https://docs.gradle.org/current/userguide/java_plugin.html#sec:java_test)を参照してください。

> ⚠️ *JUnit Platform Gradleプラグインは非推奨*：JUnitチームによって開発された基本的な`junit-platform-gradle-plugin`は、JUnit Platform 1.2では非推奨で、1.3では打ち切られます。Gradleの標準的な`test`タスクに移行してください。

#### 設定パラメータ
標準的なGralde `test`タスクは現在、テスト発見や実行に役立つJUnit Platform設定パラメータを設定するための専用DSLを提供していません。しかしながら、（下に示すように）システム・プロパティや`junit-platform.properties`を通して、ビルドインスクリプト内で設定パラメータを提供することができます。

```Gradle
test {
    // ...
    systemProperty 'junit.jupiter.conditions.deactivate', '*'
    systemProperties = [
        'junit.jupiter.extensions.autodetection.enabled': 'true',
        'junit.jupiter.testinstance.lifecycle.default': 'per_class'
    ]
    // ...
}
```

#### テストエンジンの設定
テストを実行するには、`TestEngine`実装をクラスパス上に配置する必要があります。

JUnit Jupiterベースのテストへのサポートを設定するには、次のように、`testCompile`の依存関係にJUnit Jupiter APIと`testRuntime`の依存関係にJUnit Jupiter `TestEngine`実装を設定する必要があります。

```Gradle
dependencies {
    testCompile("org.junit.jupiter:junit-jupiter-api:5.2.0")
    testRuntime("org.junit.jupiter:junit-jupiter-engine:5.2.0")
}

```

JUnit Platformは、次のように、`testCompile`の依存関係にJUnit 4と`testRuntime`の依存関係にJUnit Vintage `TestEngine`実装を設定することで、JUnit 4ベースのテストを実行することができます。

```gradle
dependencies {
    testCompile("junit:junit:4.12")
    testRuntime("org.junit.vintage:junit-vintage-engine:5.2.0")
}
```

#### ログの設定（オプション）
JUnitは、*JUL*で知られている`java.util.logging`パッケージ内のJava  Logging APIsを使って、警告とデバッグ情報を排出しています。設定オプションについては、[`LogManager`](https://docs.oracle.com/javase/8/docs/api/java/util/logging/LogManager.html)の公式ドキュメントを参照してください。

その他に、ログメッセージを[Log4J](https://logging.apache.org/log4j/2.x/)や[Logback](https://logback.qos.ch/)といった他のロギング・フレームワークにログメッセージをリダイレクトすることもできます。[`LogManager`](https://docs.oracle.com/javase/8/docs/api/java/util/logging/LogManager.html)の独自な実装を提供しているロギング・フレームワークを使うには、`java.util.logging.manager`システム・プロパティに、使う[`LogManager`](https://docs.oracle.com/javase/8/docs/api/java/util/logging/LogManager.html)実装の完全修飾クラス名を設定してください。下の例では、Log4j 2.xを設定しています（詳細については、[Log4J JDK ロギング・アダプタ](https://logging.apache.org/log4j/2.x/log4j-jul/index.html)をご覧ください）。

```Gradle
test {
    systemProperty 'java.util.logging.manager', 'org.apache.logging.log4j.jul.LogManager'
}
```

他のロギング・フレームワークは、`java.util.logging`を使ってログされたメッセージをリダイレクトするための他の方法を提供しています。例えば、[Logback](https://logback.qos.ch/)では、実行時のクラスパスに依存関係を追加することで、[JUL to SL4J ブリッジ](https://www.slf4j.org/legacy.html#jul-to-slf4j)を使うことができます。

### 4.2.2. Maven
JUnitチームは、`mvn test`を通してJUnit 4とJUnit Jupiterのテストを実行できるようなMaven Surefireのための基本的なプロバイダを開発しました。[junit5-jupiter-starter-maven](https://github.com/junit-team/junit5-samples/tree/r5.2.0/junit5-jupiter-starter-maven)内の`pom.xml`ファイルは、どのようにプロバイダを使い、開始ポイントとして使うかを示しています。

> :information_source: Maven Surefire 2.21.0 と `unit-platform-surefire-provider` を使ってください。

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.21.0</version>
            <dependencies>
                <dependency>
                    <groupId>org.junit.platform</groupId>
                    <artifactId>junit-platform-surefire-provider</artifactId>
                    <version>1.2.0</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
...
```

#### テストエンジンの設定
Maven Surefireでテストを実行するには、`TestEngine`実装を実行時クラスパスに加える必要があります。

JUnit Jupiterベースのテストへのサポートを設定するには、次のように、`test`の依存関係にJUnit Jupiter APIと`maven-surefire-plugin`の依存関係にJUnit Jupiter `TestEngine`実装を設定する必要があります。

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.21.0</version>
            <dependencies>
                <dependency>
                    <groupId>org.junit.platform</groupId>
                    <artifactId>junit-platform-surefire-provider</artifactId>
                    <version>1.2.0</version>
                </dependency>
                <dependency>
                    <groupId>org.junit.jupiter</groupId>
                    <artifactId>junit-jupiter-engine</artifactId>
                    <version>5.2.0</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
...
<dependencies>
    ...
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.2.0</version>
        <scope>test</scope>
    </dependency>
</dependencies>
...
```

JUnit Platform Surefire Providerは、次のように、`test`の依存関係にJUnit 4と`maven-surefire-plugin`の依存関係にJUnit Vintage `TestEngine`実装を設定することで、JUnit 4ベースのテストを実行することができます。

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.21.0</version>
            <dependencies>
                <dependency>
                    <groupId>org.junit.platform</groupId>
                    <artifactId>junit-platform-surefire-provider</artifactId>
                    <version>1.2.0</version>
                </dependency>
                ...
                <dependency>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                    <version>5.2.0</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
...
<dependencies>
    ...
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
</dependencies>
...
```

#### 一つのテストクラスを実行する
JUnit Platrform Surefire Providerは、Maven Surefire Pluginによってサポートされている`test`JVMシステム・プロパティをサポートしています。例えば、`org.example.MyTest`内のテストメソッドを1つだけ実行するには、コマンドラインから `mvn -Dtest=org.example.MyTest test` を実行します。詳細については、[Mavne Surefire Plugin](https://maven.apache.org/surefire/maven-surefire-plugin/examples/single-test.html)のドキュメントを参照してください。

#### テストクラス名によるフィルタリング
Maven Surefire Pluginは次のパターンにマッチする完全修飾クラス名を持つテストクラスをスキャンします。

- `**/Test*.java`
- `**/*Test.java`
- `**/*Tests.java`
- `**/*TestCase.java`

さらに、全てのネストされたクラス（staticメンバーのクラスも含む）はデフォルトでは除外されます。

しかしながら、このデフォルトの振る舞いは、明示的に`include`と`exclude`の設定を`pom.xml`にすることで、上書きすることができます。例えば、Maven Surefireにstaticなメンバークラスを除外させないためには、除外ルールを上書きします。

*Maven Surefireの除外ルールを上書きする*
```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.21.0</version>
            <configuration>
                <excludes>
                    <exclude/>
                </excludes>
            </configuration>
            ...
        </plugin>
    </plugins>
</build>
...
```

詳細については、Maven Surefireの[テストの算入と除外](https://maven.apache.org/surefire/maven-surefire-plugin/examples/inclusion-exclusion.html)をご覧ください。

#### タグによるフィルタリング
次の設定プロパティを用いることで、タグか[タグ拡張]()によってテストをフィルターすることができます。
- *タグ*か*タグ拡張*を算入するには、`groups`か`includeTags`を使ってください。
- *タグ*か*タグ拡張*を除外するには、`excludedGroups`か`excludeTags`を使ってください。

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.21.0</version>
            <configuration>
                <properties>
                    <includeTags>acceptance | !feature-a</includeTags>
                    <excludeTags>integration, regression</excludeTags>
                </properties>
            </configuration>
            <dependencies>
                ...
            </dependencies>
        </plugin>
    </plugins>
</build>
...
```

#### 設定パラメータ
`configurationParameters`プロパティを宣言し、（下に示すように）Java `Properties` ファイル・シンタックスを使うか、`junit-platform.properties`を通して、キーバリュー・ペアを提供することで、テスト探索と実行に影響を与えるJUnit Platform[設定パラメータ]()をセットすることができます。

```xml
...
<build>
    <plugins>
        ...
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.21.0</version>
            <configuration>
                <properties>
                    <configurationParameters>
                        junit.jupiter.conditions.deactivate = *
                        junit.jupiter.extensions.autodetection.enabled = true
                        junit.jupiter.testinstance.lifecycle.default = per_class
                    </configurationParameters>
                </properties>
            </configuration>
            <dependencies>
                ...
            </dependencies>
        </plugin>
    </plugins>
</build>
...
```

### 4.2.3. Ant
[Ant](https://ant.apache.org/)のバージョン`1.10.3`から、新しい[`junitlauncher`](https://ant.apache.org/manual/Tasks/junitlauncher.html)タスクが導入され、JUnit Platform上でのテスト実行がネイティブ・サポートされました。`junitlauncher`タスクは、単にJUnit Platformを起動して選択されたテストを渡す責任を担っています。JUnit Platformはその後、テストの探索と実行を登録されたテストエンジンに委譲します。

`junitlauncher`タスクは[リソース・コレクション](https://ant.apache.org/manual/Types/resources.html#collection)のようなネイティブなAnt構造になるべく近い形で、ユーザがテストエンジンに実行してほしいテストを選べるようにしています。これによって、タスクに他のAntのコアなタスクと比べて一貫して自然な感じを与えています。

> :information_source: Ant 1.10.3にある`junitlauncher`タスクのバージョンは、JUnit Platformを開始するための基本的で最小のサポートを提供しています。追加的な機能強化（分離したJVMへのテスト分岐のサポートも含む）が次のAntのリリースで利用可能になる予定です。

[`junit5-jupiter-starter-ant`](https://github.com/junit-team/junit5-samples/tree/r5.2.0/junit5-jupiter-starter-ant)プロジェクト内の`build.xml`は、どのようにタスクを使い、開始ポイントとして使うかを示しています。

#### 基本的な使い方
次の例は、`junitlauncher`タスクに1つのテストクラス（つまり、`org.myapp.test.MyFirstJUnit5Test`）を選ぶための設定方法を示しています。

```xml
<path id="test.classpath">
    <!-- The location where you have your compiled classes -->
    <pathelement location="${build.classes.dir}" />
</path>

<!-- ... -->

<junitlauncher>
    <classpath refid="test.classpath" />
    <test name="org.myapp.test.MyFirstJUnit5Test" />
</junitlauncher>
```

`test`要素を使うことで、実行したいテストを1つ選択することができます。`classpath`要素はを使うことで、JUnit Platformを開始するために使うクラスパスを指定することができます。このクラスパスは、テストクラスを特定するのにも使われます。

次の例は、`junitlauncher`タスクに複数箇所からテストクラスを選ぶための設定方法を示しています。

```xml
<path id="test.classpath">
    <!-- The location where you have your compiled classes -->
    <pathelement location="${build.classes.dir}" />
</path>
....
<junitlauncher>
    <classpath refid="test.classpath" />
    <testclasses outputdir="${output.dir}">
        <fileset dir="${build.classes.dir}">
            <include name="org/example/**/demo/**/" />
        </fileset>
        <fileset dir="${some.other.dir}">
            <include name="org/myapp/**/" />
        </fileset>
    </testclasses>
</junitlauncher>
```

上の例で、`testclasses`要素によって、違う場所にある複数のテストクラスを選択しています。

用法と設定オプションの詳細については、[`junitlauncher`タスク](https://ant.apache.org/manual/Tasks/junitlauncher.html)に関するAntの公式ドキュメントをご覧ください。
