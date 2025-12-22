# Cypher クエリ言語 - 基礎編

## 概要

CypherはNeo4j専用のクエリ言語で、ASCII Artのような直感的な構文が特徴。

```
SQLの SELECT * FROM users WHERE id = 1
  ↓
Cypherは MATCH (u:User {id: 1}) RETURN u
```

---

## 事前準備：データベースのクリア

```cypher
// 全データを削除（DETACH DELETEでリレーションシップも同時削除）
MATCH (n) DETACH DELETE n
```

---

## 1. CREATE - ノードの作成

### 基本構文

```cypher
CREATE (変数名:ラベル {プロパティ})
```

### 実践

```cypher
// 単純なノード作成
CREATE (p:Person {name: "Alice", age: 30})
```

```cypher
// 複数ラベルを持つノード
CREATE (e:Person:Employee {name: "Bob", department: "Engineering"})
```

### ハンズオン：人物ノードを3つ作成

```cypher
CREATE (alice:Person {name: "Alice", age: 30, city: "Tokyo"})
CREATE (bob:Person {name: "Bob", age: 25, city: "Osaka"})
CREATE (charlie:Person {name: "Charlie", age: 35, city: "Tokyo"})
RETURN alice, bob, charlie
```

---

## 2. MATCH - ノードの検索

### 基本構文

```cypher
MATCH (変数名:ラベル)
RETURN 変数名
```

### 実践

```cypher
// 全Personノードを取得
MATCH (p:Person)
RETURN p
```

```cypher
// プロパティで絞り込み
MATCH (p:Person {name: "Alice"})
RETURN p
```

```cypher
// 全ノードを取得（ラベル指定なし）
MATCH (n)
RETURN n
```

---

## 3. RETURN - 結果の返却

### 基本構文

```cypher
RETURN 変数名
RETURN 変数名.プロパティ
RETURN 変数名.プロパティ AS 別名
```

### 実践

```cypher
// ノード全体を返す
MATCH (p:Person)
RETURN p
```

```cypher
// 特定のプロパティだけ返す
MATCH (p:Person)
RETURN p.name, p.age
```

```cypher
// 別名をつける（AS）
MATCH (p:Person)
RETURN p.name AS 名前, p.age AS 年齢
```

---

## 4. CREATE でリレーションシップ作成

### 基本構文

```cypher
// 方向あり（a → b）
CREATE (a)-[:リレーションシップ名]->(b)

// プロパティ付き
CREATE (a)-[:リレーションシップ名 {プロパティ}]->(b)
```

### リレーションシップの方向

```
(a)-[:KNOWS]->(b)   -- aからbへの方向
(a)<-[:KNOWS]-(b)   -- bからaへの方向
(a)-[:KNOWS]-(b)    -- 方向を問わない（検索時のみ有効）
```

### 実践：既存ノード間にリレーションシップを作成

```cypher
// AliceとBobの間にKNOWS関係を作成
MATCH (a:Person {name: "Alice"}), (b:Person {name: "Bob"})
CREATE (a)-[:KNOWS {since: 2018}]->(b)
RETURN a, b
```

```cypher
// BobとCharlieの間にKNOWS関係を作成
MATCH (b:Person {name: "Bob"}), (c:Person {name: "Charlie"})
CREATE (b)-[:KNOWS {since: 2020}]->(c)
RETURN b, c
```

### 実践：ノードとリレーションシップを同時に作成

```cypher
// 会社ノードを作成し、AliceとBobを所属させる
CREATE (techcorp:Company {name: "TechCorp", industry: "IT"})
WITH techcorp
MATCH (a:Person {name: "Alice"})
CREATE (a)-[:WORKS_AT {role: "Engineer"}]->(techcorp)
WITH techcorp
MATCH (b:Person {name: "Bob"})
CREATE (b)-[:WORKS_AT {role: "Designer"}]->(techcorp)
RETURN techcorp
```

---

## 5. リレーションシップの検索

### 基本構文

```cypher
MATCH (a)-[:リレーションシップ名]->(b)
RETURN a, b
```

### 実践

```cypher
// Aliceが知っている人を検索
MATCH (alice:Person {name: "Alice"})-[:KNOWS]->(friend)
RETURN friend.name AS 友達
```

```cypher
// TechCorpで働いている人を検索
MATCH (p:Person)-[:WORKS_AT]->(c:Company {name: "TechCorp"})
RETURN p.name AS 従業員, c.name AS 会社
```

```cypher
// リレーションシップのプロパティも取得
MATCH (a:Person)-[r:KNOWS]->(b:Person)
RETURN a.name AS 人物, b.name AS 知人, r.since AS 知り合った年
```

---

## 6. 確認クエリ

```cypher
// 全ノードとリレーションシップを表示
MATCH (n)
OPTIONAL MATCH (n)-[r]->(m)
RETURN n, r, m
```

---

## まとめ

| 操作 | 構文 |
|------|------|
| ノード作成 | `CREATE (n:Label {prop: value})` |
| ノード検索 | `MATCH (n:Label) RETURN n` |
| 条件検索 | `MATCH (n:Label {prop: value}) RETURN n` |
| リレーション作成 | `CREATE (a)-[:REL]->(b)` |
| リレーション検索 | `MATCH (a)-[:REL]->(b) RETURN a, b` |

---

## 次のステップ

→ 02_cypher_intermediate.md（WHERE, SET, DELETE, MERGE）
