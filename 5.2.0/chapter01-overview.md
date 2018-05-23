# 1. 概要
本文書の目的は、テストを書くプログラマや拡張機能の開発者（extension authors）、ビルドツールやIDEベンダといった駆動部分の開発者（engine authors）に包括的なレファレンス文書を提供することです。

このドキュメントは[PDF](https://junit.org/junit5/docs/5.2.0/user-guide/index.pdf)でも利用可能です。

> 💡 *翻訳* このドキュメントは簡体字中国語でも利用可能です。

## 1.1. JUnit 5とは？
これまでのバージョンのJUnitとは異なり、JUnit 5は3つのサブプロジェクトの様々なモジュールから構成されています。

***JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage***

**JUnit Platform**は、JVM上で[テスティングフレームワークを起動させる](#junit-platformラウンチャーapi)ための基礎として作動します。また、プラットフォーム上で走るテスティングフレームワークを開発するための[`TestEngine`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/engine/TestEngine.html) APIを定義しています。さらに、プラットフォーム上でどのような`TestEngine`でも実行するために、[JUnit 4ベースのRuunner](#junit-4を用いてjunit-platformを実行する)と同様に、コマンドラインからプラットフォームを立ち上たり、[Gradle](#gradle)や[Maven](#maven)のためのプラグインを構築するための[`コンソールラウンチャー`](#コンソールラウンチャー)を提供します。

**JUnit Jupiter**は、JUnit 5でテストを書いたり拡張するための新しい[プログラミングモデル](#テストを書く)と[拡張モデル](#拡張モデル)の組み合わせです。Jupiterのサブプロジェクトは、プラットフォーム上でJupiterベースのテストを実行するための`TestEngine`を提供します。

**JUnit Vintage**は、プラットフォーム上でJUnit3とJUnit4ベースのテストを実行するための`TestEngine`を提供します。

## 1.2. サポートしているJavaのバージョン
JUnit 5は実行時にJava 8(またはそれ以上)を必要とします。しかしながら、テストコードのコンパイルはそれよりも前のバージョンのJDKでも可能です。

## 1.3. ヘルプ
JUnit 5に関連した質問は[Stack Overflow](https://stackoverflow.com/questions/tagged/junit5) でするか、[Gitter](https://gitter.im/junit-team/junit5)で会話してください。
