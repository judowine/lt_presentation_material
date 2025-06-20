## Jetpack Composeにおける「副作用」との賢い付き合い方（5分LT）

皆さん、こんにちは！今日はJetpack Compose開発で避けて通れない**「副作用（Side Effect）」**について、その概念とどう付き合っていくべきかをお話しします。

---

### 1. 「テスト容易性」って何だろう？

まず、私たちがコードを書く上で非常に大切な「テスト容易性」について考えてみましょう。

テストとは、簡単に言うと、**「私たちが書いたコードが、思い描いた通りに動くかどうか」**を確かめる作業です。これをもう少し厳密に関数に対してスコープを絞ると、**「実装された関数が、仕様に書かれた関数と等しいことを証明する」**ことだと解釈できます。

では、「等しい」とはどういうことでしょうか？

#### 1.1. 数学でいうところの関数

数学ていうところの関数というやつがあります。
これは、**「同じ入力（Input）を与えたら、必ず同じ出力（Output）が返ってくること」**対応関係のことを言います。この性質を持つ対応関係は、入力と出力だけを見ればその振る舞いを予測できるため、２つの関数の等価性を証明するには、「関数に与える任意の入力に対して、その出力が一致する」ことを示せば、２つの関数が等価であることを示すことができます。

---

### 2. Composable関数は「UIを生成する関数」

この考え方をJetpack Composeの**Composable関数**に当てはめてみましょう。

Composable関数は、アプリの**「状態（State）」**という入力を受け取って、それに対応する**「UI」**という出力を生成します。まるで数学の関数が `f(x) = y` のように、入力 `x` から出力 `y` を生み出すのと同じです。

```kotlin
@Composable
fun Greeting(name: String) { // ここでの 'name' が入力（State）
    Text("Hello, $name!") // ここでUI（出力）を生成
}
```

この`Greeting`関数は、`name`という入力が変わらない限り、常に同じ「Hello, [name]!」というUIを生成します。このように、Composable関数は**「State（入力）に対してUI（出力）を生成する関数」**として捉えることができます。

もし、このComposable関数が「入力されたStateに対して、UIが一意に定まる」のであれば、テストは非常にシンプルになります。「このStateを入れたら、本当にこのUIが表示されるか？」という一点だけを集中して確認すれば良いのです。実はComposable関数として公開されているAPIはこの性質を持っていて、これこそがComposableの持つ高い**テスト容易性**なのです。

---

### 3. 副作用は「UIの一意性」を乱す存在

しかし、現実はそう単純ではありません。ここで**「副作用」**というものが登場します。

一般的なプログラミングにおける副作用とは、**「ある処理が、その本来の責務の範囲外で、外部の状態を変更してしまうこと」**を指します。

例えば、UIを表示するだけのComposable関数が、表示中にこっそりユーザーデータをサーバーに送信したらどうでしょう？ユーザーデータの送信は「UIを表示する」という本来の責務とは関係ありませんよね。

**Jetpack Composeにおける副作用**も同じです。コンポーザブル関数の本来の責務はUIの描画ですが、その実行中に**UI以外の外部の状態を変更する操作**が発生すると、それは副作用となります。例えば、以下のようなケースです。

* ネットワークからデータを取得する
* データベースにデータを保存する
* アニメーションを開始する
* ログを記録する
* 画面を遷移させる

このような副作用を持つComposable関数は、**引数に与えられたStateに対して、UIが一意に定まらない**という問題を引き起こします。なぜなら、その関数が実行されるたびに、UI以外の部分（例えばデータベースの内容）が変わってしまうからです。

Android Developersのドキュメントでも、「コンポーザブルは副作用なしであるべきです」と明確に述べられています。副作用があると、Composeの持つテスト容易性や予測可能性といったメリットが失われてしまうのです。

---

### 4. でも、副作用は悪ではない！

「副作用はよくない」と言われると、じゃあ一切使わない方がいいのか、と思ってしまうかもしれません。しかし、それは違います。**副作用は「悪」ではありません。むしろ、アプリケーションには不可欠な要素です。**

例えば、ユーザーがボタンを押したら画面が遷移したり、サーバーから最新の情報を取得したりすることは、ユーザー体験の向上には欠かせませんよね。これらはまさに副作用です。

重要なのは、**副作用を「適切に」「予測可能な方法で」扱うこと**です。正しく管理された副作用は、アプリの実装を容易にし、コードの可読性を高めます。特にAndroidアプリは、複数のライフサイクル（Activityの生成・破棄、画面の回転など）が存在するため、副作用をどう扱うかが非常に重要になります。

---

### 5. 副作用の適切な取り扱い方：「作用」API

では、副作用を適切に扱うにはどうすれば良いのでしょうか？

Jetpack Composeは、副作用を安全に管理するために、特別な**「作用（Effect）」API**を提供しています。公式ドキュメントでは「作用とは、UI を出力せず、コンポジションの完了時に副作用の実行を引き起こすコンポーズ可能な関数です」と説明されています。

つまり、UIを描画するComposableとは別に、**「このUIの状態になったら、この副作用を実行してね」**と、副作用を明示的に、そして制御された形で記述するための仕組みが「作用」APIなのです。

---

### 6. 副作用を取り扱う主要な「作用」API

Jetpack Composeには、さまざまな種類の副作用に対応するための多様な「作用」APIが存在します。これらの違いと使い所をこちらの表にまとめました。

| API             | 概要と目的                                                                     | 主な使い所                                                                                                                                                                                                        |
| :-------------- | :----------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`rememberCoroutineScope`** | コンポーザブルのライフサイクルに紐付いた `CoroutineScope` を取得する。 | ユーザーのボタンクリックなど、**UIイベントに応じて非同期処理を起動したい**場合。例: ユーザー操作後のデータ保存、スナックバーの表示など。                                                                                 |
| **`LaunchedEffect`** | コンポーザブルがコンポジションに入ったとき（またはキーが変更されたとき）にコルーチンを起動する。 | **特定のUIの状態変化（画面表示や引数の変更）に応じて非同期処理を開始したい**場合。例: 画面表示時のネットワークデータ取得、画面遷移時の初期データロードなど。                                                         |
| **`rememberUpdatedState`** | コルーチンや長く生きるオブジェクトが、最新の値を常に参照できるようにする。 | `LaunchedEffect`などのコルーチン内で、**再コンポジションによって変更される可能性のある最新の値を参照したい**場合。コルーチンを再起動せずに最新のコールバックや値をキャプチャするのに役立ちます。                         |
| **`DisposableEffect`** | コンポジションへの出入りに応じて、リソースのセットアップとクリーンアップを行う。 | **リソースのライフサイクル管理が必要な副作用**。例: オブザーバーの登録/解除、イベントリスナーの追加/削除、Androidのシステムサービスへの登録/解除など。                                                              |
| **`SideEffect`** | コンポジションが成功裏にコミットされた後、UIが実際に描画される前に実行される。 | **Composeの状態を、Composeの外にある非Composeの状態と同期させたい**場合。例: 分析ツールへのログ送信、Android Viewシステムとの連携、カスタムビューのプロパティ更新など。**状態変更や再コンポジションは行わない。** |
| **`NoWrapper`** | デバッグ用途のコンポーザブル。                                                   | 主にCompose内部の動作確認やデバッグに用いられる。通常のアプリケーション開発で直接使用することはほとんどない。                                                                                                         |
| **`produceState`** | 外部の非同期ソース（コルーチン、Flowなど）から得られる値をComposeのStateとして公開する。 | **非同期的に時間がかかって取得される値や、継続的に更新される外部のデータストリーム**をUIに表示したい場合。例: ネットワークAPIからのデータフェッチ、Flowからのデータ収集など。                                      |
| **`derivedStateOf`** | 複数のStateオブジェクトから新しいStateを導出する際に、不要な再計算を避ける。 | **複数のStateに基づいて計算される値があり、そのStateのいずれかが変更された場合にのみ再計算したい**場合。例: リストのフィルタリング結果、複雑なUIの表示/非表示条件など。                                              |
| **`snapshotFlow`** | ComposeのStateをKotlinの`Flow`に変換する。                                     | **ComposeのStateの変化を`Flow`として監視し、コルーチン内で処理したい**場合。例: 特定のUI状態の変更をトリガーとして、データベース保存や分析イベント送信を行いたい場合など。                                           |

---

### 7. まとめと結論

Jetpack Composeにおける「副作用」とは、UI描画以外の、アプリの外部状態に影響を与える操作のことです。Composable関数を「UIを生成する純粋な関数」として保つことで高いテスト容易性が得られますが、実際のアプリ開発では副作用が不可欠です。

そこでComposeは、本日紹介したような多様な「作用」APIを提供することで、これらの副作用を**安全かつ予測可能な方法で管理する**ことを可能にしています。

これらのAPIを適切に使いこなすことで、皆さんのComposeアプリケーションはより堅牢で、管理しやすく、そして何よりも高い品質を保ったUIを持つことができるでしょう。

ご清聴ありがとうございました！