# 2. インストール
最終的なリリースやマイルストーンのためのアーティファクトは、Maven Centralにデプロイされます。

スナップショット版はSonatypeの[スナップショットレポジトリ](https://oss.sonatype.org/content/repositories/snapshots)の[/org/junit](https://oss.sonatype.org/content/repositories/snapshots/org/junit/)以下にデプロイされます。

## 2.1. 依存関係のメタデータ
### 2.1.1 JUnit Platform
- Group ID: `org.junit.platrform`
- Version: `1.2.0`
- Articfact IDs:
    - `junit-platform-commons`：JUnitの内部共通ライブラリ・ユーティリティ。これらのユーティリティは、JUnitフレームワーク内部使用のみが意図されています。*外部パーティからのいかなる使用はサポートされていません。*使用は自身の責任で行ってください！
    - `junit-platform-console`：コンソールからJUnit Platform上でテストを発見し実行することをサポートします。詳細については[Console Launcher]()をご覧ください。
    - `junit-platform-console-standalone`：全ての依存関係が包含された実行可能なJARファイルがMaven Centralの[junit-platform-standalone](https://repo1.maven.org/maven2/org/junit/platform/junit-platform-console-standalone)以下のディレクトリで提供されています。詳細については[Console Launcher]()をご覧ください。
    - `junit-platform-engine`：テストエンジンのパブリックAPIです。詳細については[自分自身のテストエンジンをプラグインする]()をご覧ください。
    - `junit-platform-gradle-plugin`：[Gradle]()を用いたJUnit Platform上のテストの発見・実行をサポートします。
    - `junit-platform-launcher`：テストプランの設定・起動するためのパブリックAPIです。典型的にはIDEやビルドツールによって使われます。詳細については[JUnit Platform Launcher API]()をご覧ください。
    - `junit-platform-runner`：JUnit 4環境におけるJUnit Platform上でテスト・テストスイートを実行するためのランナーです。詳細については[JUnit Platformを実行するためにJUnit 4を使用する]()をご覧ください。
    - `junit-platform-suite-api`：JUnit Platform上でテストスイートを設定をするためのアノテーションです。[JUnitPlatform runner]()にサポートされています。また、サードパーティによる`TestEngine`実装にもサポートされている可能性があります。
    - `junit-platform-surefire-provider`：[Maven Surefire]()を用いたJUnit Platform上のテストの発見・実行をサポートします。

### 2.1.2. JUnit Jupiter
- Group ID: `org.junit.jupiter`
- Version: `5.2.0`
- Articfact IDs:
    - `junit-jupiter-api`: [テストを書いたり]()、[拡張する]()ためのJUnit Jupiter APIです。
    - `junit-jupiter-engine`: JUnit Jupiter テストエンジンの実装です。実行時のみ必要です。
    - `junit-jupiter-params`: JUnit Jupiterでの[パラメータ化テスト]()をサポートします。
    - `junit-jupiter-migrationsupport`: JUnit 4からJUnit Jupiterへのマイグレーションをサポートします。いくつかのJUnit 4 rulesを実行する時のみ必要です。

### 2.1.3. JUnit Vintage
- Group ID: `org.junit.vintage`
- Version: `5.2.0`
- Articfact ID:
    - `junit-vintage-engine`: JUnit Vintageテストエンジンの実装です。かつてのJUnitテスト（つまり、JUnit 3やJUnit 4形式で書かれたテスト）を新しいJUnit Platform上で実行できます。

### 2.1.4. 部品表
次のMaven座標以下で提供されている *部品表* POMは、[Maven](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Importing_Dependencies)や[Gradle](https://docs.gradle.org/current/userguide/managing_transitive_dependencies.html#sec:bom_import)を用いて複数の上記アーティファクトを参照する際に、依存関係管理を容易にするために利用できます。

- Group ID: `org.junit`
- Articfact ID: `junit-bom`
- Version: `5.2.0`

### 2.1.5. 依存関係
上記のアーティファクトは全て、次の *@API Guardian JAR*で公開されているMaven POMsに依存関係があります。
- Group ID: `org.apiguardian`
- Articfact ID: `apiguardian-api`
- Version: `1.0.0`

さらに、上記アーティファクトのほとんどは、次の *OpenTest4J* JARと直接的、または推移的な依存関係があります。

- Group ID: `org.opentest4j`
- Articfact ID: `opentest4j`
- Version: `1.0.0`

## 2.2. 依存関係図

![](https://junit.org/junit5/docs/5.2.0/user-guide/images/component-diagram.svg)

## 2.3. JUnit Jupiter サンプルプロジェクト
[junit5-samples](https://github.com/junit-team/junit5-samples)レポジトリは、JUnit JupiterとJUnit Vintageをベースとしたサンプルプロジェクト集をホストしています。下記プロジェクトには`build.gradle`と`pom.xml`がそれぞれ置かれています。

- Gradleに関しては、[junit5-jupiter-starter-gradle](https://github.com/junit-team/junit5-samples/tree/r5.2.0/junit5-jupiter-starter-gradle)を確認してください。
- Mavenに関しては、[junit5-jupiter-starter-maven](https://github.com/junit-team/junit5-samples/tree/r5.2.0/junit5-jupiter-starter-maven)を確認してください。
