# 4. テストの実行
## 4.1. IDEサポート
### 4.1.1. IntelliJ IDEA
IntelliJ IDEAは、バージョン 2016.2から、JUnit Platform上でのテスト実行をサポートしています。詳細については、[IntelliJ IDEAブログの記事](https://blog.jetbrains.com/idea/2016/08/using-junit-5-in-intellij-idea/)をご覧ください。しかしながら、2017.3以降のバージョンのIDEAを使うことをお薦めします。というのも、これらの新しいバージョンのIDEAは、プロジェクト内で使われているAPIバージョンに応じて次のJARを自動でダウンロードしてくれるからです：`junit-platform-launcher`と`junit-jupiter-engine`、`junit-vintage-engine`。

> ⚠️ IDEA 2017.3より前のIntelliJ IDEAは、特定のバージョンのJUnit 5をバンドルします。そのため、新しいバージョンのJUnit JUpiterを使いたい場合、IDE内でのテスト実行はバージョンの衝突によって失敗する恐れがあります。そのような場合、ItelliJ IDEAにバンドルされたものではない新しいJUnit 5を使うために下の説明に従ってください。

異なるJUnit 5のバージョン（例えば、5.2.0）を使うには、対応するバージョンの`junit-platform-launcher`と`junit-jupiter-engine`、`junit-vintage-engine`のJARをクラスパスに含める必要があるかもしれません。

#### 追加のGradleの依存関係

```groovy
// Only needed to run tests in a version of IntelliJ IDEA that bundles older versions
testRuntime("org.junit.platform:junit-platform-launcher:1.2.0")
testRuntime("org.junit.jupiter:junit-jupiter-engine:5.2.0")
testRuntime("org.junit.vintage:junit-vintage-engine:5.2.0")
```

#### 追加のMavenの依存関係

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

EclipseでJUnit 5を使うためのさらなる情報については、[Eclipse Project Oxygen.1a (4.7.1a) - New and Noteworthy](https://www.eclipse.org/eclipse/news/4.7.1a/#junit-5-support)記事の公式な*Eclipse support for JUnit 5*の章をご覧ください。

### 4.1.3. 他のIDE
この記事を書いている時点では、IDEAとEclipseを除き、IDE内でJUnit Platform上のテスト実行を直接サポートしているIDEはありません。しかしながら、いまいまでも自身のIDEでJUnit 5を試すことができるように、JUnitチームは2つの仲介的な解決法を提供しています。それは[コンソール・ラウンチャー]()を手動で使うか、[JUnit 4ベースのランナー]()でテストを実行することができます。

## 4.2. ビルドサポート
### 4.2.1. Gradle
[バージョン4.6](https://docs.gradle.org/4.6/release-notes.html)から、GradleはJUnit Platform上でのテスト実行を[ネイティブ・サポート](https://docs.gradle.org/current/userguide/java_testing.html#using_junit5)しています。それを有効化するためには、`build.gradle`内の`test`タスク宣言で`useJUnitPlatform()`を指定するだけです。

```gradle
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

> ⚠️ *JUnit Platform Gradleプラグインは非推奨*：JUnitチームによって開発された非常に初歩的な`junit-platform-gradle-plugin`は、JUnit Platform 1.2で非推奨になり、1.3では廃止されます。Gradleの標準的な`test`タスクに移行してください。

#### 設定パラメータ
標準的なGralde `test`タスクは現在、テスト発見や実行に役立つJUnit Platform設定パラメータを設定するための専用DSLを提供していません。しかしながら、（下に示すように）システムプロパティや`junit-platform.properties`を通して、ビルドスクリプト内で設定パラメータを提供できます。

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
JUnitは、*JUL*で知られている`java.util.logging`パッケージ内のJava Logging APIsを使って、警告とデバッグ情報を排出しています。設定オプションについては、[`LogManager`](https://docs.oracle.com/javase/8/docs/api/java/util/logging/LogManager.html)の公式ドキュメントを参照してください。

その他に、ログメッセージを[Log4J](https://logging.apache.org/log4j/2.x/)や[Logback](https://logback.qos.ch/)といった他のロギングフレームワークにログメッセージをリダイレクトすることもできます。[`LogManager`](https://docs.oracle.com/javase/8/docs/api/java/util/logging/LogManager.html)のカスタム実装を提供しているロギングフレームワークを使うには、`java.util.logging.manager`システムプロパティに、使う[`LogManager`](https://docs.oracle.com/javase/8/docs/api/java/util/logging/LogManager.html)実装の*完全修飾クラス名*を設定してください。下の例では、Log4j 2.xの設定方法を示しています（詳細については、[Log4J JDK ロギングアダプタ](https://logging.apache.org/log4j/2.x/log4j-jul/index.html)をご覧ください）。

```Gradle
test {
    systemProperty 'java.util.logging.manager', 'org.apache.logging.log4j.jul.LogManager'
}
```

他のロギングフレームワークは、`java.util.logging`を使ってログされたメッセージをリダイレクトするための他の方法を提供しています。例えば、[Logback](https://logback.qos.ch/)では、実行時のクラスパスに依存関係を追加することで、[JULからSLF4Jへの橋渡し](https://www.slf4j.org/legacy.html#jul-to-slf4j)を使うことができます。

### 4.2.2. Maven
JUnitチームは、`mvn test`を通してJUnit 4とJUnit Jupiterのテストを実行できるようなMaven Surefireのための基本的なプロバイダを開発しました。[junit5-jupiter-starter-maven](https://github.com/junit-team/junit5-samples/tree/r5.2.0/junit5-jupiter-starter-maven)プロジェクト内の`pom.xml`ファイルは、開始ポイントとしてのプロバイダの利用方法を示しています。

> ℹ️ `unit-platform-surefire-provider` を用いて Maven Surefire 2.21.0 を使ってください。

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

JUnit Jupiterベースのテストへのサポートを設定するには、次のように、JUnit Jupiter API上に`test`の依存関係を設定し、`maven-surefire-plugin`の依存関係にJUnit Jupiter `TestEngine`実装を追加してください。

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

JUnit Platform Surefire Providerは、次のように、JUnit 4上に`test`の依存関係を設定し、`maven-surefire-plugin`の依存関係にJUnit Vintage `TestEngine`実装を追加することで、JUnit 4ベースのテストを実行できます。

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

#### 単一のテストクラスを実行
JUnit Platrform Surefire Providerは、Maven Surefire Pluginによってサポートされている`test`JVMシステムプロパティをサポートしています。例えば、`org.example.MyTest`内のテストメソッドだけを実行するには、コマンドラインから `mvn -Dtest=org.example.MyTest test` を実行してください。さらなる詳細については、[Mavne Surefire Plugin](https://maven.apache.org/surefire/maven-surefire-plugin/examples/single-test.html)のドキュメントを参照してください。

#### テストクラス名によるフィルタリング
Maven Surefire Pluginは次のパターンにマッチする完全修飾名を持つテストクラスをスキャンします。

- `**/Test*.java`
- `**/*Test.java`
- `**/*Tests.java`
- `**/*TestCase.java`

さらに、全てのネストされたクラス（静的なメンバークラスも含む）はデフォルトでは除外されます。

しかしながら、このデフォルトの振る舞いは、`include`と`exclude`の規則を`pom.xml`明示的に設定にすることで、上書きできます。例えば、Maven Surefireに静的なメンバークラスを除外させないためには、除外規則を上書きします。

*Maven Surefireの除外規則の上書き*
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

詳細については、Maven Surefireの[テストの包含と除外](https://maven.apache.org/surefire/maven-surefire-plugin/examples/inclusion-exclusion.html)のドキュメントをご覧ください。

#### タグによるフィルタリング
次の設定プロパティを用いることで、タグまたは[タグ表現]()によってテストをフィルターすることができます。
- *タグ*または*タグ表現*を算入するには、`groups`または`includeTags`を使ってください。
- *タグ*または*タグ表現*を除外するには、`excludedGroups`または`excludeTags`を使ってください。

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
`configurationParameters`プロパティを宣言し、（下に示すように）Java `Properties` ファイル構文を使うか`junit-platform.properties`ファイルを通してキーバリューペアを提供することで、テスト探索と実行に影響を与えるJUnit Platform[設定パラメータ]()をセットすることができます。

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
[Ant](https://ant.apache.org/)のバージョン`1.10.3`から、新しい[`junitlauncher`](https://ant.apache.org/manual/Tasks/junitlauncher.html)タスクが導入され、JUnit Platform上でのテスト実行がネイティブサポートされました。`junitlauncher`タスクは単独で、JUnit Platformを起動して選択されたテストコレクションを渡します。JUnit Platformはその後、テストの探索と実行を登録されたテストエンジンに委譲します。

`junitlauncher`タスクは、[リソースコレクション](https://ant.apache.org/manual/Types/resources.html#collection)といったネイティブなAnt構造になるべく密接に連携しています。これにより、ユーザがテストエンジンに実行してほしいテストを選択する際、他多数のコアなAntタスクと比べて、一貫して自然な感覚を与えています。

> ℹ️ Ant 1.10.3にある`junitlauncher`タスクのバージョンは、JUnit Platformを起動するための基本的で最小のサポートを提供しています。追加的な機能強化（分離したJVMへのテスト分岐のサポートも含む）が次のAntのリリースで利用可能になる予定です。

[`junit5-jupiter-starter-ant`](https://github.com/junit-team/junit5-samples/tree/r5.2.0/junit5-jupiter-starter-ant)プロジェクト内の`build.xml`は、開始ポイントとしてタスクの利用方法を示しています。

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

`test`要素を使うことで、実行したいテストを1つ選択できます。`classpath`要素はを使うことで、JUnit Platformを起動するために使うクラスパスを指定できます。このクラスパスは、実行に加えるテストクラスを配置するためにも使われます。

次の例は、複数の位置からテストクラスを選ぶための`junitlauncher`タスクの設定方法を示しています。

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

上の例では、`testclasses`要素によって、異なる位置にある複数のテストクラスを選択しています。

用法と設定オプションのさらなる詳細については、[`junitlauncher`タスク](https://ant.apache.org/manual/Tasks/junitlauncher.html)に関するAntの公式ドキュメントをご覧ください。

## 4.3. コンソールラウンチャー
[`ConsoleLauncher`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/console/ConsoleLauncher.html)はコマンドラインJavaアプリケーションで、JUnit Platformをコンソールから起動できます。例えば、JUnit VintageとJUnit Jupiterテストを実行し、テストの実行結果をコンソールに出力します。

全ての依存関係が含まれた実行可能な`junit-platform-console-standalone-1.2.0.jar`は、Mavenセントラルレポジトリ内の[`junit-platform-console-standalone`](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone)ディレクトリ以下に公開されています。スタンドアローンな`ConsoleLauncher`は次のように[実行](https://docs.oracle.com/javase/tutorial/deployment/jar/run.html)できます。

`java -jar junit-platform-console-standalone-1.2.0.jar <[Options>`

これが出力の例です。

```
├─ JUnit Vintage
│  └─ example.JUnit4Tests
│     └─ standardJUnit4Test ✔
└─ JUnit Jupiter
   ├─ StandardTests
   │  ├─ succeedingTest() ✔
   │  └─ skippedTest() ↷ for demonstration purposes
   └─ A special test case
      ├─ Custom test name containing spaces ✔
      ├─ ╯°□°）╯ ✔
      └─ 😱 ✔

Test run finished after 64 ms
[         5 containers found      ]
[         0 containers skipped    ]
[         5 containers started    ]
[         0 containers aborted    ]
[         5 containers successful ]
[         0 containers failed     ]
[         6 tests found           ]
[         1 tests skipped         ]
[         5 tests started         ]
[         0 tests aborted         ]
[         5 tests successful      ]
[         0 tests failed          ]
```

> ℹ️ *終了コード* [`ConsoleLauncher`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/console/ConsoleLauncher.html)は、いずれかのコンテナまたはテスト失敗した場合はステータスコード`1`、そうでない場合はステータスコード`0`を返します。

### 4.3.1. オプション

```
オプション                                       説明
------                                         -----------
-h, --help                                     ヘルプ情報を表示します。
--disable-ansi-colors                          出力でANSIカラーを無効化します。(全ての
                                                 ターミナルではサポートされていません。）
--details <[none,summary,flat,tree,verbose]    テスト実行時の出力詳細モードを選択します。
  >                                              次のうち、どれか1つを使います: [none, summary, flat,
                                                 tree, verbose]. 'none'が選択された場合、
                                                 サマリーとテスト失敗のみが表示されます。 (default: tree)
--details-theme <[ascii,unicode]>              テスト実行時の出力詳細ツリーのテーマを選択します。
                                                 次のうち、どれか1つを使います: [ascii, unicode]
                                                 (default: unicode)
--class-path, --classpath, --cp <Path:         追加的なクラスパスエントリを提供します。 --
  path1:path2:...>                               例えば、テストエンジンとその依存関係を追加でき
                                                 ます。このオプションは繰り返し可能です。
--reports-dir <Path>                           特定のローカルディレクトリへのレポート出力を有効化
                                                 します。（ディレクトリが存在しない場合は生成されます。）
--scan-modules                                 実験的：テスト発見で用いる全ての解決済みモジュールを
                                                 スキャンします。
-o, --select-module <String: module name>      実験的：テスト発見で用いる1つのモジュールを
                                                 選択します。このオプションは繰り返し可能です。
--scan-class-path, --scan-classpath [Path:     クラスパスまたは明示的なクラスパスのルート上の
  path1:path2:...]                               全てのディレクトリをスキャンします。引数がない場合、
                                                 -cp（ディレクトリとJARファイル）を通して供給された
                                                 追加的なクラスパスエントリと同様にシステムクラスパス
                                                 のみがスキャンされます。
                                                 クラスパス上にない明示的なクラスパスのルートは、
                                                 静かに無視されます。このオプションは繰り返し可能です。
-u, --select-uri <URI>                         テスト発見のためのURIを選択します。
                                                 このオプションは繰り返し可能です。
-f, --select-file <String>                     テスト発見のためのファイルを選択します。
                                                 このオプションは繰り返し可能です。
-d, --select-directory <String>                テスト発見のためのディレクトリを選択します。
                                                 このオプションは繰り返し可能です。
-p, --select-package <String>                  テスト発見のためのパッケージを選択します。
                                                 このオプションは繰り返し可能です。
-c, --select-class <String>                    テスト発見のためのクラスを選択します。
                                                 このオプションは繰り返し可能です。
-m, --select-method <String>                   テスト発見のためのメソッドを選択します。
                                                 このオプションは繰り返し可能です。
-r, --select-resource <String>                 テスト発見のためのクラスパス・リソースを選択します。
                                                 このオプションは繰り返し可能です。
-n, --include-classname <String>               完全修飾名がマッチするクラスのみを内包するための
                                                 正規表現を提供します。不要なクラス読み込みを避ける
                                                 ために、デフォルトのパターンは、クラス名が"Test"で始まるか、
                                                 "Test"または"Tests"で終わるクラスのみを内包
                                                 します。このオプションが繰り返し使われる場合、全ての
                                                 パターンはORを使って結合されます。
                                                 (default: ^(Test.*|.+[.$]Test.*|.*Tests?)$)
-N, --exclude-classname <String>               完全修飾名がマッチするクラスのみを除外するための
                                                 正規表現を提供します。このオプションが繰り返し使われる
                                                 場合、全てのパターンはORを使って結合されます。
--include-package <String>                     テスト実行に含まれるパッケージを提供します。
                                                 このオプションは繰り返し可能です。
--exclude-package <String>                     テスト実行から除外するパッケージを提供します。
                                                 このオプションは繰り返し可能です。
-t, --include-tag <String>                     タグがマッチするテストのみを内包するための
                                                 タグまたはタグ表現を提供します。このオプションが繰り返し
                                                 使われる場合、全てのパターンはORを使って結合され
                                                 ます。
-T, --exclude-tag <String>                     タグがマッチするテストのみを除外するための
                                                 タグまたはタグ表現を提供します。このオプションが繰り返し
                                                 使われる場合、全てのパターンはORを使って結合され
                                                 ます。
-e, --include-engine <String>                  テスト実行で内包するエンジンのIDを提供します。
                                                 このオプションは繰り返し可能です。
-E, --exclude-engine <String>                  テスト実行から除外するエンジンのIDを提供します。
                                                 このオプションは繰り返し可能です。
--config <key=value>                           テスト発見・実行のための設定パラメータをセットします。
                                                 このオプションは繰り返し可能です。
```

## 4.4. JUnit 4を用いてJUnit Platformを実行
`JUnitPlatform`ランナーは、JUnit 4ベースの`Runner`であり、JUnit 4環境内のJUNit Platform上でサポートされているプログラミングモデルを持つテスト（例えば、JUnit Jupiterテストクラス）は全て実行できます。

`@RunWith(JUnitPlatform.class)`をクラスに付与することで、JUnit 4をサポートしているもののJUnit PlatformはまだサポートしていないIDEやビルドシステムであっても実行できます。

> ℹ️ JUnit PlatformはJUnit 4にはない特徴を持っているので、そのランナーはJUnit Platformのサブセットのみをサポートしています。特に、レポート機能についてが当てはまります（[表示名vs技術的な名称]()をご覧ください）。しかし、さしあたり`JUnitPlatform`ランナーはとっかかりやすい方法です。

### 4.4.1. セットアップ
次のアーティファクトとそれらの依存関係がクラスパス上に必要です。グループIDやアーティファクトID、バージョンに関する詳細は[依存関係のメタデータ]()をご覧ください。

#### 明示的な依存関係
- *test*スコープ内での`junit-platform-runner`: `JUnitPlatform`ランナーの位置
- *test*スコープ内での`junit-4.12.jar`: JUnit 4を用いたテスト実行
- *test*スコープ内での`junit-jupiter-api`: `@Test`などを含むJUnit Jupiterを用いてテストを書くためのAPI
- *test runtime*スコープ内での`junit-jupiter-engine`: JUnit Jupiterのための`TestEngine`実装

#### 推移的な依存関係
- *test*スコープ内での`junit-platform-suite-api`
- *test*スコープ内での`junit-platform-launcher`
- *test*スコープ内での`junit-platform-engine`
- *test*スコープ内での`junit-platform-commons`
- *test*スコープ内での`opentest4j`

### 4.4.2. 表示名vs技術的な名称
`@RunWith(JUnitPlatform.class)`を通して実行するクラスにカスタム*表示名*を定義するには、単に`@SuiteDisplayName`をクラスに付与してカスタム値を提供するだけです。

デフォルトでは、*表示名*はテストアーティファクトのために使われます。しかしながら、GradleやMavenといったビルドツールでテスト実行するために`JUnitPlatrform`ランナーが使われる場合、生成されたテストレポートはしばしば、テストクラス名や特殊文字を含むカスタム表示名のような短い表示名の代わりに、テストアーティファクトの*技術的な名称*（例えば、完全修飾クラス名）を含む必要があります。レポート目的のために技術的な名称を有効化するには、単に`@RunWith(JUnitPlatform.class)`と一緒に`@UseTechnicalNames`アノテーションを宣言するだけです。

`@UseTechnicalNames`の存在は、`@SuiteDisplayName`を通して設定されたいかなるカスタム表示名を上書きすることに注意してください。

### 4.4.3. 単一のテストクラス
`JUnitPlatform`ランナーを使う1つの方法は、テストクラスに直接`@RunWith(JUnitPlatform.class)`を付与することです。次の例にあるテストメソッドは`org.junit.Test`（JUnit Vintage）ではなく、`org.junit.jupiter.api.Test`（JUnit JUpiter）が付与されていることに注意してください。さらにこの場合では、テストクラスは`public`でないといけません。そうでないと、IDEやビルドツールがJUnit 4のテストクラスとして認識しない恐れがあります。

```java
import static org.junit.jupiter.api.Assertions.fail;

import org.junit.jupiter.api.Test;
import org.junit.platform.runner.JUnitPlatform;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
public class JUnit4ClassDemo {

    @Test
    void succeedingTest() {
        /* no-op */
    }

    @Test
    void failingTest() {
        fail("Failing for failing's sake.");
    }

}
```

### 4.4.4. テストスイート
複数のテストクラスがある場合、次の例のようにテストスイートを作成できます。

```java
import org.junit.platform.runner.JUnitPlatform;
import org.junit.platform.suite.api.SelectPackages;
import org.junit.platform.suite.api.SuiteDisplayName;
import org.junit.runner.RunWith;

@RunWith(JUnitPlatform.class)
@SuiteDisplayName("JUnit 4 Suite Demo")
@SelectPackages("example")
public class JUnit4SuiteDemo {
}
```

`JUnit4SuiteDemo`は、`example`パッケージとそのサブパッケージ内にある全てのテストを発見・実行します。デフォルトでは、`Test`で始まるか、`Test`または`Tests`で終わる名前を持つテストクラスのみが含まれます。

> ℹ️ *追加的な設定オプション* `@SelectPackages`だけでなく、テスト発見・フィルタリング用の多くの設定オプションがあります。詳細については、[Javadoc](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/suite/api/package-summary.html)をご覧ください。

## 4.5. 設定パラメータ
どのテストクラスとテストエンジンを内包するか、どのパッケージをスキャンするか、などをプラットフォームに知らせるのに加えて、特定のテストエンジンや登録した拡張を指定するための追加的なカスタム設定パラメータの提供が必要な場合があります。例えば、JUnit Jupiter `TestEngine`は次のユースケースのための*設定パラメータ*をサポートしています。

- [デフォルトのテストインスタンス・ライフサイクルを変更する]()
- [拡張自動検出を有効化する]()
- [条件を無効化する]()

*設定パラメータ*はテキストベースのキーバリューペアで、次の仕組みの内の1つを通してJUnit Platform上で実行しているテストエンジンに供給されます。

1. `configurationParameter()`と`configurationParameters()`メソッド。[`Launcher` API]()に供給するリクエストを構築する`LauncherDiscoveryRequestBuilder`内に実装されています。JUnit Platformによって提供されているツールの内の1つを通してテスト実行している場合、次のように、設定パラメータを指定できます。
 - [コンソール・ラウンチャー]()： `--config`コマンドラインオプションを利用。
 - [Gradle plugin]()： `configurationParameter`または`configurationParameters`のDSLを利用。
 - [Maven Surefire provider]()： `configurationParameters`プロパティを利用。
2. JVMシステムプロパティ。
3. JUnit Platform設定ファイル：`junit-platform.properties`という名前でクラスパスのルートに置かれたJavaの`Properties`ファイルの構文規則に従ったファイル。

> ℹ️ 設定パラメータは上で定義された順序で探索されます。結果として、`Launcher`に直接供給された設定パラメータは、システムプロパティと設定ファイルを通して供給されたパラメータよりも優先して使われます。同じように、システムプロパティを通して供給された設定パラメータは、設定ファイルを通して供給された設定パラメータよりも優先して使われます。

## 4.6. タグ表現
タグ表現は、ブーリアン（boolean）表現で、`!`と`&`、`|`オペレータが使われます。さらに、`(`と`)`もオペレータの優先順位を調節するために使われます。

*表1. オペレータ（優先順位の降順）*

|オペレータ|意味|結合性|
|:--:|:--:|:--:|
|!|not|右|
|&|and|左|
|&#124;|or|左|

いくつかの観点でテストにタグ付けしている場合、タグ表現は実行するテストの選択を助けてくれます。テストタイプ（例えば、*micro*や*integration*、*end-to-end*）や特徴（**foo**や**bar**、**baz**）によってタグ付けするときは、次のタグ表現が役に立つでしょう。

|タグ表現|選択|
|:--:|:--:|
|foo|**foo**の全てのテスト|
|bar &#124; baz|**bar**の全てのテストに加えて**baz**の全てのテスト|
|bar & baz|**bar**と**baz**相互に関わる全てのテスト|
|foo & !end-to-end|**foo**のうち、*end-to-end*でない全てのテスト|
|(micro &#124; integration) & (foo &#124; baz)|**foo**または**baz**のうち、*micro*または*integration*の全てのテスト|
