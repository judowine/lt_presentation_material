はい、承知いたしました。先のLT原稿に、`SideEffect`の説明を追加します。

---

## Jetpack Composeにおける「副作用」との賢い付き合い方（5分LT）

皆さん、こんにちは！今日はJetpack Compose開発で避けて通れない「副作用」について、その概念とどう付き合っていくべきかをお話しします。

### 1. 「副作用」って何だろう？

まず、プログラミングにおける「副作用」とは何でしょうか？これは、**「本来の責務ではない処理が、ある操作の結果として発生する状態の変化」**を指します。

例を挙げましょう。例えば、商品の合計金額を計算する`calculateTotal()`という関数があったとします。その本来の責務は、合計金額を「計算して返す」ことです。しかし、もしこの関数が同時に、データベースに注文履歴を書き込んだり、ユーザーのポイントを更新したりしたらどうでしょう？これらは「合計金額を計算する」という責務の範囲外ですよね。これが「副作用」です。

このような副作用があると、コードは予測しにくく、テストしにくく、再利用もしにくくなります。

---

### 2. Jetpack Composeにおける「副作用」

では、Jetpack Composeにおける副作用とは何でしょうか？

Composeのコンポーザブル関数は、基本的に**UIを描画すること**がその責務です。理想的には、同じ入力が与えられれば常に同じUIが生成され、その関数が実行されてもUI以外の外部の状態は一切変化しない、**「純粋な関数」**であることが望ましいとされています。

しかし、実際のアプリケーションでは、UIの描画だけでなく、以下のようなUIの外部で発生する処理がどうしても必要になります。

* ネットワークからのデータ取得（APIコール）
* データベースへのアクセス
* ユーザーイベントに応じた画面遷移
* 分析ツールのログ送信
* Androidのシステムサービス連携（位置情報取得など）

これらはすべて、コンポーザブル関数の本来の責務である「UIの描画」の範囲外です。これらが**Jetpack Composeにおける「副作用」**と呼ばれます。

これらの副作用を適切に扱わないと、UIが意図しないタイミングで更新されたり、リソースリークが発生したりする可能性があります。

---

### 3. 副作用との賢い付き合い方：専用APIの活用

そこでJetpack Composeは、これらの副作用を**安全かつ予測可能な方法で処理する**ための専用APIを提供しています。主なものをいくつかご紹介しましょう。

#### 3-1. `LaunchedEffect`

これは、**「コンポーザブルが画面に表示されたら、このコルーチンを一度起動して！」**という指示を出すものです。例えば、画面が表示されたときにサーバーからデータを取得したい場合などに使います。キーを指定することで、そのキーが変更されたときにコルーチンを再起動することもできます。

```kotlin
@Composable
fun UserProfile(userId: String) {
    LaunchedEffect(userId) {
        // userIdが変わるたびにユーザーデータをフェッチ
        // 例: ApiService.fetchUser(userId)
    }
    // ... UI表示
}
```

#### 3-2. `rememberCoroutineScope`

これは、**「コンポーザブルのライフサイクルに紐付いたコルーチンスコープをちょうだい！」**という指示です。ユーザーがボタンを押したときなど、UIイベントに応じて非同期処理を起動したい場合に便利です。このスコープで起動したコルーチンは、コンポーザブルが画面から消えると自動的にキャンセルされます。

```kotlin
@Composable
fun SaveButton() {
    val scope = rememberCoroutineScope()
    Button(onClick = {
        scope.launch {
            // ボタンクリックでデータを保存 (非同期処理)
            // 例: dataRepository.saveData()
        }
    }) {
        Text("保存")
    }
}
```

#### 3-3. `SideEffect`

これは、**「UIが描画された直後に、この処理を実行してね！」**という指示です。Composeの状態を、Composeの外にある非Composeの状態（例えば、既存のAndroid Viewシステムや分析ツール）と同期させたい場合に利用します。**注意点として、ここでComposeの状態を変更したり、再コンポジションをトリガーするような処理は避けるべきです。**

```kotlin
@Composable
fun AnalyticsScreen(eventName: String) {
    SideEffect {
        // UIが描画される直前に分析イベントを送信
        // 例: AnalyticsService.logEvent(eventName)
    }
    Text("分析イベント: $eventName をログに記録しました")
}
```

#### 3-4. `DisposableEffect`

これは、**「画面に表示されたらこれをセットアップして、画面から消えたらこれをクリーンアップしてね！」**という指示です。例えば、位置情報リスナーの登録と解除、イベントリスナーの追加と削除など、リソースのライフサイクル管理が必要な場合に最適です。

```kotlin
@Composable
fun LocationTracker() {
    DisposableEffect(Unit) { // Unitは初回のみ実行の意
        // ロケーションリスナーを登録
        val listener = object : LocationListener { /* ... */ }
        // locationManager.requestLocationUpdates(...)

        onDispose {
            // コンポーザブルが消えたらリスナーを解除
            // locationManager.removeUpdates(...)
        }
    }
    // ... UI表示
}
```

#### 3-5. `produceState`

これは、**「外部の非同期ソース（APIコールやデータストリーム）から得られる値を、Composeの状態として公開して！」**という指示です。非同期で取得されるデータをUIに反映させたい場合に、シンプルに記述できます。

```kotlin
@Composable
fun UserDataLoader(userId: String) {
    val user by produceState<User?>(initialValue = null, userId) {
        // userIdが変わるたびにユーザーデータをフェッチし、stateとして公開
        // value = ApiService.fetchUser(userId)
    }
    // ... userの状態をUIに反映
}
```

---

### 4. まとめ

Jetpack Composeにおける「副作用」とは、UI描画以外の、アプリの外部状態に影響を与える操作のことです。そして、Composeはこれらの副作用を安全に管理するために、`LaunchedEffect`、`rememberCoroutineScope`、`SideEffect`、`DisposableEffect`、`produceState`といった強力な専用APIを提供しています。

これらのAPIを適切に使い分けることで、UIコンポーザブルをより純粋に保ち、コードの可読性、テスト容易性、保守性を格段に向上させることができます。

ご清聴ありがとうございました！