---
title: "sqlgenの紹介"
emoji: "🎈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["sqlc","sqlgen", "sql", "golang"]
published: false
---

## sqlgenとは

[sqlgen](https://github.com/miyataka/sqlgen)は，DBから基本的なSQLを一括生成するCLIツールです．

## なぜ作ったのか

私はとあるプロジェクトで[sqlc](https://github.com/sqlc-dev/sqlc)を利用しています． \
sqlcはとても便利なのですが，単純なINSERT文をはじめ，すべてSQLを手書きする必要があり，さすがに煩雑だと感じたので，一括で生成するCLIを作りました．

## sqlgenのインストール方法
sqlgenは，ライブラリではなく，CLIツールとして開発しており，プログラムから利用することは現状考えていません．
よって，`go get` ではなく `go install` でインストールするほうが望ましいと考えます．
以下のようにinstallできます．

```bash
# postgresqlの場合
go install github.com/miyataka/sqlgen/cmd/psqlgen@latest

# mysqlの場合
go install github.com/miyataka/sqlgen/cmd/mysqlgen@latest
```

お気づきの方もいらっしゃるかもしれませんが，このあたりの構成は，[sqldef](https://github.com/sqldef/sqldef)を参考にさせていただきました．


## sqlgenの実行方法

dsn(data source name)を指定して使います．dsnは必須パラメータです．
標準出力に生成したクエリを出力します．

```bash
psqlgen --dsn="postgres://user:password@localhost:5432/dbname"
# mysqlgen --dsn="user:password@tcp(localhost:3306)/dbname"
```

オプションとして`--sqlc`フラグが存在します．このフラグを指定することにより，sqlcに対応したコメントが同時に生成されるようになります．
```bash
psqlgen --dsn="postgres://user:password@localhost:5432/dbname" --sqlc
# mysqlgen --dsn="user:password@tcp(localhost:3306)/dbname" --sqlc
```


## 中身をすこしだけ紹介
psqlgenとmysqlgenはほぼ同様の構成になっています．psqlgenのほうだけすこしコードを紹介します．

全体の処理の流れとしては
1. dsnをparseして問題ないことを確認
2. dsnをもとにデータベースへ接続
3. information_schema (+alpha)をもとにINSERT, SELECT文を取得
4. 標準出力に出力

面白いところは3だけなので，そこを紹介します．

psqlgen(postgresql)の場合，以下のSQLでINSERT文を取得しています．
コード上は[こちら](https://github.com/miyataka/sqlgen/blob/cb81fd7c8f782143cae78f91cc24539e166dd470/cmd/psqlgen/main.go#L85-L109)
```sql
WITH column_with_placeholders AS (
    SELECT
        c.table_schema,
        c.table_name,
        c.column_name,
        c.ordinal_position,
        '$' || ROW_NUMBER() OVER (PARTITION BY c.table_schema, c.table_name ORDER BY c.ordinal_position) AS placeholder
    FROM
        information_schema.columns c
    WHERE
        c.table_schema = $1 -- schema name
        AND (c.column_default IS NULL OR NOT c.column_default LIKE 'nextval(%'::text ) -- sequenceを除外
)
SELECT
    'INSERT INTO ' || table_name ||
    ' (' || string_agg(column_name, ', ' ORDER BY ordinal_position) || ') ' ||
    'VALUES (' || string_agg(placeholder, ', ' ORDER BY ordinal_position) || ') RETURNING *;' AS insert_statement
FROM
    column_with_placeholders
GROUP BY
    table_schema, table_name
ORDER BY
    table_name;
```

WITHから始まる部分はCTE（Common Table Expression，共通テーブル式）といいます．サブクエリみたいに思っておけばここでは十分です．

`column_with_placeholders` のWHERE句で，default値としてsequenceを利用しているカラムを除外しています．
これは自動発番される`id`などのカラムはINSERT文に含まないほうが個人的に実用的だったからです．

次に，SELECT句の`'$' || ROW_NUMBER() OVER (PARTITION BY c.table_schema, c.table_name ORDER BY c.ordinal_position) AS placeholder`部分に注目してください．
window関数`ROW_NUMBER`を使って，同一スキーマかつ同一テーブル名の範囲でプレースホルダを採番しています．
[`ordinal_position`](https://www.postgresql.jp/document/16/html/infoschema-columns.html#INFOSCHEMA-COLUMNS)
というのは，`information_schema`というメタデータが格納されたスキーマにあるカラムです．カラムの順序が定義されています．
ちなみに `information_schema` は標準SQLとして定義されているそうです．

CTE以外の部分のSELECTは 文字列結合を利用してクエリを組み立ているだけですね．省略しちゃいます．


SELECT文を生成している箇所は[こちら](https://github.com/miyataka/sqlgen/blob/cb81fd7c8f782143cae78f91cc24539e166dd470/cmd/psqlgen/main.go#L153-L199)

```sql
WITH column_list AS (
    SELECT
        c.table_schema,
        c.table_name,
        c.column_name,
        c.ordinal_position
    FROM
        information_schema.columns c
    WHERE
        c.table_schema = $1 -- schema name
),
primary_keys AS (
    SELECT
        kcu.table_schema,
        kcu.table_name,
        kcu.column_name,
        ROW_NUMBER() OVER (PARTITION BY kcu.table_schema, kcu.table_name ORDER BY kcu.column_name) AS placeholder_number
    FROM
        information_schema.table_constraints tc
        JOIN information_schema.key_column_usage kcu
        ON tc.constraint_name = kcu.constraint_name
        AND tc.table_schema = kcu.table_schema
        AND tc.table_name = kcu.table_name
    WHERE
        tc.constraint_type = 'PRIMARY KEY'
        AND kcu.table_schema = $1 -- schema name
),
where_pk AS (
        SELECT
            pk.table_schema,
            pk.table_name,
            string_agg(pk.column_name || ' = $' || pk.placeholder_number, ' AND ') AS cond
        FROM primary_keys pk
        GROUP BY pk.table_schema, pk.table_name
)
SELECT
    'SELECT ' || string_agg(cl.column_name, ', ' ORDER BY cl.ordinal_position) ||
    ' FROM ' || cl.table_name ||
    ' WHERE ' || pk.cond ||
    ';' AS select_statement
FROM
    column_list cl INNER JOIN where_pk pk ON cl.table_schema = pk.table_schema AND cl.table_name = pk.table_name
GROUP BY
    cl.table_schema, cl.table_name, pk.cond
ORDER BY cl.table_name;
```

こちらはCTEがさらに多いですね．CTEをひとつずつみていきましょう． \
まずは`column_list`．
以下の部分です．
```sql
WITH column_list AS (
    SELECT
        c.table_schema,
        c.table_name,
        c.column_name,
        c.ordinal_position
    FROM
        information_schema.columns c
    WHERE
        c.table_schema = $1 -- schema name
),
```

これは`information_schema`に存在するテーブルから，テーブル名とカラム名，内部のカラム順(`ordinal_position`)を取得しているだけです．

次に`primary_keys`． 以下の部分です．
```sql
primary_keys AS (
    SELECT
        kcu.table_schema,
        kcu.table_name,
        kcu.column_name,
        ROW_NUMBER() OVER (PARTITION BY kcu.table_schema, kcu.table_name ORDER BY kcu.column_name) AS placeholder_number
    FROM
        information_schema.table_constraints tc
        JOIN information_schema.key_column_usage kcu
        ON tc.constraint_name = kcu.constraint_name
        AND tc.table_schema = kcu.table_schema
        AND tc.table_name = kcu.table_name
    WHERE
        tc.constraint_type = 'PRIMARY KEY'
        AND kcu.table_schema = $1 -- schema name
),
```

これは `information_schema` の `table_constraints` と `key_column_usage` をJOINして，PKを取得しています．
SELECT部分では，またwindow関数(`ROW_NUMBER`)を使いつつ採番しています．

次に`where_pk`です．以下の部分です．
```sql
where_pk AS (
        SELECT
            pk.table_schema,
            pk.table_name,
            string_agg(pk.column_name || ' = $' || pk.placeholder_number, ' AND ') AS cond
        FROM primary_keys pk
        GROUP BY pk.table_schema, pk.table_name
)
```

さきほどの`primary_keys`を `string_agg` を利用して，一行に集約しつつ，WHERE句に指定できるような条件式として生成しています．

最後に，CTEを利用したSELECT部分です．以下の部分です．
```sql
SELECT
    'SELECT ' || string_agg(cl.column_name, ', ' ORDER BY cl.ordinal_position) ||
    ' FROM ' || cl.table_name ||
    ' WHERE ' || pk.cond ||
    ';' AS select_statement
FROM
    column_list cl INNER JOIN where_pk pk ON cl.table_schema = pk.table_schema AND cl.table_name = pk.table_name
GROUP BY
    cl.table_schema, cl.table_name, pk.cond
ORDER BY cl.table_name;
```

CTEとして定義した`column_list` と `where_pk`をINNER JOINしつつ，GROUP BYしています．
GROUP BYする理由は，column_listはカラムの分だけレコードがあるけれども，必要なのはテーブル単位だからです．


## まとめ
- データベースから基本的なSQLを自動生成する`sqlgen`というツールを開発した．
- sqlcから利用できるコメント自動生成機能付き
- `information_schema` を使うことで，面倒なクエリの一括生成ができるよ
