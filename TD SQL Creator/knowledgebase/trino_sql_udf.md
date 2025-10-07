# TreasureData Trino SQL UDF リファレンス

## 概要
TreasureDataのTrinoエンジンでは、ネイティブのTrino関数に加えて、TreasureData独自のUDF（User Defined Functions）が利用可能です。これらのUDFは、データ分析やETL処理を効率化するための強力な機能を提供します。

## 主要なUDFカテゴリ

### 1. データ集計・分析系

#### TD_APPROX_MOST_FREQUENT
```sql
TD_APPROX_MOST_FREQUENT(long num_buckets, long/varchar values, long capacity)
```
- **説明**: コレクションから頻出する値を近似的に抽出
- **戻り値**: 要素をキー、推定頻度を値とするマップ
- **例**: 
```sql
SELECT TD_APPROX_MOST_FREQUENT(3, values, 10);
```

#### SMART_DIGEST
```sql
string SMART_DIGEST(col [,weight = 1.0])
```
- **説明**: 文字列から可変長ダイジェストを生成（6-10文字）
- **注意**: 衝突率は約5%。weightパラメータで調整可能

### 2. 時間処理系

#### TD_TIME_RANGE
```sql
boolean TD_TIME_RANGE(int/long unix_timestamp,
                      int/long/string start_time,
                      int/long/string end_time
                      [, string default_timezone = 'UTC'])
```
- **説明**: タイムスタンプが指定範囲内かチェック
- **例**:
```sql
SELECT ... WHERE TD_TIME_RANGE(time, '2013-01-01', '2013-01-02', 'PDT')
```

#### TD_INTERVAL
```sql
boolean TD_INTERVAL(int/long time,
                    string interval_string,
                    [, string default_timezone = 'UTC'])
```
- **説明**: 相対的な時間範囲を計算
- **フォーマット例**:
  - `-7d`: 過去7日間
  - `-1M`: 先月
  - `1y`: 今年
  - `-15m`: 過去15分
- **例**:
```sql
SELECT ... WHERE TD_INTERVAL(time, '-7d', 'JST')
```

#### TD_INTERVAL_RANGE
```sql
TD_INTERVAL_RANGE('interval string', 'time zone')
```
- **説明**: TD_INTERVALの時間範囲を確認
- **戻り値**: ARRAY[(開始時刻), (終了時刻)]

#### TD_DATE_TRUNC
```sql
long TD_DATE_TRUNC(string unit,
                   long time
                   [, string default_timezone = 'UTC'])
```
- **説明**: タイムスタンプを指定単位で切り捨て
- **サポート単位**: 'minute', 'hour', 'day', 'week', 'month', 'quarter', 'year'

#### TD_TIME_FORMAT
```sql
string TD_TIME_FORMAT(long unix_timestamp,
                      string format
                      [, string timezone = 'UTC'])
```
- **説明**: UNIXタイムスタンプを指定フォーマットの文字列に変換
- **例**:
```sql
SELECT TD_TIME_FORMAT(time, 'yyyy-MM-dd HH:mm:ss z', 'JST')
```

#### TD_TIME_PARSE
```sql
long TD_TIME_PARSE(string time
                   [, string default_timezone = 'UTC'])
```
- **説明**: 時刻文字列をUNIXタイムスタンプに変換

#### TD_TIME_ADD
```sql
long TD_TIME_ADD(int/long/string time,
                 string duration
                 [, string default_timezone = 'UTC'])
```
- **説明**: タイムスタンプに期間を加算
- **期間フォーマット**: 'Nw', 'Nd', 'Nh', 'Nm', 'Ns' (N=数値)

#### TD_TIME_STRING
```sql
TD_TIME_STRING(time, 'interval string', time zone?)
```
- **説明**: TD_TIME_FORMATの便利版
- **フォーマット例**:
  - `y`: yyyy-MM-dd HH:mm:ssZ
  - `d!`: yyyy-MM-dd（切り捨て）

#### TD_SCHEDULED_TIME
```sql
long TD_SCHEDULED_TIME()
```
- **説明**: スケジュールクエリの実行予定時刻を返す

### 3. IP/地理情報系

#### TD_IP_TO_COUNTRY_CODE
```sql
string TD_IP_TO_COUNTRY_CODE(string ip)
```
- **説明**: IPアドレスから国コードを取得（IPv4/IPv6対応）
- **例**: 'JP'

#### TD_IP_TO_COUNTRY_NAME
```sql
string TD_IP_TO_COUNTRY_NAME(string ip)
```
- **説明**: IPアドレスから国名を取得
- **例**: 'Japan'

#### TD_IP_TO_CITY_NAME
```sql
string TD_IP_TO_CITY_NAME(string ip)
```
- **説明**: IPアドレスから都市名を取得

#### TD_IP_TO_LATITUDE / TD_IP_TO_LONGITUDE
```sql
string TD_IP_TO_LATITUDE(string ip)
string TD_IP_TO_LONGITUDE(string ip)
```
- **説明**: IPアドレスから緯度/経度を取得

#### TD_IP_TO_CONNECTION_TYPE
```sql
string TD_IP_TO_CONNECTION_TYPE(string ip)
```
- **説明**: 接続タイプを取得
- **戻り値**: 'dial-up', 'cable/DSL', 'corporate', 'cellular'

#### TD_IP_TO_DOMAIN
```sql
string TD_IP_TO_DOMAIN(string ip)
```
- **説明**: IPアドレスからドメインを取得

#### TD_IP_TO_TIME_ZONE
```sql
string TD_IP_TO_TIME_ZONE(string ip)
```
- **説明**: IPアドレスからタイムゾーンを取得

#### TD_IP_TO_POSTAL_CODE
```sql
string TD_IP_TO_POSTAL_CODE(string ip)
```
- **説明**: IPアドレスから郵便番号を取得

#### TD_IP_TO_SUBDIVISION_NAMES
```sql
array<string> TD_IP_TO_SUBDIVISION_NAMES(string ip)
```
- **説明**: 行政区画のリストを取得（州、県など）

#### TD_LAT_LONG_TO_COUNTRY
```sql
string TD_LAT_LONG_TO_COUNTRY(string type, double latitude, double longitude)
```
- **説明**: 緯度経度から国名を取得
- **タイプ**: 'FULL_NAME', 'THREE_LETTER_ABBREVIATION', 'POSTAL_ABBREVIATION'

### 4. ユーザーエージェント解析系

#### TD_PARSE_AGENT
```sql
MAP(varchar,varchar) TD_PARSE_AGENT(user_agent varchar)
```
- **説明**: ユーザーエージェント文字列を解析（Woothee実装）
- **戻り値マップのキー**:
  - `os`: OS名
  - `vendor`: ベンダー名
  - `os_version`: OSバージョン
  - `name`: ブラウザ名
  - `category`: 'pc', 'smartphone', 'mobilephone', 'appliance', 'crawler', 'misc', 'unknown'
  - `version`: ブラウザバージョン
- **例**:
```sql
SELECT element_at(TD_PARSE_AGENT(agent), 'os') AS os FROM www_access
```

#### TD_PARSE_USER_AGENT
```sql
string TD_PARSE_USER_AGENT(user_agent string [, options string])
```
- **説明**: ユーザーエージェント文字列を詳細解析
- **オプション**:
  - `os`: OS情報（JSON）
  - `os_family`: OS名（文字列）
  - `ua`: ユーザーエージェント情報（JSON）
  - `ua_family`: ブラウザ名（文字列）
  - `device`: デバイス情報

### 5. セッション分析系

#### TD_SESSIONIZE_WINDOW
```sql
string TD_SESSIONIZE_WINDOW(int/long unix_timestamp, int timeout)
```
- **説明**: イベントデータをセッション単位にグループ化
- **パラメータ**:
  - unix_timestamp: イベントのタイムスタンプ
  - timeout: セッションタイムアウト（秒）
- **使用例**:
```sql
SELECT
  TD_SESSIONIZE_WINDOW(time, 3600)
  OVER (PARTITION BY ip_address ORDER BY time) as session_id,
  time,
  ip_address,
  path
FROM web_logs
```

### 6. その他のユーティリティ

#### TD_CURRENCY_CONV
```sql
string TD_CURRENCY_CONV(string date, string from_currency, string to_currency, float value)
```
- **説明**: 指定日付の為替レートで通貨変換
- **例**:
```sql
SELECT TD_CURRENCY_CONV('2015-01-01', 'USD', 'JPY', 1.0)
```

#### TD_MD5
```sql
string TD_MD5(col)
```
- **説明**: MD5ハッシュダイジェストを計算

#### TD_URL_DECODE
```sql
string TD_URL_DECODE(col)
string TD_URL_DECODE(url [, locale])
```
- **説明**: URLデコード（EUC-KRもサポート）
- **例**:
```sql
SELECT TD_URL_DECODE('%BA%ED%B7%E7%C5%F5%BD%BA', 'ko')
```

### 7. DELETE文サポート

```sql
DELETE FROM <table_name> [ WHERE <condition> ]
```
- **説明**: 条件に合致する行を削除
- **使用例**:
```sql
-- 航空便の配送記録を削除
DELETE FROM lineitem WHERE shipmode = 'AIR'

-- 特定日のログを削除
DELETE FROM lineitem WHERE TD_TIME_RANGE(time, '2017-07-01','2017-07-02')
```

## 地理空間関数（Geometry型）

### ST_Point
```sql
point ST_Point(double, double)
```
- **説明**: 座標値からポイントオブジェクトを作成

### ST_Polygon
```sql
polygon ST_Polygon(varchar)
```
- **説明**: WKT表現からポリゴンオブジェクトを作成

### ST_Intersects
```sql
boolean ST_Intersects(Geometry, Geometry)
```
- **説明**: 2つのジオメトリが空間的に交差するかチェック

### ST_Intersection
```sql
Geometry ST_Intersection(Geometry, Geometry)
```
- **説明**: 2つのジオメトリの交差部分を取得

## ベストプラクティス

### 1. タイムベースパーティショニングの活用
- `TD_TIME_RANGE`や`TD_INTERVAL`を使用してクエリを最適化
- フルテーブルスキャンを避けてパフォーマンス向上

### 2. ユーザーエージェント解析
- `element_at()`関数を使用してマップから安全に値を取得
- `[]`演算子は存在しないキーでエラーになるため避ける

### 3. IPアドレス解析の注意点
- MaxmindのGeoデータベースを使用
- HiveとTrinoで異なるバージョンを使用する可能性があるため、結果が異なる場合がある

### 4. セッション分析
- `TD_SESSIONIZE_WINDOW`を使用して一貫性のある結果と高パフォーマンスを実現
- OVER句でユーザー識別子ごとにパーティション化

## パフォーマンス最適化Tips

1. **大規模クエリでの結果テーブルクリーンアップ**
```sql
DROP TABLE IF EXISTS my_result;
CREATE TABLE my_result AS SELECT * FROM my_table;
```

2. **時間範囲の指定**
- 常に時間範囲を指定してパーティションプルーニングを活用
- `TD_INTERVAL`で動的な時間範囲を簡潔に記述

3. **近似関数の活用**
- `TD_APPROX_MOST_FREQUENT`で大規模データの頻出値を効率的に抽出
- メモリ使用量を大幅に削減

## 参考リンク
- [TreasureData API Documentation](https://api-docs.treasuredata.com/en/tools/presto/api/)
- [Trino Functions Documentation](https://trino.io/docs/current/functions.html)