# dm2geojson

## 概要

都市計画基本図等のDMデータをGeoJSON形式に変換します。線・面・記号・注記の4種類に分割して出力します。

## 前提条件

- Node.js 18以上

## ディレクトリ構成

```
dm2geojson/
├── index.js          # メインスクリプト
├── dm.js             # DMファイル読み込みクラス
├── dmfiles.js        # DMファイルリスト管理クラス
├── geojsonWriter.js  # GeoJSON出力クラス
├── epsgDefs.js       # 座標参照系定義
├── package.json
├── input/
│   ├── 2500/         # 縮尺1/2500のDMファイル（*.dm）を配置
│   └── 10000/        # 縮尺1/10000のDMファイル（*.dm）を配置
└── output/           # 変換後のGeoJSONファイルが出力される
```

## セットアップ

```bash
npm install
```

## 利用方法

DMファイルを `input/<縮尺>/` フォルダに配置してから実行します。

```bash
# 縮尺1/2500、座標系はデフォルト（EPSG:6674）
node index.js

# 縮尺1/10000
node index.js --scale 10000

# 入力座標系を指定する場合
node index.js --epsg 6675

# 入力フォルダを直接指定する場合
node index.js --scale 2500 --input /path/to/dm_folder
```

### オプション

| オプション | デフォルト | 説明 |
|---|---|---|
| `--scale` | `2500` | 縮尺（出力ファイル名に使用） |
| `--epsg` | `6674` | 入力データの座標参照系（EPSGコード） |
| `--input` | `input/<scale>/` | DMファイルが格納されたフォルダ |

### 出力ファイル

`output/` フォルダに以下の4ファイルが生成されます。

| ファイル名 | 内容 |
|---|---|
| `都市計画基本図_<縮尺>_線.geojson` | 線要素（E2） |
| `都市計画基本図_<縮尺>_面.geojson` | 面要素（E1） |
| `都市計画基本図_<縮尺>_記号.geojson` | 記号・点要素（E5） |
| `都市計画基本図_<縮尺>_注記.geojson` | 注記要素（E7） |

出力されたGeoJSONはQGIS等に読み込むことで地図表示や、tippecanoe等のツールを使ってベクトルタイルへの変換が可能です。

なお、以下のレコードタイプは現在未対応のためスキップされます。

| RecordType | 名称 |
|---|---|
| E3 | 円 |
| E4 | 円弧 |
| E6 | 方向 |
| E8 | 属性 |

### 出力GeoJSONの属性

線・面・記号に共通する属性：

| 属性名 | 説明 | 例 |
|---|---|---|
| `Code` | 分類コード（DMの層番号） | `1106` |
| `Elno` | 要素識別番号 | `08ND392-1106-0001` |
| `RecordType` | レコードタイプ | `E1`（面）、`E2`（線）、`E5`（記号） |
| `DataType` | データタイプ（日本語） | `面`、`線`、`点` |
| `DataKind` | 実データ区分 | `0`（データなし）、`2`（二次元）、`4`（注記） |

注記（E7）のみに追加される属性：

| 属性名 | 説明 | 例 |
|---|---|---|
| `Text` | 注記文字列 | `300` |
| `Vnflag` | 縦横フラグ | `0`（横書き）、`1`（縦書き） |
| `Angle` | 文字の角度（度） | `86` |

## 座標系

出力はすべて EPSG:4326（WGS84 / 緯度経度）です。

入力は `--epsg` で指定します。対応するEPSGコードは以下のとおりです。

| 系 | JGD2011 | JGD2000 |
|---|---|---|
| 第1系 | 6669 | 2443 |
| 第2系 | 6670 | 2444 |
| 第3系 | 6671 | 2445 |
| 第4系 | 6672 | 2446 |
| 第5系 | 6673 | 2447 |
| 第6系 | **6674** | 2448 |
| 第7系 | 6675 | 2449 |
| 第8系 | 6676 | 2450 |
| 第9系 | 6677 | 2451 |
| 第10系 | 6678 | 2452 |
| 第11系 | 6679 | 2453 |
| 第12系 | 6680 | 2454 |
| 第13系 | 6681 | 2455 |
| 第14系 | 6682 | 2456 |
| 第15系 | 6683 | 2457 |
| 第16系 | 6684 | 2458 |
| 第17系 | 6685 | 2459 |
| 第18系 | 6686 | 2460 |
| 第19系 | 6687 | 2461 |

太字（**6674**）がデフォルト値（JGD2011 第6系）です。

## ベクトルタイル作成（参考）

出力したGeoJSONから[tippecanoe](https://github.com/felt/tippecanoe)と[pmtiles](https://github.com/protomaps/go-pmtiles)を使ってベクトルタイルを作成できます。

### 1/2500 → MBTiles

```bash
tippecanoe \
  -o kihonzu_2500.mbtiles \
  -Z15 -z16 \
  -r1 \
  --no-feature-limit \
  --no-tile-size-limit \
  --force \
  -L kihonzu_2500_line:都市計画基本図_2500_線.geojson \
  -L kihonzu_2500_polygon:都市計画基本図_2500_面.geojson \
  -L kihonzu_2500_symbol:都市計画基本図_2500_記号.geojson \
  -L kihonzu_2500_annotation:都市計画基本図_2500_注記.geojson
```

### 1/10000 → MBTiles

```bash
tippecanoe \
  -o kihonzu_10000.mbtiles \
  -Z2 -z14 \
  -r1 \
  --no-feature-limit \
  --no-tile-size-limit \
  --force \
  -L kihonzu_10000_line:都市計画基本図_10000_線.geojson \
  -L kihonzu_10000_polygon:都市計画基本図_10000_面.geojson \
  -L kihonzu_10000_symbol:都市計画基本図_10000_記号.geojson \
  -L kihonzu_10000_annotation:都市計画基本図_10000_注記.geojson
```

### 結合・PMTiles変換

```bash
# 2縮尺を1ファイルに結合
tile-join \
  -o kihonzu.mbtiles \
  --force \
  --no-tile-size-limit \
  kihonzu_10000.mbtiles \
  kihonzu_2500.mbtiles

# PMTilesに変換
pmtiles convert kihonzu.mbtiles kihonzu.pmtiles
pmtiles convert kihonzu_10000.mbtiles kihonzu_10000.pmtiles
pmtiles convert kihonzu_2500.mbtiles kihonzu_2500.pmtiles
```

## データ出典

- 静岡市 都市計画基本図（DMデータ）
  https://data.bodik.jp/dataset/221007_1712212695
