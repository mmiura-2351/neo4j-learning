# Cypher クエリ作成問題 - 理解度チェック

## 事前準備：テスト用データのセットアップ

以下のクエリを実行してテスト用データを作成してください。

```cypher
// データクリア
MATCH (n) DETACH DELETE n
```

```cypher
// テスト用データ作成
// 人物
CREATE (taro:Person {name: "Taro", age: 28, city: "Tokyo"})
CREATE (hanako:Person {name: "Hanako", age: 32, city: "Osaka"})
CREATE (ken:Person {name: "Ken", age: 25, city: "Tokyo"})
CREATE (yuki:Person {name: "Yuki", age: 30, city: "Nagoya"})
CREATE (mai:Person {name: "Mai", age: 27, city: "Tokyo"})
CREATE (ryo:Person {name: "Ryo", age: 35, city: "Fukuoka"})

// 会社
CREATE (alpha:Company {name: "Alpha Inc", industry: "IT", founded: 2015})
CREATE (beta:Company {name: "Beta Corp", industry: "Finance", founded: 2010})
CREATE (gamma:Company {name: "Gamma Ltd", industry: "IT", founded: 2020})

// スキル
CREATE (python:Skill {name: "Python"})
CREATE (java:Skill {name: "Java"})
CREATE (sql:Skill {name: "SQL"})
CREATE (neo4j:Skill {name: "Neo4j"})
CREATE (react:Skill {name: "React"})

// プロジェクト
CREATE (proj1:Project {name: "Web App", budget: 5000000, status: "active"})
CREATE (proj2:Project {name: "Data Pipeline", budget: 3000000, status: "completed"})
CREATE (proj3:Project {name: "Mobile App", budget: 8000000, status: "active"})

// 友人関係 (KNOWS)
CREATE (taro)-[:KNOWS {since: 2018}]->(hanako)
CREATE (taro)-[:KNOWS {since: 2020}]->(ken)
CREATE (hanako)-[:KNOWS {since: 2019}]->(yuki)
CREATE (ken)-[:KNOWS {since: 2021}]->(mai)
CREATE (yuki)-[:KNOWS {since: 2020}]->(ryo)
CREATE (mai)-[:KNOWS {since: 2022}]->(ryo)

// 勤務関係 (WORKS_AT)
CREATE (taro)-[:WORKS_AT {role: "Engineer", since: 2019}]->(alpha)
CREATE (hanako)-[:WORKS_AT {role: "Manager", since: 2018}]->(alpha)
CREATE (ken)-[:WORKS_AT {role: "Developer", since: 2021}]->(beta)
CREATE (yuki)-[:WORKS_AT {role: "Analyst", since: 2020}]->(beta)
CREATE (mai)-[:WORKS_AT {role: "Designer", since: 2022}]->(gamma)
CREATE (ryo)-[:WORKS_AT {role: "CTO", since: 2020}]->(gamma)

// スキル保有 (HAS_SKILL)
CREATE (taro)-[:HAS_SKILL {level: "expert"}]->(python)
CREATE (taro)-[:HAS_SKILL {level: "intermediate"}]->(neo4j)
CREATE (hanako)-[:HAS_SKILL {level: "expert"}]->(sql)
CREATE (hanako)-[:HAS_SKILL {level: "intermediate"}]->(python)
CREATE (ken)-[:HAS_SKILL {level: "expert"}]->(java)
CREATE (ken)-[:HAS_SKILL {level: "beginner"}]->(sql)
CREATE (yuki)-[:HAS_SKILL {level: "expert"}]->(sql)
CREATE (yuki)-[:HAS_SKILL {level: "intermediate"}]->(python)
CREATE (mai)-[:HAS_SKILL {level: "expert"}]->(react)
CREATE (ryo)-[:HAS_SKILL {level: "expert"}]->(python)
CREATE (ryo)-[:HAS_SKILL {level: "expert"}]->(neo4j)

// プロジェクト参加 (PARTICIPATES_IN)
CREATE (taro)-[:PARTICIPATES_IN {role: "Lead"}]->(proj1)
CREATE (hanako)-[:PARTICIPATES_IN {role: "PM"}]->(proj1)
CREATE (ken)-[:PARTICIPATES_IN {role: "Developer"}]->(proj2)
CREATE (yuki)-[:PARTICIPATES_IN {role: "Analyst"}]->(proj2)
CREATE (mai)-[:PARTICIPATES_IN {role: "Designer"}]->(proj3)
CREATE (ryo)-[:PARTICIPATES_IN {role: "Architect"}]->(proj3)
CREATE (taro)-[:PARTICIPATES_IN {role: "Reviewer"}]->(proj2)
```

---

## 基礎レベル（10問）

### 問題 1
全てのPersonノードの名前と年齢を取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)
RETURN p.name AS 名前, p.age AS 年齢
```
</details>

---

### 問題 2
Tokyoに住んでいるPersonを全て取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person {city: "Tokyo"})
RETURN p.name
```

または

```cypher
MATCH (p:Person)
WHERE p.city = "Tokyo"
RETURN p.name
```
</details>

---

### 問題 3
30歳以上のPersonの名前と都市を取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)
WHERE p.age >= 30
RETURN p.name AS 名前, p.city AS 都市
```
</details>

---

### 問題 4
IT業界の会社を全て取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (c:Company {industry: "IT"})
RETURN c.name
```
</details>

---

### 問題 5
Taroが知っている人（直接の友人）の名前を取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (taro:Person {name: "Taro"})-[:KNOWS]->(friend)
RETURN friend.name AS 友人
```
</details>

---

### 問題 6
Alpha Incで働いている人の名前と役職を取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)-[r:WORKS_AT]->(c:Company {name: "Alpha Inc"})
RETURN p.name AS 名前, r.role AS 役職
```
</details>

---

### 問題 7
Pythonスキルを持っている人を全て取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)-[:HAS_SKILL]->(:Skill {name: "Python"})
RETURN p.name
```
</details>

---

### 問題 8
新しいPersonノード（name: "Sato", age: 29, city: "Kyoto"）を作成してください。

<details>
<summary>解答</summary>

```cypher
CREATE (sato:Person {name: "Sato", age: 29, city: "Kyoto"})
RETURN sato
```
</details>

---

### 問題 9
Kenの年齢を26に更新してください。

<details>
<summary>解答</summary>

```cypher
MATCH (ken:Person {name: "Ken"})
SET ken.age = 26
RETURN ken
```
</details>

---

### 問題 10
問題8で作成したSatoノードを削除してください。

<details>
<summary>解答</summary>

```cypher
MATCH (sato:Person {name: "Sato"})
DELETE sato
```
</details>

---

## 中級レベル（10問）

### 問題 11
各都市に住んでいる人数を集計してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)
RETURN p.city AS 都市, count(p) AS 人数
ORDER BY 人数 DESC
```
</details>

---

### 問題 12
Personの平均年齢を計算してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)
RETURN avg(p.age) AS 平均年齢
```
</details>

---

### 問題 13
各会社に所属する従業員の名前をリストで取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)-[:WORKS_AT]->(c:Company)
RETURN c.name AS 会社, collect(p.name) AS 従業員
```
</details>

---

### 問題 14
スキルを2つ以上持っている人の名前とスキル数を取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)-[:HAS_SKILL]->(s:Skill)
WITH p, count(s) AS skillCount
WHERE skillCount >= 2
RETURN p.name AS 名前, skillCount AS スキル数
```
</details>

---

### 問題 15
"expert"レベルのスキルを持つ人とそのスキル名を取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)-[r:HAS_SKILL {level: "expert"}]->(s:Skill)
RETURN p.name AS 名前, s.name AS スキル
```
</details>

---

### 問題 16
activeステータスのプロジェクトに参加している人を取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)-[:PARTICIPATES_IN]->(proj:Project {status: "active"})
RETURN DISTINCT p.name AS 参加者, proj.name AS プロジェクト
```
</details>

---

### 問題 17
TaroとHanakoの両方が参加しているプロジェクトを見つけてください。

<details>
<summary>解答</summary>

```cypher
MATCH (taro:Person {name: "Taro"})-[:PARTICIPATES_IN]->(proj:Project),
      (hanako:Person {name: "Hanako"})-[:PARTICIPATES_IN]->(proj)
RETURN proj.name AS 共通プロジェクト
```
</details>

---

### 問題 18
MERGEを使用して、Skillノード "Go" を作成してください（既に存在すれば再利用）。

<details>
<summary>解答</summary>

```cypher
MERGE (go:Skill {name: "Go"})
ON CREATE SET go.created = datetime()
RETURN go
```
</details>

---

### 問題 19
TokyoまたはOsakaに住んでいて、かつ30歳未満の人を取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)
WHERE p.city IN ["Tokyo", "Osaka"] AND p.age < 30
RETURN p.name, p.city, p.age
```
</details>

---

### 問題 20
年齢順に並べて上位3人のPersonを取得してください（若い順）。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)
RETURN p.name AS 名前, p.age AS 年齢
ORDER BY p.age ASC
LIMIT 3
```
</details>

---

## 上級レベル（10問）

### 問題 21
Taroから2ホップ以内で到達できる全ての人を取得してください（Taro自身は除く）。

<details>
<summary>解答</summary>

```cypher
MATCH (taro:Person {name: "Taro"})-[:KNOWS*1..2]->(reachable:Person)
RETURN DISTINCT reachable.name AS 到達可能な人
```
</details>

---

### 問題 22
TaroからRyoへの最短経路を見つけ、経由する人の名前を全て取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH path = shortestPath(
  (taro:Person {name: "Taro"})-[:KNOWS*]-(ryo:Person {name: "Ryo"})
)
RETURN [node IN nodes(path) | node.name] AS 経路
```
</details>

---

### 問題 23
同じ会社で働いていて、共通のスキルを持つ人のペアを見つけてください。

<details>
<summary>解答</summary>

```cypher
MATCH (p1:Person)-[:WORKS_AT]->(c:Company)<-[:WORKS_AT]-(p2:Person),
      (p1)-[:HAS_SKILL]->(s:Skill)<-[:HAS_SKILL]-(p2)
WHERE p1.name < p2.name
RETURN p1.name AS 人物1, p2.name AS 人物2,
       c.name AS 会社, collect(DISTINCT s.name) AS 共通スキル
```
</details>

---

### 問題 24
友達がいるが、Pythonスキルを持っていない人を見つけてください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)-[:KNOWS]->()
WHERE NOT EXISTS { (p)-[:HAS_SKILL]->(:Skill {name: "Python"}) }
RETURN DISTINCT p.name
```
</details>

---

### 問題 25
各人について、友人数とスキル数を集計してください（友人やスキルがない場合は0）。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)
OPTIONAL MATCH (p)-[:KNOWS]->(friend)
WITH p, count(DISTINCT friend) AS friendCount
OPTIONAL MATCH (p)-[:HAS_SKILL]->(skill)
RETURN p.name AS 名前,
       friendCount AS 友人数,
       count(DISTINCT skill) AS スキル数
```
</details>

---

### 問題 26
IT業界の会社で働いていて、SQLスキルを持つ人を取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)-[:WORKS_AT]->(c:Company {industry: "IT"}),
      (p)-[:HAS_SKILL]->(:Skill {name: "SQL"})
RETURN p.name AS 名前, c.name AS 会社
```
</details>

---

### 問題 27
友達の友達（2ホップ先）だが、直接の友達ではない人を「おすすめの友達」として取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (me:Person {name: "Taro"})-[:KNOWS]->()-[:KNOWS]->(fof:Person)
WHERE me <> fof AND NOT EXISTS { (me)-[:KNOWS]->(fof) }
RETURN DISTINCT fof.name AS おすすめの友達
```
</details>

---

### 問題 28
プロジェクトごとに、参加者全員のスキルをまとめて取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (proj:Project)<-[:PARTICIPATES_IN]-(p:Person)-[:HAS_SKILL]->(s:Skill)
RETURN proj.name AS プロジェクト,
       collect(DISTINCT p.name) AS 参加者,
       collect(DISTINCT s.name) AS チームスキル
```
</details>

---

### 問題 29
友人関係のsinceプロパティを使い、2020年以降に知り合った友人ペアを取得してください。

<details>
<summary>解答</summary>

```cypher
MATCH (p1:Person)-[r:KNOWS]->(p2:Person)
WHERE r.since >= 2020
RETURN p1.name AS 人物1, p2.name AS 人物2, r.since AS 知り合った年
ORDER BY r.since
```
</details>

---

### 問題 30
全Personに対して、スキルレベルを★で表現して出力してください（expert=★★★, intermediate=★★, beginner=★）。

<details>
<summary>解答</summary>

```cypher
MATCH (p:Person)-[r:HAS_SKILL]->(s:Skill)
RETURN p.name AS 名前, s.name AS スキル,
  CASE r.level
    WHEN "expert" THEN "★★★"
    WHEN "intermediate" THEN "★★"
    WHEN "beginner" THEN "★"
    ELSE "未評価"
  END AS レベル
ORDER BY p.name, s.name
```
</details>

---

## 採点基準

| レベル | 正解数 | 評価 |
|--------|--------|------|
| 基礎 | 8-10問 | 基礎構文を理解 |
| 中級 | 7-10問 | 実務で使えるレベル |
| 上級 | 7-10問 | 複雑なグラフ操作が可能 |

---

## 次のステップ

→ 06_performance.md（インデックスとパフォーマンス最適化）
