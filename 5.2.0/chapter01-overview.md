# 1. 概要
このドキュメントの目的は、テストを書くプログラマや機能を拡張する開発者（extension authors）、駆動部分の開発者（engine authors）ならびにビルドツールやIDEのベンダーのために、包括的な参考資料を提供することです。

このドキュメントは[PDF](https://junit.org/junit5/docs/5.2.0/user-guide/index.pdf)でも利用可能です。

> 💡 *翻訳* このドキュメントは[簡体字中国語](http://sjyuan.cc/junit5/user-guide-cn/)でも利用可能です。

## 1.1. JUnit 5とは？
これまでのバージョンのJUnitとは異なり、JUnit 5は3つのサブプロジェクトで開発されている様々なモジュールから構成されています。

***JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage***

**JUnit Platform**は、JVM上で[テスティングフレームワークを起動させる](#junit-platformラウンチャーapi)ための基礎としての機能を果たします。また、プラットフォーム上で動作するテスティングフレームワークを開発するための[`TestEngine`](https://junit.org/junit5/docs/5.2.0/api/org/junit/platform/engine/TestEngine.html) APIを定義しています。さらに、プラットフォーム上であらゆる`TestEngine`を実行するために、[JUnit 4ベースのRuunner](#junit-4を用いてjunit-platformを実行する)と同様に、コマンドラインからプラットフォームを立ち上たり、[Gradle](#gradle)や[Maven](#maven)向けプラグインを構築するための[`コンソールラウンチャー`](#コンソールラウンチャー)を提供します。

**JUnit Jupiter**は、JUnit 5でテストを書いたり拡張するための新しい[プログラミングモデル](#テストを書く)と[拡張モデル](#拡張モデル)の組み合わせです。Jupiterのサブプロジェクトは、プラットフォーム上でJupiterベースのテストを実行するための`TestEngine`を提供します。

**JUnit Vintage**は、プラットフォーム上でJUnit3とJUnit4ベースのテストを実行するための`TestEngine`を提供します。

## 1.2. サポートしているJavaのバージョン
JUnit 5は実行時にJava 8(またはそれ以上)を必要とします。しかしながら、それよりも前のバージョンのJDKでコンパイルされたコードもテスト可能です。

## 1.3. ヘルプ
JUnit 5に関連した質問は[Stack Overflow](https://stackoverflow.com/questions/tagged/junit5)に投稿するか、[Gitter](https://gitter.im/junit-team/junit5)でチャットしてください。
