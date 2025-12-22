# Cypher クエリ言語 - パターンマッチング編

## 事前準備：サンプルデータの作成

より複雑なグラフ構造を作成します。

```cypher
// 全データを削除
MATCH (n) DETACH DELETE n
```

```cypher
// ソーシャルネットワークのサンプルデータ
CREATE (alice:Person {name: "Alice", age: 30})
CREATE (bob:Person {name: "Bob", age: 25})
CREATE (charlie:Person {name: "Charlie", age: 35})
CREATE (diana:Person {name: "Diana", age: 28})
CREATE (eve:Person {name: "Eve", age: 32})
CREATE (frank:Person {name: "Frank", age: 40})

CREATE (techcorp:Company {name: "TechCorp"})
CREATE (startup:Company {name: "StartupX"})
CREATE (bigco:Company {name: "BigCo"})

CREATE (python:Skill {name: "Python"})
CREATE (java:Skill {name: "Java"})
CREATE (neo4j:Skill {name: "Neo4j"})
CREATE (react:Skill {name: "React"})

// 友人関係（KNOWS）
CREATE (alice)-[:KNOWS {since: 2018}]->(bob)
CREATE (alice)-[:KNOWS {since: 2019}]->(charlie)
CREATE (bob)-[:KNOWS {since: 2020}]->(charlie)
CREATE (bob)-[:KNOWS {since: 2021}]->(diana)
CREATE (charlie)-[:KNOWS {since: 2019}]->(eve)
CREATE (diana)-[:KNOWS {since: 2022}]->(eve)
CREATE (eve)-[:KNOWS {since: 2020}]->(frank)

// 勤務関係（WORKS_AT）
CREATE (alice)-[:WORKS_AT]->(techcorp)
CREATE (bob)-[:WORKS_AT]->(techcorp)
CREATE (charlie)-[:WORKS_AT]->(startup)
CREATE (diana)-[:WORKS_AT]->(startup)
CREATE (eve)-[:WORKS_AT]->(bigco)
CREATE (frank)-[:WORKS_AT]->(bigco)

// スキル（HAS_SKILL）
CREATE (alice)-[:HAS_SKILL {level: "expert"}]->(python)
CREATE (alice)-[:HAS_SKILL {level: "intermediate"}]->(neo4j)
CREATE (bob)-[:HAS_SKILL {level: "expert"}]->(react)
CREATE (bob)-[:HAS_SKILL {level: "beginner"}]->(python)
CREATE (charlie)-[:HAS_SKILL {level: "expert"}]->(java)
CREATE (charlie)-[:HAS_SKILL {level: "intermediate"}]->(python)
CREATE (diana)-[:HAS_SKILL {level: "expert"}]->(neo4j)
CREATE (eve)-[:HAS_SKILL {level: "intermediate"}]->(java)
CREATE (frank)-[:HAS_SKILL {level: "expert"}]->(python)
```

```cypher
// 確認
MATCH (n) RETURN n
```

---

## 1. 基本的なパターンマッチング

### 単一リレーションシップ

```cypher
// Aliceの友人を取得
MATCH (alice:Person {name: "Alice"})-[:KNOWS]->(friend)
RETURN friend.name AS 友人
```

### 方向を無視した検索

```cypher
// Aliceと繋がっている人（方向問わず）
MATCH (alice:Person {name: "Alice"})-[:KNOWS]-(person)
RETURN person.name AS 関係者
```

### 複数のリレーションシップタイプ

```cypher
// KNOWSまたはWORKS_ATで繋がっているノード
MATCH (alice:Person {name: "Alice"})-[:KNOWS|WORKS_AT]-(connected)
RETURN connected.name AS 接続先, labels(connected) AS タイプ
```

---

## 2. 可変長パス

### 基本構文

```cypher
// *min..max でホップ数を指定
(a)-[:REL*1..3]->(b)  // 1〜3ホップ
(a)-[:REL*2]->(b)     // ちょうど2ホップ
(a)-[:REL*]->(b)      // 任意の長さ（注意：無限ループの可能性）
```

### 実践：友達の友達

```cypher
// Aliceから2ホップで到達できる人
MATCH (alice:Person {name: "Alice"})-[:KNOWS*2]->(fof)
RETURN DISTINCT fof.name AS 友達の友達
```

```cypher
// Aliceから1〜3ホップで到達できる人
MATCH (alice:Person {name: "Alice"})-[:KNOWS*1..3]->(reachable)
RETURN DISTINCT reachable.name AS 到達可能な人
```

### パスを取得

```cypher
// パス全体を取得
MATCH path = (alice:Person {name: "Alice"})-[:KNOWS*1..3]->(target)
RETURN path, length(path) AS ホップ数
ORDER BY ホップ数
```

---

## 3. 最短経路

### shortestPath

```cypher
// AliceからFrankへの最短経路
MATCH path = shortestPath(
  (alice:Person {name: "Alice"})-[:KNOWS*]-(frank:Person {name: "Frank"})
)
RETURN path, length(path) AS ホップ数
```

### allShortestPaths（全ての最短経路）

```cypher
// 全ての最短経路を取得
MATCH path = allShortestPaths(
  (alice:Person {name: "Alice"})-[:KNOWS*]-(frank:Person {name: "Frank"})
)
RETURN path, length(path) AS ホップ数
```

### 経路上のノードを抽出

```cypher
// 最短経路上の人物名を取得
MATCH path = shortestPath(
  (alice:Person {name: "Alice"})-[:KNOWS*]-(frank:Person {name: "Frank"})
)
RETURN [node IN nodes(path) | node.name] AS 経路
```

---

## 4. パターンの組み合わせ

### 複数パターンのマッチング

```cypher
// 同じ会社で働いていて、同じスキルを持つ人
MATCH (p1:Person)-[:WORKS_AT]->(company)<-[:WORKS_AT]-(p2:Person),
      (p1)-[:HAS_SKILL]->(skill)<-[:HAS_SKILL]-(p2)
WHERE p1.name < p2.name  // 重複排除
RETURN p1.name AS 人物1, p2.name AS 人物2,
       company.name AS 会社, skill.name AS 共通スキル
```

### 三角関係の発見

```cypher
// 友達の友達が自分の友達でもある（三角形）
MATCH (a:Person)-[:KNOWS]->(b:Person)-[:KNOWS]->(c:Person)-[:KNOWS]->(a)
WHERE a.name < b.name AND b.name < c.name  // 重複排除
RETURN a.name, b.name, c.name
```

---

## 5. OPTIONAL MATCH

存在しない場合もNULLとして結果に含める（SQLのLEFT JOINに相当）

```cypher
// 全員のスキルを取得（スキルがない人もNULLで表示）
MATCH (p:Person)
OPTIONAL MATCH (p)-[:HAS_SKILL]->(s:Skill)
RETURN p.name AS 名前, collect(s.name) AS スキル一覧
```

```cypher
// MATCHとOPTIONAL MATCHの違い
// MATCH: スキルがある人のみ
MATCH (p:Person)-[:HAS_SKILL]->(s:Skill)
RETURN p.name, s.name
```

---

## 6. WITH句 - クエリのパイプライン

### 基本構文

```cypher
MATCH ...
WITH 変数1, 変数2, 集計結果 AS 別名
WHERE 条件
MATCH ...
RETURN ...
```

### 実践：中間結果のフィルタリング

```cypher
// 友人が2人以上いる人を見つけ、その友人を取得
MATCH (p:Person)-[:KNOWS]->(friend)
WITH p, count(friend) AS friendCount
WHERE friendCount >= 2
MATCH (p)-[:KNOWS]->(f)
RETURN p.name AS 人物, friendCount AS 友人数, collect(f.name) AS 友人リスト
```

### 実践：サブクエリ的な処理

```cypher
// スキルを2つ以上持つ人とその勤務先
MATCH (p:Person)-[:HAS_SKILL]->(s:Skill)
WITH p, count(s) AS skillCount, collect(s.name) AS skills
WHERE skillCount >= 2
MATCH (p)-[:WORKS_AT]->(c:Company)
RETURN p.name AS 名前, skills AS スキル, c.name AS 勤務先
```

---

## 7. UNWIND - リストの展開

```cypher
// リストを行に展開
WITH ["Python", "Java", "Neo4j"] AS skillNames
UNWIND skillNames AS skillName
MATCH (s:Skill {name: skillName})
RETURN s.name
```

```cypher
// パスのノードを展開
MATCH path = shortestPath(
  (alice:Person {name: "Alice"})-[:KNOWS*]-(frank:Person {name: "Frank"})
)
UNWIND nodes(path) AS node
RETURN node.name AS 経由ノード
```

---

## 8. EXISTS / NOT EXISTS

### パターンの存在確認

```cypher
// Pythonスキルを持つ人
MATCH (p:Person)
WHERE EXISTS { (p)-[:HAS_SKILL]->(:Skill {name: "Python"}) }
RETURN p.name
```

```cypher
// Neo4jスキルを持っていない人
MATCH (p:Person)
WHERE NOT EXISTS { (p)-[:HAS_SKILL]->(:Skill {name: "Neo4j"}) }
RETURN p.name
```

---

## 9. CASE式

```cypher
// 条件分岐
MATCH (p:Person)-[r:HAS_SKILL]->(s:Skill)
RETURN p.name AS 名前, s.name AS スキル,
  CASE r.level
    WHEN "expert" THEN "★★★"
    WHEN "intermediate" THEN "★★"
    WHEN "beginner" THEN "★"
    ELSE "未評価"
  END AS レベル
```

---

## 10. 実践的なクエリ例

### 推薦システム：友達が持っていて自分が持っていないスキル

```cypher
MATCH (me:Person {name: "Alice"})-[:KNOWS]->(friend)-[:HAS_SKILL]->(skill)
WHERE NOT EXISTS { (me)-[:HAS_SKILL]->(skill) }
RETURN skill.name AS 推薦スキル, collect(DISTINCT friend.name) AS 持っている友人
```

### 共通の友人を持つ人を発見

```cypher
MATCH (me:Person {name: "Alice"})-[:KNOWS]->(mutual)<-[:KNOWS]-(other)
WHERE me <> other AND NOT EXISTS { (me)-[:KNOWS]-(other) }
RETURN other.name AS おすすめの人, collect(mutual.name) AS 共通の友人
```

### 会社間のつながり

```cypher
// 異なる会社で働く人同士の友人関係
MATCH (p1:Person)-[:WORKS_AT]->(c1:Company),
      (p2:Person)-[:WORKS_AT]->(c2:Company),
      (p1)-[:KNOWS]->(p2)
WHERE c1 <> c2
RETURN c1.name AS 会社1, p1.name AS 人物1,
       p2.name AS 人物2, c2.name AS 会社2
```

---

## まとめ

| パターン | 構文 | 用途 |
|----------|------|------|
| 可変長パス | `*1..3` | N〜Mホップの探索 |
| 最短経路 | `shortestPath()` | 2点間の最短ルート |
| OPTIONAL MATCH | `OPTIONAL MATCH` | LEFT JOIN相当 |
| WITH | `WITH ... WHERE` | 中間結果のフィルタ |
| UNWIND | `UNWIND list AS item` | リストを行に展開 |
| EXISTS | `EXISTS { pattern }` | パターン存在確認 |

---

## 次のステップ

→ 04_data_modeling.md（データモデリングのベストプラクティス）
