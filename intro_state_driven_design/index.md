# テスタブルなアプリ開発のアーキテクチャ：ステート駆動と宣言的UI

## 1. はじめに：なぜ今、テスタブルなアプリ開発なのか？

- モバイルアプリ開発と、宣言的UI (Jetpack Compose / SwiftUI) の台頭
- **テスタビリティの重要性**：バグの早期発見、変更への強さ、開発効率向上

---

## 2. 宣言的UIとは何か？：UIは「ステート」に駆動される

- **命令的UI** (Android XML / UIKit)：UIの手順を記述 → 手動で更新
- **宣言的UI** (Compose / SwiftUI)：UIの**状態（あるべき姿）**を宣言 → ステート変更で自動再描画

#### StatefulCounter の例
- UIが自身でステートを持つシンプルさとその限界

---

## 3. テスタビリティを最大化する「ステートレスな宣言的UI」

### ステートフルな宣言的UIの課題

- UIコンポーネントがステートとロジックを両方持つ
- テストがUIテストに寄りがちで、遅く不安定

### ステートレスな宣言的UIの利点

- UIコンポーネントは「表示とイベント通知」に徹する純粋な表示器
- ステートはUIの**外部（UI State Holder / ViewModel）**で管理

#### シーケンス図で見る単方向データフロー

- **ステートフルなUI**：UI内部でイベント処理・ステート変更
- **ステートレスなUI**：UIがイベント通知 → UI State Holderがステート変更 → 新しいステートをUIに通知 → UIが再描画

#### テスタビリティの向上

- UIとステート管理ロジックを分離してテストできる

---

## 4. アプリの「ドメインステート」とビジネスロジックの分離

- UIのステートのその先へ：アプリの本質である「ドメインステート」（ビジネスロジック層のステート）
  - 例：ログイン状態、商品リスト、カートの中身など

### なぜドメインステートがテスタビリティの鍵なのか？

- UIから独立して純粋なビジネスロジックのテストが可能
- 複雑なルールも画面を見ずに、高速に検証できる

---

## 5. テスタビリティを損なう「副作用」の管理

### プログラミングにおける「副作用」とは？

- ネットワーク通信、DBアクセスなど

### 宣言的UIの落とし穴

- Composable / View内に直接副作用を記述してしまう問題点
  - テストの困難性、関心の分離の欠如、予期せぬ再実行

### 副作用の理想的な管理

- UIはイベントを通知するだけ
- ドメインステートの変更ロジックは副作用を持たない純粋な関数
- 副作用は特定の責任を持つ層（UseCase / Repository）に集約・実行
- 結果をドメインステートに反映し、UIを更新

---

## 6. まとめ：テスタブルなアプリ開発への道

- 宣言的UIの強力な仕組みを活かすには「ステート駆動」の理解が不可欠
- ステートレスな宣言的UIと、明確なステート・副作用管理のアーキテクチャが重要
- **ロジックとUIを分離し、それぞれを独立してテストできることで、堅牢で変更に強いアプリを実現する**
