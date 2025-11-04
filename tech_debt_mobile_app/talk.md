## 今日書くコードが明日の技術負債になる - Compose開発における設計負債の予防（5分LT）

皆さん、こんにちは！今日は「今日書くコードが明日の技術負債になる」というテーマで、Jetpack Compose開発における設計負債の予防についてお話しします。

---

### はじめに：技術負債は「過去の遺産」ではない

技術負債と聞くと、「過去に誰かが書いた古いコード」「レガシーシステム」といったイメージを持つ方も多いかもしれません。しかし、**技術負債は今日、この瞬間にも生まれています**。

私たちが今日書くコードが、明日の技術負債になる可能性があるのです。だからこそ、重要なのは**予防的な設計**です。

今日は、Jetpack Compose開発において、どのように設計負債を予防していくかについて、実践的な視点からお話しします。

---

### 1. 技術負債とは何か？

#### 1.1. 技術負債の定義

技術負債という概念は、1992年にWard Cunninghamによって提唱されました。彼はこれを次のように定義しています：

**「短期的な解決策を選ぶことで生じる、将来の開発コスト」**

これは金融における借金の概念とよく似ています：
- **元本** = コードの問題そのもの
- **利子** = その問題を放置することで増え続けるメンテナンスコスト

借金と同じように、技術負債も放置すると利息が膨らんでいき、最終的には返済が困難になってしまいます。

#### 1.2. 技術負債の2つのカテゴリー

技術負債は大きく2つのカテゴリーに分類できます。

**1. 技術選択負債**

これは、プラットフォームやライブラリの選択に関する負債です：
- Android ViewからJetpack Composeへの移行
- RealmからRoom Databaseへの切り替え
- 依存ライブラリのバージョン更新

これらは、時間とリソースが必要で、すぐには解決できない場合が多いです。

**2. 設計負債**

一方、設計負債は、コードの書き方や構造に関する負債です：
- ハードコーディング
- 高い結合度
- 巨大なクラスや関数
- 責務の混在

**重要なのは、設計負債は今日から改善できる**ということです。プラットフォームの移行を待つ必要はありません。

今日は、この**設計負債**に焦点を当てます。

---

### 2. 問題のあるコード例と3つの視点

まず、実際によくある問題のあるコードを見てみましょう。

```kotlin
@Composable
fun UserListScreen(viewModel: UserListViewModel) {
    val users by viewModel.users.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()
    var showDialog by remember { mutableStateOf(false) }
    var selectedUser by remember { mutableStateOf<User?>(null) }

    LaunchedEffect(Unit) {
        viewModel.loadUsers()
    }

    Column {
        TopAppBar(title = { Text("ユーザー一覧") })

        if (isLoading) {
            CircularProgressIndicator()
        } else {
            LazyColumn {
                items(users) { user ->
                    Row(
                        modifier = Modifier.fillMaxWidth()
                            .clickable {
                                selectedUser = user
                                showDialog = true
                            }
                            .padding(16.dp)
                    ) {
                        AsyncImage(
                            model = user.avatarUrl,
                            contentDescription = null,
                            modifier = Modifier.size(48.dp).clip(CircleShape)
                        )
                        Spacer(modifier = Modifier.width(12.dp))
                        Column {
                            Text(user.name, style = MaterialTheme.typography.bodyLarge)
                            Text(user.email, style = MaterialTheme.typography.bodySmall)
                        }
                    }
                }
            }
        }

        if (showDialog && selectedUser != null) {
            AlertDialog(
                onDismissRequest = { showDialog = false },
                title = { Text("ユーザー詳細") },
                text = { Text("${selectedUser?.name}の詳細情報") },
                confirmButton = {
                    TextButton(onClick = { showDialog = false }) {
                        Text("OK")
                    }
                }
            )
        }
    }
}
```

一見、Composeらしく書かれているように見えますが、このコードには多くの問題が潜んでいます。

設計負債を見つけるために、**3つの視点**から分析してみましょう。

#### 2.1. 視点① Preview関数が書けるか？

最初の視点は、「このComposableのPreviewを書けるか？」です。

先ほどのコードにPreviewを書こうとすると：

```kotlin
@Preview
@Composable
fun UserListScreenPreview() {
    // ViewModelが必要...
    // FakeViewModelを作る？でも状態の組み合わせが多いと大変
    UserListScreen(viewModel = FakeUserListViewModel())
}
```

ViewModelに依存していると、Previewを書くのが複雑になります。

**これの何が問題なのでしょうか？**

- FakeViewModelを用意する必要がある
- 様々な状態（ローディング中、エラー、空リストなど）を確認するために複数のFakeを作る必要がある
- 状態の組み合わせが増えると管理が困難
- デザインの微調整のたびにFakeを修正する必要がある
- 結果として、Previewを書くのが面倒になり、ビルド＆実行に頼ることになる

Previewを簡単に書けないということは、そのコンポーネントが外部依存を多く持ちすぎていることを示す重要なシグナルです。

#### 2.2. 視点② 状態管理が見やすいか？

2つ目の視点は、「状態管理が明確か？」です。

先ほどのコードを見てみると：

```kotlin
@Composable
fun UserListScreen(viewModel: UserListViewModel) {
    // 画面固有の状態
    var showDialog by remember { mutableStateOf(false) }
    var selectedUser by remember { mutableStateOf<User?>(null) }

    // ViewModelの状態
    val users by viewModel.users.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()

    // 副作用
    LaunchedEffect(Unit) {
        viewModel.loadUsers()
    }

    // そしてUIの定義が数百行...
}
```

**問題点**：
- 状態管理とUI表示が混在している
- どこで何の状態を持っているか把握しづらい
- 副作用の発生タイミングが不明確
- テストが書きにくい

これは、以前の資料で紹介した「副作用との賢い付き合い方」で説明した問題と同じです。

#### 2.3. 視点③ 責務が分離されているか？

3つ目の視点は、「責務が適切に分離されているか？」です。

先ほどのコードでは、1つのComposableに以下のすべてが詰め込まれています：

- **状態管理**（ViewModel接続、ローカル状態）
- **副作用**（データ読み込み）
- **UI構築**（リスト、ダイアログ）
- **イベント処理**（クリック、ダイアログ表示）

この結果：
- **変更の影響範囲が不明確**：どこを変更すると何が壊れるか分からない
- **テストが困難**：UIテストでしかテストできない
- **再利用できない**：他の場所で使えない

---

### 3. 予防的な設計の実践

では、どうすれば設計負債を予防できるのでしょうか？4つの重要な原則を見ていきましょう。

#### 3.1. 原則① Statelessを意識する

最も重要な原則は、**コンポーネントをできる限りStateless（状態を持たない）にする**ことです。

**Before: Stateful（状態を持つ）**
```kotlin
@Composable
fun UserCard(user: User) {
    var expanded by remember { mutableStateOf(false) }
    // 状態を内部で管理している
}
```

**After: Stateless（状態を持たない）**
```kotlin
@Composable
fun UserCard(
    user: User,
    expanded: Boolean,
    onExpandChange: (Boolean) -> Unit
) {
    // 状態を持たず、パラメータで受け取る
}
```

**Statelessのメリット**：
- **再利用可能**：異なる文脈で使える
- **Previewが書ける**：依存がないので簡単
- **テストが容易**：入力と出力だけをテストすればよい
- **予測可能な動作**：同じ入力なら同じ結果

これは、以前の資料で説明した「ステート駆動と宣言的UI」の考え方そのものです。

#### 3.2. 原則② コンポーネントを分離する

次に、大きなComposableを小さなコンポーネントに分離します。

```kotlin
@Composable
fun UserListScreen(viewModel: UserListViewModel) {
    val users by viewModel.users.collectAsState()
    val isLoading by viewModel.isLoading.collectAsState()
    var selectedUser by remember { mutableStateOf<User?>(null) }

    // StatelessなContentに委譲
    UserListContent(
        users = users,
        isLoading = isLoading,
        onUserClick = { selectedUser = it }
    )

    // ダイアログも分離
    selectedUser?.let { user ->
        UserDetailDialog(
            user = user,
            onDismiss = { selectedUser = null }
        )
    }
}

@Composable
fun UserListContent(
    users: List<User>,
    isLoading: Boolean,
    onUserClick: (User) -> Unit
) {
    // Statelessな実装
}
```

この設計により：
- `UserListScreen`は状態管理とイベント処理に集中
- `UserListContent`は表示に集中（Stateless）
- それぞれの責務が明確

#### 3.3. 原則③ 小さなコンポーネントに分割

さらに、小さな単位に分割していきます。

```kotlin
@Composable
fun UserCard(user: User, onClick: () -> Unit) {
    Row(
        modifier = Modifier.fillMaxWidth()
            .clickable(onClick = onClick)
            .padding(16.dp)
    ) {
        UserAvatar(avatarUrl = user.avatarUrl)
        Spacer(modifier = Modifier.width(12.dp))
        UserInfo(name = user.name, email = user.email)
    }
}

@Composable
fun UserAvatar(avatarUrl: String) {
    AsyncImage(
        model = avatarUrl,
        contentDescription = null,
        modifier = Modifier.size(48.dp).clip(CircleShape)
    )
}

@Composable
fun UserInfo(name: String, email: String) {
    Column {
        Text(name, style = MaterialTheme.typography.bodyLarge)
        Text(email, style = MaterialTheme.typography.bodySmall)
    }
}
```

これにより、各コンポーネントの責務が明確になり、再利用性が高まります。

そして、**Previewも簡単に書けるようになります**：

```kotlin
@Preview
@Composable
fun UserCardPreview() {
    UserCard(
        user = User(
            name = "山田太郎",
            email = "yamada@example.com",
            avatarUrl = "https://..."
        ),
        onClick = {}
    )
}

@Preview
@Composable
fun UserListContentPreview() {
    UserListContent(
        users = listOf(
            User("山田太郎", "yamada@example.com", "..."),
            User("佐藤花子", "sato@example.com", "...")
        ),
        isLoading = false,
        onUserClick = {}
    )
}
```

これで、修正のたびにビルド＆実行する必要がなくなり、開発効率が大幅に向上します。

#### 3.4. 原則④ 副作用を適切に管理

副作用（ネットワーク通信、データベースアクセスなど）は、UI層から分離して管理します。

```kotlin
class UserViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(UserUiState())
    val uiState: StateFlow<UserUiState> = _uiState.asStateFlow()

    fun loadUsers() {
        viewModelScope.launch {
            _uiState.value = _uiState.value.copy(isLoading = true)
            val users = userRepository.fetchUsers()
            _uiState.value = UserUiState(
                users = users,
                isLoading = false
            )
        }
    }
}
```

**この設計の利点**：
- 副作用（ネットワーク通信）はViewModel/UseCaseで管理
- UIは副作用を持たない純粋な表示器
- ロジックはUIフレームワークから独立してテスト可能
- 単方向データフロー

これは、以前の資料「副作用との賢い付き合い方」で説明した原則を実践したものです。

---

### 4. 設計原則がもたらすメリット

これらの設計原則を実践することで、以下のメリットが得られます：

1. **修正箇所がすぐ見つかる**：Previewで即座に確認できる
2. **変更に強い**：影響範囲が明確で、安心して変更できる
3. **テストしやすい**：UIとロジックを独立してテストできる
4. **再利用性が高い**：コンポーネントが汎用的で、様々な場所で使える

そして最も重要なのは、**技術負債が溜まりにくい**ということです。

#### 4.1. 責務の分離とAtomic Design

今やったことの本質は、**責務の分離**です。

- **状態管理の責務** → 親コンポーネント／ViewModel
- **表示の責務** → 子コンポーネント（Stateless）

コンポーネントを階層的に考える方法論として、**Atomic Design**という考え方があります：

- **小さな単位**（Atoms）：UserAvatar、UserInfo
- **中くらいの単位**（Molecules）：UserCard
- **大きな単位**（Organisms）：UserListContent

このように、小さなコンポーネントから大きなコンポーネントを構成することで、コードが管理しやすくなり、設計負債が溜まりにくくなります。

---

### 5. まとめ

#### 技術負債は「過去の遺産」ではない

**今日書くコードが明日の負債になる**

私たちが今日書くコードの品質が、将来の開発効率を左右します。

#### 設計負債を予防する4つのポイント

1. **Previewを簡単に書ける設計にする**
   - Previewを書くのが複雑 = 依存が多すぎる

2. **Statelessを意識する**
   - 状態を持たないコンポーネントは再利用しやすい

3. **責務を分離する**
   - 1つのコンポーネントに1つの責務

4. **副作用を適切に管理する**
   - UIは純粋な表示器、副作用はViewModel/UseCaseへ

#### 明日から始められること

- 新しいComposableを書くとき、**Stateless**にできないか考える
- **Previewを先に書いてみる**（簡単に書けなければ設計を見直す）
- **100行を超えたら分割を検討**する
- **副作用はUI層から分離**する

小さな意識の積み重ねが、将来の開発効率を大きく変えます。

技術負債を恐れるのではなく、予防的な設計を実践することで、長期的に保守しやすく、変更に強いコードを書き続けることができます。

ご清聴ありがとうございました！
