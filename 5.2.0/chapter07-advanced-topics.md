# 7. 先進的なトピック
## 7.1. JUnit PlatformラウンチャーAPI
JUnit 5の主要な目的の一つは、JUnitとそのプログラム的なクライアントービルドツールとIDEー間のインターフェイスをより強固で安定したものにすることです。その目的は、テスト発見と実行の内部を、外界から必要とされる全てのフィルタリングと設定から切り離すことです。

JUnit 5は、テスト発見・フィルタリング・実行に使うことのできる`Launcher`のコンセプトを導入します。さらに、サードパーティによるテストライブラリーSpockやCucumber、FitNesseのようなものですーは、カスタム[`TestEngine`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/engine/TestEngine.html)を提供することによってJUnit Platformのラウンチング（launching）・インフラにプラグインすることができます。

ラウンチング（launching）APIは[`junit-platform-launcher`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/package-summary.html)モジュールにあります。

ラウンチング（launching）APIの使用例は、[`junit-platform-console`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/console/package-summary.html)プロジェクトの[`ConsoleLauncher`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/console/ConsoleLauncher.html)にあります。

### 7.1.1. テストの発見
*テストディレクトリ*をプラットフォーム自身専用の特徴として導入することは、（上手くいけば）IDEやビルドツールを、過去にテストクラスやテストメソッドを特定するのに経験しなければならなかった困難のほとんどから解放するでしょう。

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

（訳注：[原典](https://junit.org/junit5/docs/5.2.0/user-guide/#launcher-api-discovery)では分割されていますが、上と下のファイルは同一のファイルかと思われます。）

現在のところ、クラスとメソッド、またはあるパッケージ内の全てのクラスの選択、もしくはクラスパス内の全てのテストを探索することでさえ実現可能です。発見は、全ての参加するテストエンジンをまたがって行われます。

その結果として、`TestPlan`は、`LauncherDiscoveryRequest`にフィットする全てのエンジン、テストクラス、そしてテストメソッドに関する階層的（かつ、読み込み専用）な記述となります。クライアントは、木を横断し、ノードに関する詳細を集め、オリジナルソース（クラスやメソッド、ファイルの位置）へのリンクを手に入れることができます。テストプランの全てのノードは、*ユニークなID*を持っており、特定のテストやテストグループを呼び出すのに使うことができます。

### 7.1.2. テストの実行
テストを実行するために、クライアントは、発見フェイズと同じ`LauncherDiscoveryRequest`を使うか、新しいリクエストを生成することができます。テスト進行とレポートは、次の例のように`Launche`に1つかそれ以上の[`TestExecutionListener`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/TestExecutionListener.html)実装を登録することで達成できます。

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

### 7.1.3. 自身のテストエンジンをプラグインする
JUnitは現在のところ、2つの利用可能な[`TestEngine`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/engine/TestEngine.html)を提供しています。
- [`junit-jupiter-engine`](https://junit.org/junit5/docs/5.2.0/api/org/junit/jupiter/engine/package-summary.html)：JUnit Jupiterのコアです。
- [`junit-vintage-engine`](https://junit.org/junit5/docs/5.2.0/api/org/junit/vintage/engine/package-summary.html)：JUnit 4上の薄いレイヤーで、*ヴィンテージ*テストをラウンチャー・インフラで実行できるようにします。

サードパーティもまた、[junit-platform-engine](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/engine/package-summary.html)内のインターフェイスを実装して、そのエンジンを*登録する*ことで、彼ら自身の`TestEngine`に貢献するかもしれません。エンジンの登録は、現在のところ、Javaの`java.util.ServiceLoader`メカニズムによってサポートされています。例えば、`junit-jupiter-engine`モジュールは、`junit-jupiter-engine`JAR中の`/META-INF/services`内にある`org.junit.platform.engine.TestEngine`という名前のファイルの`org.junit.jupiter.engine.JupiterTestEngine`を登録します。

#### 7.1.4. 自身のテスト実行リスナーをプラグインする
テスト実行リスナーをプログラム的に登録するためのpublicな[`Launcher`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/Launcher.html)APIメソッドに加えて、実行時にJavaの`java.util.ServiceLoader`機能を通して発見されるカスタム[`TestExecutionListener`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/TestExecutionListener.html)実装が自動的に`DefaultLauncher`に登録されます。例えば、[`TestExecutionListener`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/launcher/TestExecutionListener.html)を実装し、`/META-INF/services/org.junit.platform.launcher.TestExecutionListener`ファイル内で宣言されている`example.TestInfoPrinter`クラスは、自動的に読み込まれ登録されます。
