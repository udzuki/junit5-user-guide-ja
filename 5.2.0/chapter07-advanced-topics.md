# 7. 先進的なトピック
## 7.1. JUnit PlatformラウンチャーAPI
JUnit 5の主要な目的の一つは、JUnitとそのプログラム的なクライアント（ビルドツールとIDE）間のインターフェイスをより強固で安定したものにすることです。その目的は、テスト発見と実行の内部を、外界から必要とされる全てのフィルタリングと設定から切り離すことです。

JUnit 5は、テスト発見・フィルタリング・実行に使うことのできる`Launcher`のコンセプトを導入します。さらに、サードパーティによるテストライブラリ（SpockやCucumber、FitNesse）は、カスタム[`TestEngine`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/engine/TestEngine.html)を提供することによってJUnit Platformの起動基盤にプラグインできます。

起動APIは[`junit-platform-launcher`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/package-summary.html)モジュールにあります。

起動APIの使用例は、[`junit-platform-console`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/console/package-summary.html)プロジェクトの[`ConsoleLauncher`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/console/ConsoleLauncher.html)にあります。

### 7.1.1. テストの発見
*テストディレクトリ*をプラットフォーム自身専用の特徴として導入することは、過去にテストクラスやテストメソッドを特定するのに経験しなければならなかった困難のほとんどからIDEやビルドツールを（上手くいけば）解放するでしょう。

使用例：
```java
import static org.junit.platform.engine.discovery.ClassNameFilter.includeClassNamePatterns;
import static org.junit.platform.engine.discovery.DiscoverySelectors.selectClass;
import static org.junit.platform.engine.discovery.DiscoverySelectors.selectPackage;

import org.junit.platform.launcher.Launcher;
import org.junit.platform.launcher.LauncherDiscoveryRequest;
import org.junit.platform.launcher.TestExecutionListener;
import org.junit.platform.launcher.TestPlan;
import org.junit.platform.launcher.core.LauncherDiscoveryRequestBuilder;
import org.junit.platform.launcher.core.LauncherFactory;
import org.junit.platform.launcher.listeners.SummaryGeneratingListener;
```

```java
LauncherDiscoveryRequest request = LauncherDiscoveryRequestBuilder.request()
    .selectors(
        selectPackage("com.example.mytests"),
        selectClass(MyTestClass.class)
    )
    .filters(
        includeClassNamePatterns(".*Tests")
    )
    .build();

Launcher launcher = LauncherFactory.create();

TestPlan testPlan = launcher.discover(request);
```

（訳注：[原典](https://junit.org/junit5/docs/5.2.0/user-guide/#launcher-api-discovery)では分割されていますが、上2つのソースコード片は同一のファイルかと思われます。）

現在のところ、クラスとメソッド、あるパッケージ内の全てのクラスを選択することや、クラスパス内の全てのテストを探索することさえ可能です。発見は、全ての関係するテストエンジンで行われます。

結果として`TestPlan`は、`LauncherDiscoveryRequest`に適合する全てのエンジンやクラス、テストメソッドの階層的（かつ、読み込み専用）な記述となります。クライアントは、そのツリーを走査し、ノードに関する詳細を集め、元ソース（クラスやメソッド、ファイルの位置）へのリンクを入手できます。テストプランの全てのノードは、*ユニークなID*を持っており、特定のテストやテストグループを呼び出すのに使用できます。

### 7.1.2. テストの実行
テストを実行するために、クライアントは、発見フェイズと同じ`LauncherDiscoveryRequest`を使うか、新しいリクエストを生成できます。テスト進行とレポートは、次の例のように`Launche`に1つ以上の[`TestExecutionListener`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/TestExecutionListener.html)実装を登録することで達成できます。

```java
LauncherDiscoveryRequest request = LauncherDiscoveryRequestBuilder.request()
    .selectors(
        selectPackage("com.example.mytests"),
        selectClass(MyTestClass.class)
    )
    .filters(
        includeClassNamePatterns(".*Tests")
    )
    .build();

Launcher launcher = LauncherFactory.create();

// Register a listener of your choice
TestExecutionListener listener = new SummaryGeneratingListener();
launcher.registerTestExecutionListeners(listener);

launcher.execute(request);
```

`execute()`メソッドに返り値はありませんが、リスナーを用いることで、容易に自身のオブジェクト内で最終結果を集約することができます。例については、[`SummaryGeneratingListener`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/listeners/SummaryGeneratingListener.html)をご覧ください。

### 7.1.3. 自身のテストエンジンをプラグイン
JUnitは現在のところ、2つの利用可能な[`TestEngine`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/engine/TestEngine.html)を提供しています。
- [`junit-jupiter-engine`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/engine/package-summary.html)：JUnit Jupiterのコアです。
- [`junit-vintage-engine`](https://junit.org/junit5/docs/5.2.0/api/org/junit/vintage/engine/package-summary.html)：JUnit 4の上部に位置する薄いレイヤーで、*古い*テストを起動基盤で実行できるようにします。

サードパーティもまた、[junit-platform-engine](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/engine/package-summary.html)モジュール内のインターフェイスを実装してエンジンを*登録する*ことで、自身の`TestEngine`に貢献する可能性があります。エンジンの登録は、現在のところ、Javaの`java.util.ServiceLoader`機構によってサポートされています。例えば、`junit-jupiter-engine`モジュールは、`junit-jupiter-engine`JAR中の`/META-INF/services`内にある`org.junit.platform.engine.TestEngine`という名前のファイルの`org.junit.jupiter.engine.JupiterTestEngine`を登録します。

#### 7.1.4. 自身のテスト実行リスナーをプラグイン
テスト実行リスナーをプログラム的に登録するためのパブリックな[`Launcher`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/Launcher.html)APIメソッドに加えて、Javaの`java.util.ServiceLoader`機能を通して実行時に発見されるカスタム[`TestExecutionListener`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/TestExecutionListener.html)実装が自動的に`DefaultLauncher`を用いて登録されます。例えば、[`TestExecutionListener`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/TestExecutionListener.html)を実装し、`/META-INF/services/org.junit.platform.launcher.TestExecutionListener`ファイル内で宣言されている`example.TestInfoPrinter`クラスは、自動的に読み込まれ登録されます。
