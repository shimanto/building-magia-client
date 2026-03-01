# Building Magia — Token Design Specification
# Building Magia — トークン設計書

**Version / バージョン**: 0.2
**Date / 日付**: 2026-02-28
**Status / ステータス**: Draft / 草案

---

## 1. Concept Overview / コンセプト概要

### English
Building Magia uses the Monaparty protocol to issue NFT cards representing real-world buildings. Each card belongs to one of three character classes (Goho / Kinnara / Karura). The card's uniqueness is determined by a combination of the building's real-world data and 9 randomly assigned direction attributes (8 directions + center), inspired by Japanese esoteric cosmology. Cards are limited-edition NFTs with a defined supply per building.

### 日本語
Building MagiaはMonapartyプロトコルを使用して、実在するビル・建物をモチーフにしたNFTカードを発行します。各カードは3つのキャラクタークラス（ゴホウ / キンナラ / カルラ）のいずれかに属します。カードのユニークさは、建物の実データと、日本の陰陽道に由来する9つの方位属性（8方位＋中央）のランダムな組み合わせによって決定されます。カードは建物ごとに枚数制限のある限定NFTです。

---

## 2. Token Structure / トークン構造

### 2.1 Parent Tokens (Class Tokens) / 親トークン（クラストークン）

3つのクラストークンを親アセットとして発行します。

| Token Name | Class | Color | Cost | Role |
|-----------|-------|-------|------|------|
| `GOHO` | 護法童子（ゴホウ） | Blue / 青 | 50 XMP | Parent of all Goho building cards |
| `KINNARA` | 緊那羅（キンナラ） | Green / 緑 | 50 XMP | Parent of all Kinnara building cards |
| `KARURA` | 迦楼羅（カルラ） | Red-Yellow / 赤黄 | 50 XMP | Parent of all Karura building cards |

### 2.2 Sub-Asset NFT Cards (Building Cards) / サブアセットNFTカード（ビルカード）

各ビル・建物ごとにサブアセットとして発行します。

**Naming Convention / 命名規則:** `{CLASS}.{BUILDING_ID}`

**Token Parameters / トークンパラメータ:**

| Parameter | Value | Reason |
|-----------|-------|--------|
| `quantity` | 10〜1000+ | 枚数制限で希少性を演出 |
| `divisible` | `true` | 追加発行の柔軟性を確保（後でLOCKも可能） |
| `description` | Monacard2.0 JSON | ビル情報・方位属性・画像CIDを格納 |

**Cost per sub-asset / サブアセット発行コスト:** 25 XMP + 少量のMONA

---

## 3. Card Metadata (Monacard2.0 Format) / カードメタデータ

### 3.1 9-Direction Attributes / 9方位属性

各ビルカードは、8方位＋中央の9つの方位属性を持ちます。これらの属性のレアリティ（N, R, SR, SSR）はNFT発行時にランダムに割り当てられ、カードのユニークさを決定します。

| Direction | Name (JP) | Associated Part | Role |
|---|---|---|---|
| N | 子 (Ne) | Foundation / 基礎・地下 | Affects DEF |
| NE | 丑寅 (Ushitora) | Entrance / 鬼門 | Special Abilities |
| E | 卯 (U) | Facade / 外壁 | Affects ATK |
| SE | 辰巳 (Tatsumi) | Windows / 採光 | Affects SPD |
| S | 午 (Uma) | Rooftop / 屋上 | Affects Max HP |
| SW | 未申 (Hitsujisaru) | Utilities / 裏鬼門 | Recovery Abilities |
| W | 酉 (Tori) | Interior / 内装 | Affects Magic Power (MP) |
| NW | 戌亥 (Inui) | Structure / 骨格 | Durability |
| C | 土 (Tsuchi) | Purpose / 用途 | Determines Class (GOHO/KINNARA/KARURA) |

### 3.2 JSON Schema / JSONスキーマ

`add_description`フィールドに、9方位属性を含むゲームステータスをJSON文字列として格納します。

```json
{
  "monacard": {
    "name": "渋谷スクランブルスクエア",
    "img_url": "ipfs://bafybei...",
    "tag": "BuildingMagia,GOHO,Shibuya,Office",
    "add_description": "{\"class\":\"GOHO\",\"building_name\":\"渋谷スクランブルスクエア\",\"atk\":72,\"def\":65,\"spd\":80,\"rarity\":\"SR\",\"directions\":{\"n\":\"SR\",\"ne\":\"R\",\"e\":\"SR\",\"se\":\"N\",\"s\":\"SSR\",\"sw\":\"R\",\"w\":\"SR\",\"nw\":\"N\",\"c\":\"GOHO\"}}"
  }
}
```

---

## 4. Game Currency / ゲーム内通貨

### 4.1 XMP (Primary Currency / 主要通貨)

カードパック購入、バトル参加費、カードアップグレード等に使用します。

### 4.2 KEKKAI (Event Reward Token / イベント報酬トークン)

イベント報酬やランキング賞品として配布する独自トークンです。

| Parameter | Value |
|-----------|-------|
| Token Name | `KEKKAI` |
| 発行費用 | 50 XMP（名前付きトークン） |
| `quantity` | 100,000,000（1億枚） |
| `divisible` | `true` |

---

## 5. NFT Issuance Costs Summary / NFT発行コスト一覧

| Item | XMP Cost | Notes |
|------|----------|-------|
| 親トークン発行（3クラス） | 50 XMP × 3 = 150 XMP | 一度のみ |
| サブアセット発行（ビルカード1種） | 25 XMP | 建物1棟ごと |
| KEKKAI発行 | 50 XMP | 一度のみ |

**初期発行コスト試算（10棟の場合）:**
`150 (親) + 250 (ビル10種) + 50 (KEKKAI) = 450 XMP` + 少量MONA

---

## 6. Divisible=true の設計方針 / Design Policy for divisible=true

ユーザーからの指摘に基づき、`divisible=true`を採用します。これにより、需要に応じた柔軟な追加発行が可能となり、供給量を固定したい場合は後から`LOCK`トランザクションで対応します。

---

## 7. Mpurse Integration Flow / Mpurse連携フロー

（変更なし）

---

## 8. References / 参考資料

（変更なし）
