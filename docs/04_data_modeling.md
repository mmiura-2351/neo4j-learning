# データモデリング - ベストプラクティス

## 1. グラフモデリングの基本原則

### RDBとの思考の違い

| RDB | グラフDB |
|-----|----------|
| テーブル設計から始める | ユースケース（質問）から始める |
| 正規化を重視 | 接続性を重視 |
| JOINで関係を表現 | リレーションシップで直接接続 |
| 外部キーで参照 | ポインタで直接参照 |

### グラフモデリングの手順

1. **ユースケースを定義** - どんな質問に答えたいか？
2. **エンティティを特定** - ノードになるもの
3. **関係を特定** - リレーションシップになるもの
4. **プロパティを決定** - 必要な属性
5. **クエリで検証** - 想定したクエリが効率的に書けるか

---

## 2. ノード vs リレーションシップ vs プロパティ

### 判断基準

```
それ自体を検索対象にしたい？ → ノード
2つのものを繋ぐ？ → リレーションシップ
付加情報？ → プロパティ
```

### 例：ECサイトの注文

```
❌ 悪い例：注文をプロパティで表現
(User {orders: ["order1", "order2"]})

✅ 良い例：注文をノードで表現
(User)-[:PLACED]->(Order)-[:CONTAINS]->(Product)
```

### ノードにすべきもの

- 検索の起点になる
- 複数のリレーションシップを持つ
- 複数のプロパティを持つ
- 他のエンティティと複雑な関係がある

### リレーションシップにすべきもの

- 2つのノードを接続する
- アクション・動詞で表現できる（KNOWS, PURCHASED, WORKS_AT）
- 時間や量などの属性を持つ場合がある

---

## 3. リレーションシップの設計

### 命名規則

```
✅ 良い例：動詞・過去形・大文字スネークケース
:PURCHASED
:WORKS_AT
:FOLLOWS
:CREATED_BY

❌ 悪い例
:purchase（動詞の原形）
:WorksAt（キャメルケース）
:user_order（名詞）
```

### 方向の決め方

意味的に自然な方向を選ぶ：

```cypher
// 自然な方向
(User)-[:PURCHASED]->(Product)      // ユーザーが商品を購入した
(Employee)-[:WORKS_AT]->(Company)   // 従業員が会社で働く
(Post)-[:WRITTEN_BY]->(Author)      // 投稿が著者によって書かれた

// 検索時は方向を無視できる
MATCH (p:Product)<-[:PURCHASED]-(u:User)  // 商品を買ったユーザー
```

### リレーションシップにプロパティを持たせる

```cypher
// いつから働いているか、役職は何か
(alice)-[:WORKS_AT {since: 2020, role: "Engineer"}]->(company)

// 購入数量、価格
(user)-[:PURCHASED {quantity: 2, price: 1000, date: date()}]->(product)
```

---

## 4. ラベルの設計

### 単一ラベル vs 複数ラベル

```cypher
// 複数ラベルで分類を表現
CREATE (bob:Person:Employee:Manager {name: "Bob"})

// ラベルでフィルタリング
MATCH (m:Manager) RETURN m        // マネージャーのみ
MATCH (e:Employee) RETURN e       // 全従業員（マネージャー含む）
MATCH (p:Person) RETURN p         // 全人物
```

### ラベルの粒度

```
✅ 良い例：適度な粒度
:Person, :Company, :Product

❌ 悪い例：細かすぎる
:MaleUser, :FemaleUser, :JapaneseProduct
→ プロパティで表現すべき
```

---

## 5. 中間ノードパターン

### 多対多の関係を詳細化する

```
❌ シンプルだが情報が少ない
(User)-[:PURCHASED]->(Product)

✅ 中間ノードで詳細を表現
(User)-[:PLACED]->(Order)-[:CONTAINS]->(Product)
                     ↓
              {date, total, status}
```

### 実践例：イベント参加

```cypher
// 中間ノード Attendance でイベント参加を表現
CREATE (alice:Person {name: "Alice"})
CREATE (conf:Event {name: "Tech Conference 2024"})
CREATE (attendance:Attendance {
  registeredAt: datetime(),
  ticketType: "VIP",
  paid: true
})
CREATE (alice)-[:REGISTERED]->(attendance)-[:FOR_EVENT]->(conf)
```

### ハンズオン：注文システム

```cypher
// データクリア
MATCH (n) DETACH DELETE n
```

```cypher
// 注文システムのモデル
CREATE (alice:Customer {name: "Alice", email: "alice@example.com"})
CREATE (bob:Customer {name: "Bob", email: "bob@example.com"})

CREATE (laptop:Product {name: "Laptop", price: 120000, category: "Electronics"})
CREATE (mouse:Product {name: "Mouse", price: 3000, category: "Electronics"})
CREATE (book:Product {name: "Graph Database Book", price: 4500, category: "Books"})

// 注文（中間ノード）
CREATE (order1:Order {orderId: "ORD-001", orderDate: date("2024-01-15"), status: "delivered"})
CREATE (order2:Order {orderId: "ORD-002", orderDate: date("2024-02-20"), status: "shipped"})

// 注文明細（中間ノード）
CREATE (item1:OrderItem {quantity: 1, unitPrice: 120000})
CREATE (item2:OrderItem {quantity: 2, unitPrice: 3000})
CREATE (item3:OrderItem {quantity: 1, unitPrice: 4500})

// リレーションシップ
CREATE (alice)-[:PLACED]->(order1)
CREATE (order1)-[:CONTAINS]->(item1)-[:PRODUCT]->(laptop)
CREATE (order1)-[:CONTAINS]->(item2)-[:PRODUCT]->(mouse)

CREATE (bob)-[:PLACED]->(order2)
CREATE (order2)-[:CONTAINS]->(item3)-[:PRODUCT]->(book)
```

```cypher
// クエリ例：Aliceの注文内容
MATCH (c:Customer {name: "Alice"})-[:PLACED]->(o:Order)-[:CONTAINS]->(item)-[:PRODUCT]->(p:Product)
RETURN o.orderId AS 注文番号,
       o.orderDate AS 注文日,
       p.name AS 商品名,
       item.quantity AS 数量,
       item.unitPrice * item.quantity AS 小計
```

---

## 6. 時系列データのモデリング

### イベントチェーン

```cypher
// ユーザーの行動履歴をチェーンで表現
CREATE (u:User {name: "Alice"})
CREATE (e1:Event {type: "login", timestamp: datetime("2024-01-01T10:00:00")})
CREATE (e2:Event {type: "view_product", timestamp: datetime("2024-01-01T10:05:00")})
CREATE (e3:Event {type: "purchase", timestamp: datetime("2024-01-01T10:10:00")})

CREATE (u)-[:PERFORMED]->(e1)
CREATE (e1)-[:NEXT]->(e2)
CREATE (e2)-[:NEXT]->(e3)
```

```cypher
// 行動履歴をたどる
MATCH (u:User {name: "Alice"})-[:PERFORMED]->(first)
MATCH path = (first)-[:NEXT*0..]->(event)
RETURN event.type AS イベント, event.timestamp AS 時刻
ORDER BY event.timestamp
```

---

## 7. 階層構造のモデリング

### 組織図

```cypher
// データクリア
MATCH (n) DETACH DELETE n
```

```cypher
// 組織階層
CREATE (ceo:Employee {name: "CEO", title: "Chief Executive Officer"})
CREATE (cto:Employee {name: "CTO", title: "Chief Technology Officer"})
CREATE (cfo:Employee {name: "CFO", title: "Chief Financial Officer"})
CREATE (dev1:Employee {name: "Dev1", title: "Senior Developer"})
CREATE (dev2:Employee {name: "Dev2", title: "Junior Developer"})
CREATE (acc1:Employee {name: "Acc1", title: "Accountant"})

CREATE (cto)-[:REPORTS_TO]->(ceo)
CREATE (cfo)-[:REPORTS_TO]->(ceo)
CREATE (dev1)-[:REPORTS_TO]->(cto)
CREATE (dev2)-[:REPORTS_TO]->(dev1)
CREATE (acc1)-[:REPORTS_TO]->(cfo)
```

```cypher
// CTOの配下全員を取得
MATCH (cto:Employee {name: "CTO"})<-[:REPORTS_TO*]-(subordinate)
RETURN subordinate.name AS 部下, subordinate.title AS 役職
```

```cypher
// 各人の上司チェーン
MATCH (e:Employee)-[:REPORTS_TO*]->(boss)
RETURN e.name AS 従業員, collect(boss.name) AS 上司チェーン
```

### カテゴリ階層

```cypher
// 商品カテゴリ
CREATE (root:Category {name: "All Products"})
CREATE (electronics:Category {name: "Electronics"})
CREATE (books:Category {name: "Books"})
CREATE (computers:Category {name: "Computers"})
CREATE (phones:Category {name: "Phones"})

CREATE (electronics)-[:SUBCATEGORY_OF]->(root)
CREATE (books)-[:SUBCATEGORY_OF]->(root)
CREATE (computers)-[:SUBCATEGORY_OF]->(electronics)
CREATE (phones)-[:SUBCATEGORY_OF]->(electronics)
```

---

## 8. アンチパターン

### ❌ リストをプロパティに格納

```cypher
// 悪い例
CREATE (u:User {name: "Alice", skills: ["Python", "Java", "Neo4j"]})

// 良い例
CREATE (u:User {name: "Alice"})
CREATE (python:Skill {name: "Python"})
CREATE (u)-[:HAS_SKILL]->(python)
```

### ❌ 過度に汎用的なリレーションシップ

```cypher
// 悪い例：意味が曖昧
(a)-[:RELATED_TO]->(b)

// 良い例：具体的な関係
(a)-[:PURCHASED]->(b)
(a)-[:REVIEWED]->(b)
```

### ❌ リレーションシップの代わりにプロパティで参照

```cypher
// 悪い例
CREATE (order:Order {customerId: "123"})

// 良い例
MATCH (c:Customer {id: "123"})
CREATE (c)-[:PLACED]->(order:Order {...})
```

### ❌ 密結合ノード（スーパーノード）

```cypher
// 問題：1つのノードに大量のリレーションシップ
// 例：全ユーザーが :LIVES_IN -> (Japan) を持つ

// 対策：中間ノードで分散、または別の設計を検討
```

---

## 9. RDBからの移行ガイド

### テーブル → ノード

```
RDB: users テーブル
  id | name  | email
  1  | Alice | alice@example.com

Graph:
  (:User {id: 1, name: "Alice", email: "alice@example.com"})
```

### 外部キー → リレーションシップ

```
RDB: orders テーブル
  id | user_id | total
  1  | 1       | 5000

Graph:
  (:User)-[:PLACED]->(:Order {id: 1, total: 5000})
```

### 中間テーブル → リレーションシップ or 中間ノード

```
RDB: user_roles テーブル（多対多）
  user_id | role_id

Graph（シンプルな場合）:
  (:User)-[:HAS_ROLE]->(:Role)

Graph（属性がある場合）:
  (:User)-[:ASSIGNED]->(:Assignment {since: date})-[:ROLE]->(:Role)
```

---

## まとめ

| 原則 | 説明 |
|------|------|
| ユースケース駆動 | 答えたい質問からモデルを設計 |
| ノード = 名詞 | 検索対象になるエンティティ |
| リレーションシップ = 動詞 | ノード間のアクション・関係 |
| 中間ノード | 多対多の詳細情報を表現 |
| ラベル = 分類 | 複数ラベルで柔軟に分類 |
