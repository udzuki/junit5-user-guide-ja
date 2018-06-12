# 5.2.0
**リリース日**：2018年4月29日

**スコープ**：JUnit BOM、Java 9とJava 10を用いたビルドを可能とするMaven Surefire 2.21.0へのサポート、パラメータ化テストにおける引数に関する*引数集約*と*拡大プリミティブ変換*、`@MethodSource`に関する外部のファクトリメソッド、ならびに様々なマイナーな改善とバグ修正。

このリリースに関する全ての*解決済みの*イシューとプル・リクエストの完全なリストに関しては、GitHub上のJUnitレポジトリにあるマイルストーン[`5.2 M1`](https://github.com/junit-team/junit5/milestone/22?closed=1)と[`5.2 RC1`](https://github.com/junit-team/junit5/milestone/25?closed=1)、[`5.2 GA`](https://github.com/junit-team/junit5/milestone/26?closed=1)をご覧ください。

## 全体的な改善
- JUnit BOM：[Maven](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Importing_Dependencies)または[Gradle](https://docs.gradle.org/current/userguide/managing_transitive_dependencies.html#sec:bom_import)を用いた依存関係の管理を容易にするために、*部品表*であるPOMが`org.junit:junit-bom:5.2.0`のMaven座標以下に提供されています。

## JUnit Platform
### バグ修正
- スペースを含むタグ表現が、JUnit Platform Maven Surefireプロバイダによってサポートされています。
- `ConsoleLauncher`に対して重複して供給される`--config`キーが適切にレポートされています。
- （`HierarchicalTestEngine`基盤における）`Node.after()`内で投げられる例外は、もはやそれより先に投げられた例外を隠蔽しません。

### 新機能と改善
- JUnit Platform Surefireプロバイダ（`junit-platform-surefire-provider`）は、Java 9とJava 10が使えるSurefire `2.21.0`と作動ならびに必要とします。
- クラス名でのフィルタリングのためのデフォルトの*内包パターン*は、`Test`で始まるか、`Test`または`Tests`で終わるものとマッチします。
  - このパターンは、`ConsoleLauncher`とJUnit Platform Gradleプラグイン、`JUnitPlatform`ランナーによって使われます。

## JUnit Jupiter
### バグ修正
- `@TestInstance(PER_CLASS)`ライフサイクル・モードを用いる際、`AfterAllCallback`によって投げられる例外はもはやクラスレベルで投げられる例外を隠蔽しません。

### 新機能と改善
- `Assertions`において`assertDoesNotThrow()`メソッドを導入：入力されたコードブロックを実行しても、いかなる種類の例外も*投げない*ことをアサートします。
- `Assertions`において`fail()`メソッドを導入：明示的な失敗メッセージがなくても、テストを失敗させることができます。
- `@ParameterizedTest`に供給される引数に対する*拡大プリミティブ変換*のための暗示的なサポートが導入されました。
  - 例えば、`@ValueSource(ints = { 1, 2, 3 })`が付与されたパラメータ化テストは、`int`や`long`、`float`、`double`型の引数を許容するよう宣言されます。
- `@MethodSource`は、*完全修飾クラス名*によって参照された外部クラスに宣言されている`static`なファクトリメソッドをサポートしています。
- 複数の`@ParameterizedTest`引数を1つのオブジェクトに集約することをサポートしています。
  - 詳細については、[引数の集約]()をご覧ください。

## JUnit Vintage
変更はありません。
