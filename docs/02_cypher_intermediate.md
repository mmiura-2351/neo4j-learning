# Cypher クエリ言語 - 中級編

## 事前準備：サンプルデータの作成

基礎編のデータを使用します。DBをクリアして再作成してください。

```cypher
// 全データを削除
MATCH (n) DETACH DELETE n
```

```cypher
// サンプルデータ作成
CREATE (alice:Person {name: "Alice", age: 30, city: "Tokyo", email: "alice@example.com"})
CREATE (bob:Person {name: "Bob", age: 25, city: "Osaka", email: "bob@example.com"})
CREATE (charlie:Person {name: "Charlie", age: 35, city: "Tokyo"})
CREATE (diana:Person {name: "Diana", age: 28, city: "Nagoya"})
CREATE (techcorp:Company {name: "TechCorp", industry: "IT", founded: 2010})
CREATE (startup:Company {name: "StartupX", industry: "IT", founded: 2020})

CREATE (alice)-[:KNOWS {since: 2018}]->(bob)
CREATE (bob)-[:KNOWS {since: 2020}]->(charlie)
CREATE (alice)-[:KNOWS {since: 2019}]->(diana)
CREATE (alice)-[:WORKS_AT {role: "Engineer", since: 2018}]->(techcorp)
CREATE (bob)-[:WORKS_AT {role: "Designer", since: 2020}]->(techcorp)
CREATE (diana)-[:WORKS_AT {role: "CEO", since: 2020}]->(startup)
```

---

## 1. WHERE - 条件指定

### 基本構文

```cypher
MATCH (n:Label)
WHERE 条件
RETURN n
```

### 比較演算子

| 演算子 | 意味 |
|--------|------|
| `=` | 等しい |
| `<>` | 等しくない |
| `<`, `>` | 小なり、大なり |
| `<=`, `>=` | 以下、以上 |

### 実践：基本的なWHERE

```cypher
// 30歳以上のPersonを検索
MATCH (p:Person)
WHERE p.age >= 30
RETURN p.name, p.age
```

```cypher
// Tokyoに住んでいるPersonを検索
MATCH (p:Person)
WHERE p.city = "Tokyo"
RETURN p.name, p.city
```

### 論理演算子（AND, OR, NOT）

```cypher
// Tokyoに住んでいて30歳以上
MATCH (p:Person)
WHERE p.city = "Tokyo" AND p.age >= 30
RETURN p.name, p.age, p.city
```

```cypher
// TokyoまたはOsakaに住んでいる
MATCH (p:Person)
WHERE p.city = "Tokyo" OR p.city = "Osaka"
RETURN p.name, p.city
```

```cypher
// Tokyo以外に住んでいる
MATCH (p:Person)
WHERE NOT p.city = "Tokyo"
RETURN p.name, p.city
```

### IN演算子

```cypher
// 複数の値のいずれかに一致
MATCH (p:Person)
WHERE p.city IN ["Tokyo", "Osaka"]
RETURN p.name, p.city
```

### 文字列操作

| 関数 | 意味 |
|------|------|
| `STARTS WITH` | 前方一致 |
| `ENDS WITH` | 後方一致 |
| `CONTAINS` | 部分一致 |

```cypher
// 名前がAで始まる
MATCH (p:Person)
WHERE p.name STARTS WITH "A"
RETURN p.name
```

```cypher
// メールがexample.comで終わる
MATCH (p:Person)
WHERE p.email ENDS WITH "example.com"
RETURN p.name, p.email
```

### NULL チェック

```cypher
// emailプロパティが存在しないPersonを検索
MATCH (p:Person)
WHERE p.email IS NULL
RETURN p.name
```

```cypher
// emailプロパティが存在するPersonを検索
MATCH (p:Person)
WHERE p.email IS NOT NULL
RETURN p.name, p.email
```

---

## 2. SET - プロパティの更新

### 基本構文

```cypher
MATCH (n:Label {条件})
SET n.property = value
RETURN n
```

### 実践：プロパティの追加・更新

```cypher
// Charlieにemailを追加
MATCH (c:Person {name: "Charlie"})
SET c.email = "charlie@example.com"
RETURN c
```

```cypher
// Aliceの年齢を更新
MATCH (a:Person {name: "Alice"})
SET a.age = 31
RETURN a.name, a.age
```

### 複数プロパティを一度に更新

```cypher
// 複数プロパティを更新
MATCH (b:Person {name: "Bob"})
SET b.age = 26, b.city = "Tokyo"
RETURN b
```

### プロパティをまとめて上書き（+=）

```cypher
// 既存プロパティを保持しつつ追加・更新
MATCH (d:Person {name: "Diana"})
SET d += {email: "diana@startup.com", phone: "090-1234-5678"}
RETURN d
```

### ラベルの追加

```cypher
// Aliceに新しいラベルを追加
MATCH (a:Person {name: "Alice"})
SET a:Employee:Manager
RETURN labels(a) AS ラベル一覧
```

---

## 3. REMOVE - プロパティ・ラベルの削除

### 基本構文

```cypher
MATCH (n:Label {条件})
REMOVE n.property
RETURN n
```

### 実践

```cypher
// Dianaのphoneプロパティを削除
MATCH (d:Person {name: "Diana"})
REMOVE d.phone
RETURN d
```

```cypher
// Aliceからmanagerラベルを削除
MATCH (a:Person {name: "Alice"})
REMOVE a:Manager
RETURN labels(a) AS ラベル一覧
```

---

## 4. DELETE - ノード・リレーションシップの削除

### 基本構文

```cypher
// リレーションシップの削除
MATCH (a)-[r:REL]->(b)
DELETE r

// ノードの削除（リレーションシップがないこと）
MATCH (n:Label {条件})
DELETE n

// ノードとリレーションシップを同時に削除
MATCH (n:Label {条件})
DETACH DELETE n
```

### 実践

```cypher
// 新しいテスト用ノードを作成
CREATE (test:Person {name: "TestUser", age: 99})
```

```cypher
// 作成したノードを削除
MATCH (t:Person {name: "TestUser"})
DELETE t
```

```cypher
// リレーションシップ付きノードを作成
CREATE (temp:Person {name: "TempUser"})-[:KNOWS]->(t2:Person {name: "TempUser2"})
```

```cypher
// DETACH DELETEでリレーションシップごと削除
MATCH (t:Person)
WHERE t.name STARTS WITH "Temp"
DETACH DELETE t
```

---

## 5. MERGE - 存在確認付き作成

### 基本構文

```cypher
MERGE (n:Label {識別プロパティ})
ON CREATE SET n.prop = value  // 新規作成時のみ実行
ON MATCH SET n.prop = value   // 既存ノード発見時のみ実行
RETURN n
```

### MERGEとCREATEの違い

| 操作 | 挙動 |
|------|------|
| CREATE | 常に新規作成（重複を許容） |
| MERGE | 存在すれば再利用、なければ作成 |

### 実践：ノードのMERGE

```cypher
// 存在しないノード → 新規作成される
MERGE (e:Person {name: "Eve"})
ON CREATE SET e.age = 22, e.city = "Fukuoka", e.created = datetime()
ON MATCH SET e.lastSeen = datetime()
RETURN e
```

```cypher
// 同じクエリを再実行 → 既存ノードが返される（ON MATCHが実行される）
MERGE (e:Person {name: "Eve"})
ON CREATE SET e.age = 22, e.city = "Fukuoka", e.created = datetime()
ON MATCH SET e.lastSeen = datetime()
RETURN e
```

### 実践：リレーションシップのMERGE

```cypher
// AliceとEveの間にKNOWS関係を作成（存在しなければ）
MATCH (a:Person {name: "Alice"}), (e:Person {name: "Eve"})
MERGE (a)-[r:KNOWS]->(e)
ON CREATE SET r.since = 2024
RETURN a, r, e
```

---

## 6. 便利な集計関数

### COUNT, SUM, AVG, MIN, MAX

```cypher
// Personの総数
MATCH (p:Person)
RETURN count(p) AS 人数
```

```cypher
// 平均年齢
MATCH (p:Person)
RETURN avg(p.age) AS 平均年齢
```

```cypher
// 都市別の人数
MATCH (p:Person)
RETURN p.city AS 都市, count(p) AS 人数
ORDER BY 人数 DESC
```

### COLLECT - 結果をリストにまとめる

```cypher
// 各都市に住む人の名前をリスト化
MATCH (p:Person)
RETURN p.city AS 都市, collect(p.name) AS 住民
```

---

## 7. ORDER BY, LIMIT, SKIP

```cypher
// 年齢順にソート
MATCH (p:Person)
RETURN p.name, p.age
ORDER BY p.age DESC
```

```cypher
// 上位2件のみ取得
MATCH (p:Person)
RETURN p.name, p.age
ORDER BY p.age DESC
LIMIT 2
```

```cypher
// ページネーション（2件目から3件取得）
MATCH (p:Person)
RETURN p.name, p.age
ORDER BY p.age
SKIP 1 LIMIT 3
```

---

## まとめ

| 操作 | 構文 | 用途 |
|------|------|------|
| WHERE | `WHERE 条件` | 検索条件の指定 |
| SET | `SET n.prop = value` | プロパティの更新 |
| REMOVE | `REMOVE n.prop` | プロパティ・ラベルの削除 |
| DELETE | `DELETE n` / `DETACH DELETE n` | ノード・関係の削除 |
| MERGE | `MERGE (n:Label {prop})` | 存在確認付き作成 |

---

## 次のステップ

→ 03_pattern_matching.md（リレーションシップとパターンマッチング）
