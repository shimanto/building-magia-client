# Building Magia — NFT Card Design & Issuance Specification
# Building Magia — NFTカード設計と発行仕様書

**Version / バージョン**: 0.3
**Date / 日付**: 2026-03-03
**Status / ステータス**: Final / 最終版

---

## 1. Concept Overview / コンセプト概要

Building MagiaはMonapartyプロトコルを使用して、実在するビル・建物をモチーフにしたNFTカードを発行します。各カードは3つのキャラクタークラス（ゴホウ / キンナラ / カルラ）のいずれかに属します。カードのユニークさは、建物の実データと、日本の陰陽道に由来する9つの方位属性（8方位＋中央）のランダムな組み合わせによって決定されます。カードは建物ごとに枚数制限のある限定NFTです。

---

## 2. Token Structure / トークン構造

### 2.1 Parent Tokens (Class Tokens) / 親トークン（クラストークン）

3つのクラストークンを親アセットとして発行します。これは一度だけ行います。

| Token Name | Class | Color | Cost | Role |
|-----------|-------|-------|------|------|
| `GOHO` | 護法童子（ゴホウ） | Blue / 青 | 50 XMP | 全てのゴホウカードの親 |
| `KINNARA` | 緊那羅（キンナラ） | Green / 緑 | 50 XMP | 全てのキンナラカードの親 |
| `KARURA` | 迦楼羅（カルラ） | Red-Yellow / 赤黄 | 50 XMP | 全てのカルラカードの親 |

### 2.2 Sub-Asset NFT Cards (Building Cards) / サブアセットNFTカード（ビルカード）

各ビル・建物ごとにサブアセットとして発行します。

- **命名規則:** `{CLASS}.{BUILDING_ID}` (例: `GOHO.SHIBUYA_SCRAMBLE`)
- **発行コスト:** 25 XMP + 少量のMONA
- **パラメータ:**
  - `quantity`: 10〜1000+ (枚数制限で希少性を演出)
  - `divisible`: `true` (追加発行の柔軟性を確保。後から`LOCK`可能)
  - `description`: Monacard2.0 JSON (ビル情報・方位属性・画像CIDを格納)

---

## 3. Card Metadata (Monacard2.0 Format) / カードメタデータ

### 3.1 9-Direction Attributes / 9方位属性

各カードは9つの方位属性を持ち、レアリティ（N, R, SR, SSR）がランダムに割り当てられます。

| 方位 | 名称 | 建物パーツ | ゲーム効果 |
|---|---|---|---|
| N | 子 | 基礎・地下 | DEF（防御力） |
| NE | 丑寅（鬼門） | エントランス | 特殊能力 |
| E | 卯 | 外壁・ファサード | ATK（攻撃力） |
| SE | 辰巳 | 窓・採光 | SPD（速度） |
| S | 午 | 屋上・頂部 | Max HP |
| SW | 未申（裏鬼門） | 設備・インフラ | 回復力 |
| W | 酉 | 内装・ロビー | MP（魔法力） |
| NW | 戌亥 | 構造・骨格 | 耐久力 |
| C | 土 | 用途・テナント | クラス決定 |

### 3.2 Image Handling / 画像の取り扱い

カード画像（建物の写真）は**IPFS**にアップロードし、その**CID**（Content Identifier）をメタデータに含めます。これにより、画像データが分散型ネットワーク上で永続的に保持されます。

- **推奨サービス:** [Pinata](https://pinata.cloud/), [NFT.Storage](https://nft.storage/)
- **URL形式:** `ipfs://{CID}`

### 3.3 JSON Schema / JSONスキーマ

`description`フィールドに、Monacard2.0の仕様に準拠した以下のJSONを格納します。`add_description`には、さらにJSONを文字列として埋め込みます。

```json
{
  "monacard": {
    "name": "渋谷スクランブルスクエア",
    "img_url": "ipfs://bafybeigdyrzt5sfp7udm7hu76uh7y26nf3efuylqabf3oclgtqy55fbzdi",
    "tag": "BuildingMagia,GOHO,Shibuya,Office",
    "add_description": "{\"class\":\"GOHO\",\"building_name\":\"渋谷スクランブルスクエア\",\"address\":\"東京都渋谷区渋谷2-24-12\",\"atk\":72,\"def\":65,\"spd\":80,\"rarity\":\"SR\",\"directions\":{\"n\":\"SR\",\"ne\":\"R\",\"e\":\"SR\",\"se\":\"N\",\"s\":\"SSR\",\"sw\":\"R\",\"w\":\"SR\",\"nw\":\"N\",\"c\":\"GOHO\"}}"
  }
}
```

---

## 4. NFT Issuance Procedure / NFT発行手順

### Step 1: 親トークンの発行（初回のみ）

[Monapalette](https://monapalette.komikikaku.com/) 等のツールを使い、`GOHO`, `KINNARA`, `KARURA` の3つの親トークンを発行します。

### Step 2: ビルカードの準備

1. **データ収集:** ビルの名前、住所、写真などの情報を集めます。
2. **画像アップロード:** ビルの写真をIPFSにアップロードし、CIDを取得します。
3. **方位属性生成:** 9方位のレアリティ（N/R/SR/SSR）をランダムに決定します。

### Step 3: JSONメタデータの作成

上記スキーマに従って、`description`に格納する完全なJSON文字列を作成します。

### Step 4: サブアセットの発行

Monapaletteで「サブアセット発行」を選択し、以下の情報を入力します。

- **親アセット:** `GOHO` / `KINNARA` / `KARURA` のいずれか
- **サブアセット名:** `{BUILDING_ID}` (例: `SHIBUYA_SCRAMBLE`)
- **発行量:** 10〜1000+
- **分割可能:** `はい` (divisible=true)
- **詳細:** 作成したJSONメタデータを貼り付け

---

## 5. Game Currency & Costs / ゲーム内通貨とコスト

（変更なし）

