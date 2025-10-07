# Treasure Data Hive UDF リファレンス

TreasureDataのHive環境で利用可能なユーザー定義関数（UDF）のリファレンスです。

## 日時操作

### TD_TIME_RANGE
指定時間範囲内かを判定。**時間ベースパーティショニングに必須**。

```sql
boolean TD_TIME_RANGE(int/long unix_timestamp, start_time, end_time [, string timezone = 'UTC'])
-- 範囲: start_time <= time < end_time
```

**例**
```sql
WHERE TD_TIME_RANGE(time, '2013-01-01 PDT')  -- 2013-01-01以降
WHERE TD_TIME_RANGE(time, '2013-01-01', '2013-02-01')  -- 1月
```

### TD_INTERVAL
相対時間範囲を計算。WHERE句での使用推奨。

```sql
boolean TD_INTERVAL(int/long time, string interval [, string timezone = 'UTC'])
```

**interval形式**: '-Nd'(N日前), '1M'(今月), '-1h'(過去1時間), '-7d'(過去7日), '-1w'(先週), '1y'(今年)

**例**
```sql
WHERE TD_INTERVAL(time, '-7d')  -- 過去7日間
WHERE TD_INTERVAL(time, '1M')   -- 今月
WHERE TD_INTERVAL(time, '-1h/now')  -- 過去1時間から現在まで
WHERE TD_INTERVAL(time, '-7d', 'JST')  -- JSTでの過去7日
```

### TD_TIME_FORMAT
UNIXタイムスタンプを文字列に変換。

```sql
string TD_TIME_FORMAT(long timestamp, string format [, string timezone = 'UTC'])
```

**例**
```sql
SELECT TD_TIME_FORMAT(time, 'yyyy-MM-dd HH:mm:ss z')  -- 2012-01-01 00:00:00 UTC
SELECT TD_TIME_FORMAT(time, 'yyyy-MM-dd', 'JST')
```

### TD_TIME_PARSE
時間文字列をUNIXタイムスタンプに変換。

```sql
long TD_TIME_PARSE(string time [, string timezone = 'UTC'])
```

### TD_TIME_ADD
タイムスタンプに期間を加算/減算。

```sql
long TD_TIME_ADD(time, string duration [, string timezone = 'UTC'])
-- duration: '1w'(1週間後), '-2d'(2日前), '1h30m'(1時間30分後)
```

**例**
```sql
SELECT ... WHERE TD_TIME_RANGE(time,
    TD_TIME_ADD(TD_SCHEDULED_TIME(), '-1d'),
    TD_SCHEDULED_TIME())  -- スケジュールクエリで過去1日間
```

### TD_DATE_TRUNC
指定単位でタイムスタンプを切り捨て。

```sql
long TD_DATE_TRUNC(string unit, long time [, string timezone = 'UTC'])
-- unit: minute, hour, day, week, month, quarter, year
```

**例**
```sql
SELECT TD_DATE_TRUNC('day', time)  -- 日の開始時刻に切り捨て
SELECT TD_DATE_TRUNC('month', time, 'JST')  -- JSTで月初に切り捨て
```

### TD_SCHEDULED_TIME
スケジュールクエリの実行予定時刻を返す。

```sql
long TD_SCHEDULED_TIME()
```

---

## 集計関数

### TD_FIRST / TD_LAST
比較カラムの最小値/最大値を持つ行の値を取得。

```sql
TD_FIRST(ret_col, cmp_col1, cmp_col2, ...)  -- 最小値の行
TD_LAST(ret_col, cmp_col1, cmp_col2, ...)   -- 最大値の行
```

**例**
```sql
SELECT page_id, TD_FIRST(referer, time) AS first_referer FROM logs GROUP BY page_id
SELECT user, TD_LAST(url, time) AS last_url FROM logs GROUP BY user
```

### TD_AVGIF / TD_SUMIF
条件を満たす行のみ集計。

```sql
double TD_AVGIF(double column, boolean predicate)
double TD_SUMIF(double column, boolean predicate)
```

**例**
```sql
SELECT TD_AVGIF(age, age > 20) FROM tbl
SELECT TD_SUMIF(amount, amount > 0) FROM tbl
```

---

## IP地理情報（IPv4/IPv6対応）

### 基本的な地理情報
```sql
string TD_IP_TO_COUNTRY_CODE(string ip)  -- 国コード（例: JP）
string TD_IP_TO_COUNTRY_NAME(string ip)  -- 国名（例: Japan）
string TD_IP_TO_CITY_NAME(string ip)     -- 都市名
string TD_IP_TO_LATITUDE(string ip)      -- 緯度
string TD_IP_TO_LONGITUDE(string ip)     -- 経度
string TD_IP_TO_TIME_ZONE(string ip)     -- タイムゾーン
string TD_IP_TO_POSTAL_CODE(string ip)   -- 郵便番号
```

### 行政区分・その他
```sql
string TD_IP_TO_MOST_SPECIFIC_SUBDIVISION_NAME(string ip)  -- 詳細な行政区分
string TD_IP_TO_LEAST_SPECIFIC_SUBDIVISION_NAME(string ip) -- 広域な行政区分
array<string> TD_IP_TO_SUBDIVISION_NAMES(string ip)        -- 行政区分リスト
string TD_IP_TO_DOMAIN(string ip)        -- ドメイン
string TD_IP_TO_CONNECTION_TYPE(string ip) -- 接続タイプ（dial-up, cable/DSL, corporate, cellular）
string TD_IP_TO_METRO_CODE(string ip)    -- メトロコード（米国のみ）
```

**使用例**
```sql
SELECT 
    TD_IP_TO_COUNTRY_NAME(ip) AS country,
    TD_IP_TO_CITY_NAME(ip) AS city,
    TD_IP_TO_LATITUDE(ip) AS lat,
    TD_IP_TO_LONGITUDE(ip) AS lon
FROM access_logs
```

---

## ユーザーエージェント解析

### TD_PARSE_AGENT
Wootheeライブラリで解析、Map型で返す。

```sql
MAP(string,string) TD_PARSE_AGENT(string user_agent)
```

**取得可能な情報**
- os: OS名（例: Windows 7、またはキャリア名）
- vendor: ベンダー名（例: Google）
- os_version: OSバージョン（例: NT 6.1）
- name: ブラウザ名（例: Chrome）
- category: デバイス種別（pc, smartphone, mobilephone, appliance, crawler, misc, unknown）
- version: バージョン（例: 16.0.912.77）

**例**
```sql
SELECT 
    TD_PARSE_AGENT(agent)['os'] AS os,
    TD_PARSE_AGENT(agent)['name'] AS browser,
    TD_PARSE_AGENT(agent)['category'] AS device_type,
    TD_PARSE_AGENT(agent)['version'] AS version
FROM www_access
```

### TD_PARSE_USER_AGENT
より詳細なオプション指定が可能。

```sql
string TD_PARSE_USER_AGENT(string user_agent [, string options])
-- options: os, os_family, os_major, os_minor, ua, ua_family, ua_major, ua_minor, device
```

**例**
```sql
SELECT TD_PARSE_USER_AGENT(agent, 'os_family') AS os FROM www_access
```

---

## セッション化・ランキング

### TD_SESSIONIZE
イベントをセッションにグループ化。**サブクエリでCLUSTER BY必須**。

```sql
string TD_SESSIONIZE(int/long timestamp, int timeout_sec, string sessionize_by)
-- 返り値: セッションUUID
```

**例**
```sql
SELECT TD_SESSIONIZE(time, 3600, ip_address) as session_id, time, ip_address, path
FROM (
    SELECT time, ip_address, path FROM web_logs
    DISTRIBUTE BY ip_address SORT BY ip_address, time
) t
```

### TD_X_RANK
パーティション内でのランクを返す。**サブクエリでCLUSTER BY必須**。

```sql
long TD_X_RANK(keys)
```

**例**
```sql
SELECT TD_X_RANK(c), c, u FROM
    (SELECT country AS c, user_id AS u FROM users CLUSTER BY c) t

SELECT TD_X_RANK(c, lc1), c, lc1, u FROM
    (SELECT country AS c, location1 AS lc1, user_id AS u FROM users CLUSTER BY c, lc1) t
```

---

## 数値・文字列・URL

### TD_DIVIDE
ゼロ除算を安全に処理（分母0の場合は0を返す）。

```sql
double TD_DIVIDE(double numerator, double denominator)
```

### TD_MD5
MD5ハッシュダイジェストを計算。

```sql
string TD_MD5(col)
```

### TD_SUBSTRING_INENCODING
指定エンコーディングで部分文字列を取得。

```sql
string TD_SUBSTRING_INENCODING(string str, int max_bytes, string charset)
```

**例**
```sql
SELECT TD_SUBSTRING_INENCODING(column, 10, 'UTF-8') FROM tbl
```

### TD_URL_ENCODE / TD_URL_DECODE
URL文字列のエンコード/デコード。

```sql
string TD_URL_ENCODE(col [, string charset])
string TD_URL_DECODE(col)
```

**例**
```sql
SELECT TD_URL_ENCODE(decoded, 'utf-8'), TD_URL_DECODE(encoded) FROM table
```

---

## その他のユーティリティ

### TD_ARRAY_INDEX
配列の指定インデックスの値を取得。

```sql
int/long/string TD_ARRAY_INDEX(array column, int i)
```

**例**
```sql
SELECT TD_ARRAY_INDEX(ARRAY(11,12,13), 2)  -- 結果: 13
```

### TD_NUMERIC_RANGE
数値範囲を複数行で生成。

```sql
int TD_NUMERIC_RANGE(double start, double end, double step)
```

**例**
```sql
SELECT TD_NUMERIC_RANGE(0, 10, 2)  -- 結果: 0, 2, 4, 6, 8
```

### TD_PIVOT / TD_UNPIVOT
クロス集計とその逆操作。

```sql
TD_PIVOT(key_col, value_col, 'key1,key2')
TD_UNPIVOT('key_name1, key_name2', value_col1, value_col2)
```

### TD_LAT_LONG_TO_COUNTRY
緯度経度を国名に変換。

```sql
string TD_LAT_LONG_TO_COUNTRY(string type, double latitude, double longitude)
-- type: FULL_NAME, THREE_LETTER_ABBREVIATION, POSTAL_ABBREVIATION, SORTABLE_NAME
```

**例**
```sql
SELECT TD_LAT_LONG_TO_COUNTRY('FULL_NAME', 37, -122)
```

---

## エージェント開発のための実践パターン

### 1. 時間範囲フィルタリング（増分処理）
```sql
WHERE TD_TIME_RANGE(time,
    TD_TIME_ADD(TD_SCHEDULED_TIME(), '-1d'),
    TD_SCHEDULED_TIME())
```

### 2. 相対的な時間範囲
```sql
WHERE TD_INTERVAL(time, '-7d')  -- 過去7日間
WHERE TD_INTERVAL(time, '1M')   -- 今月
```

### 3. 最初/最後の値取得
```sql
SELECT 
    user_id,
    TD_FIRST(page_url, time) AS first_page,
    TD_LAST(page_url, time) AS last_page
FROM access_logs
GROUP BY user_id
```

### 4. IPアドレスから地理情報抽出
```sql
SELECT 
    TD_IP_TO_COUNTRY_NAME(ip) AS country,
    TD_IP_TO_CITY_NAME(ip) AS city,
    TD_IP_TO_LATITUDE(ip) AS lat,
    TD_IP_TO_LONGITUDE(ip) AS lon
FROM access_logs
```

### 5. ユーザーエージェント解析
```sql
SELECT 
    TD_PARSE_AGENT(user_agent)['os'] AS os,
    TD_PARSE_AGENT(user_agent)['name'] AS browser,
    TD_PARSE_AGENT(user_agent)['category'] AS device_type
FROM access_logs
```

### 6. セッション分析
```sql
SELECT TD_SESSIONIZE(time, 3600, user_id) as session_id, *
FROM (
    SELECT * FROM events
    DISTRIBUTE BY user_id SORT BY user_id, time
) t
```

---

## 関数カテゴリ別クイックリファレンス

| カテゴリ | 関数 | 用途 |
|---------|------|------|
| **時間フィルタリング** | TD_TIME_RANGE, TD_INTERVAL | WHERE句での時間範囲指定 |
| **時間変換** | TD_TIME_FORMAT, TD_TIME_PARSE, TD_TIME_ADD | タイムスタンプと文字列の変換 |
| **時間集約** | TD_DATE_TRUNC | 日/月/年単位への切り捨て |
| **条件付き集計** | TD_AVGIF, TD_SUMIF | 条件を満たす行のみ集計 |
| **最初/最後** | TD_FIRST, TD_LAST | 時系列での最初/最後の値 |
| **IP地理情報** | TD_IP_TO_* | IPから国/都市/緯度経度など |
| **ユーザーエージェント** | TD_PARSE_AGENT | デバイス/ブラウザ情報 |
| **セッション化** | TD_SESSIONIZE | イベントのセッション化 |
| **ランキング** | TD_X_RANK | グループ内順位 |
| **URL** | TD_URL_ENCODE, TD_URL_DECODE | URLエンコーディング |

---

## 重要な注意事項

### パフォーマンス
- **必ずTD_TIME_RANGEまたはTD_INTERVALをWHERE句で使用**して時間ベースパーティショニングを活用する
- CLUSTER BYはORDER BYよりスケーラブル（並列処理可能）

### 地理情報関数
- HiveとTrino(Presto)でMaxmindデータベースのバージョンが異なる場合があり、結果が一致しないことがある

### セッション化・ランキング
- TD_SESSIONIZE、TD_X_RANKは**必ずサブクエリ内でCLUSTER BYまたはDISTRIBUTE BY + SORT BY**を使用

### 時間操作
- 年（year）と月（month）の期間指定はTD_TIME_ADDでサポートされていない（日数が可変のため）

---

**参考**: https://api-docs.treasuredata.com/en/tools/hive/td_hive_function_reference/