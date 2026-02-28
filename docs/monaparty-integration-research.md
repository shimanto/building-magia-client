# Monaparty / XMP Integration Research
# Monaparty / XMP 連携技術調査

**Date / 調査日**: 2026-02-28  
**Purpose / 目的**: Building Magia ゲームへのMonapartyウォレット連携・NFTキャラクター・XMPゲーム内通貨機能の実装調査

---

## 1. Overview / 概要

### English
Monaparty is a crypto-asset platform based on the Counterparty protocol, running on the Monacoin blockchain. It enables token issuance, NFT creation, and DEX trading. The native token is **XMP**, which is required to issue named tokens (0.5 XMP + small MONA fee).

### 日本語
MonapartyはMonacoinブロックチェーン上で動作するCounterpartyベースのトークン発行プラットフォームです。トークン発行・NFT作成・DEX取引が可能です。ネイティブトークンは**XMP**で、名前付きトークンの発行には0.5 XMP + 少量のMONAが必要です。

---

## 2. Wallet / ウォレット

### Mpurse (Primary / 主要)

| Item | Detail |
|------|--------|
| Type | Browser Extension (Chrome / Firefox) |
| Chrome Store | https://chrome.google.com/webstore/detail/mpurse/ljkohnccmlcpleonoiabgfggnhpkihaa |
| Firefox Add-on | https://addons.mozilla.org/firefox/addon/mpchain_mpurse/ |
| GitHub | https://github.com/tadajam/mpurse |
| Role | MetaMask equivalent for Monaparty |

**Mpurse is the primary wallet for browser-based DApp integration.**  
MpurseはブラウザベースのDApp連携における主要ウォレットです（EthereumにおけるMetaMaskに相当）。

### Other Wallets / その他のウォレット

| Wallet | URL | Feature |
|--------|-----|---------|
| Monapalette | https://monapalette.komikikaku.com/ | Web wallet with DEX / DEX機能付きウェブウォレット |
| Monya | https://monya-wallet.github.io/ | All-in-one wallet / 多機能ウォレット |
| Counterwallet-mona | https://wallet.monaparty.me/ | Official wallet / 公式ウォレット |

---

## 3. Mpurse JavaScript API

### Connection / 接続確認

```javascript
// Check if Mpurse is installed / Mpurseのインストール確認
if (typeof window.mpurse === 'undefined') {
  alert('Please install Mpurse wallet extension / Mpurse拡張機能をインストールしてください');
}
```

### Get Address / アドレス取得

```javascript
// Requires user approval / ユーザー承認が必要
const address = await window.mpurse.getAddress();
console.log(address); // e.g. "MLinW5mA2Rnu7EjDQpnsrh6Z8APMBH6rAt"
```

### Get Balances / 残高取得

```javascript
// Get all token balances for an address / アドレスの全トークン残高取得
const params = { address: address };
const balances = await window.mpurse.mpchain('balances', params);
// Returns array of { asset, quantity, ... }
```

### Get Specific Asset Balance / 特定アセット残高取得

```javascript
const params = { address: address, asset: 'XMP' };
const balance = await window.mpurse.mpchain('balance', params);
```

### Send Asset / アセット送信

```javascript
// Requires user manual approval / ユーザーの手動承認が必要
const txHash = await window.mpurse.sendAsset(
  'RECIPIENT_ADDRESS',  // 送信先アドレス
  'XMP',                // アセット名
  10.0,                 // 数量
  'plain',              // メモタイプ
  'game_reward'         // メモ内容
);
```

### Sign Message / メッセージ署名（認証用）

```javascript
// Used for login authentication / ログイン認証に使用
const signature = await window.mpurse.signMessage('BuildingMagia_Login_' + Date.now());
```

### Event Listeners / イベントリスナー

```javascript
window.mpurse.updateEmitter
  .on('stateChanged', isUnlocked => {
    console.log('Wallet state:', isUnlocked);
  })
  .on('addressChanged', address => {
    console.log('Address changed:', address);
  });
```

---

## 4. Mpchain API (via Mpurse or Direct HTTP)

### Base URL
```
https://mpchain.info/api/
```

### Key Endpoints / 主要エンドポイント

| Method | Endpoint | Description |
|--------|----------|-------------|
| `address` | `/api/address/{address}` | Address info / アドレス情報 |
| `balances` | `/api/balances/{address}` | All token balances / 全トークン残高 |
| `balance` | `/api/balance/{address}/{asset}` | Single token balance / 単一トークン残高 |
| `issuances` | `/api/issuances/{asset}` | Token issuance info / トークン発行情報 |
| `holders` | `/api/holders/{asset}` | Token holders list / トークン保有者一覧 |
| `sends` | `/api/sends/{address}` | Send history / 送信履歴 |

### Direct HTTP Example / 直接HTTP呼び出し例

```javascript
// Without Mpurse / Mpurseなしで直接呼び出し
const response = await fetch(`https://mpchain.info/api/balances/${address}`);
const data = await response.json();
```

---

## 5. Monacard API

### Purpose / 目的
Monacard links Monaparty tokens to images, enabling NFT card display.  
MonacardはMonapartyトークンに画像を紐付け、NFTカードとして表示するサービスです。

### Key Endpoints / 主要エンドポイント

| Endpoint | Description |
|----------|-------------|
| `http://card.mona.jp/api/card_list` | List of all registered card token names / 全カードトークン名一覧 |
| `http://card.mona.jp/api/card_detail?assets=TOKEN1,TOKEN2` | Card details by asset name / アセット名でカード詳細取得 |
| `http://card.mona.jp/api/card_detail?tag=TAG` | Cards by tag / タグでカード取得 |

### Card Detail Fields / カード詳細フィールド

| Field | Description |
|-------|-------------|
| `asset` | Counterblock API identifier |
| `asset_common_name` | Human-readable token name |
| `card_name` | Card display name / カード表示名 |
| `imgur_url` | Image URL (legacy) / 画像URL（旧） |
| `cid` | IPFS CID for image / 画像のIPFS CID |
| `add_description` | Card description / カード説明 |
| `tag` | Tags (comma-separated) / タグ（カンマ区切り） |

### Image Access / 画像アクセス

```
https://mcspare.nachatdayo.com/image_server/img/{cid}    # Original size
https://mcspare.nachatdayo.com/image_server/img/{cid}l   # 448x640px
https://mcspare.nachatdayo.com/image_server/img/{cid}m   # 224x320px
https://mcspare.nachatdayo.com/image_server/img/{cid}t   # 112x160px (thumbnail)
```

---

## 6. Token Issuance / トークン発行

### Requirements / 必要条件

| Item | Detail |
|------|--------|
| Named token fee | 0.5 XMP + small MONA |
| Random name token | MONA only (no XMP needed) |
| Issuance tools | Monapalette or Counterwallet-mona |
| NFT (quantity=1) | Set `divisible=false`, `quantity=1` |

### Token Properties / トークンプロパティ

- **quantity**: Total supply / 総発行数
- **divisible**: Whether token is divisible / 分割可能フラグ
- **callable**: Whether token can be called back / コール可能フラグ
- **description**: Monacard2.0 JSON for NFT metadata / NFTメタデータ用JSON

### Monacard2.0 NFT Registration / Monacard2.0 NFT登録

To register as a Monacard NFT, write the following JSON into the token's `description` field:  
MonacardのNFTとして登録するには、トークンの`description`フィールドに以下のJSONを書き込む：

```json
{
  "monacard": {
    "name": "護法童子・阿",
    "img_url": "ipfs://bafybei...",
    "tag": "BuildingMagia,Character",
    "add_description": "Building Magiaのキャラクター"
  }
}
```

---

## 7. Existing DApp Examples / 既存DApp事例

### Token Battle (http://blog.utyuu.space/tokenbattle_in/)
- Input: Name + Monaparty address
- Randomly selects 5 tokens from the address
- Assigns strength values and battles
- **Key insight**: Uses address input (no Mpurse), fetches token list via Mpchain API

### モナーアイランド REBIRTH (https://monamonbattle.mona.jp/)
- Uses Monacard API to load user's NFT cards as battle units
- PvP card battle game
- **Key insight**: Monacard NFTs become playable characters directly

### Space Monacoin (https://spacemona.komikikaku.com/)
- Cookie-clicker style game
- Token-based progression

---

## 8. Implementation Plan for Building Magia / Building Magia実装計画

### Phase 1: Wallet Connection / ウォレット接続

```
1. Detect Mpurse installation
2. Request address via getAddress()
3. Display connected address
4. Fetch XMP balance via mpchain('balance')
```

### Phase 2: NFT Character System / NFTキャラクターシステム

```
1. Issue 3 character tokens on Monaparty (護法童子・阿, 緊那羅・吽, 迦楼羅・釈)
   - quantity=1, divisible=false (true NFT)
   - Register as Monacard with character artwork
2. On game load, fetch player's balances
3. Check if player holds any of the 3 character tokens
4. Load held characters as playable units
```

### Phase 3: XMP Currency / XMP通貨

```
1. Display XMP balance in game UI
2. Use XMP as entry fee or reward currency
3. Issue game-specific token (e.g. BMAGIA) for in-game rewards
4. XMP can be used to purchase BMAGIA via Dispenser
```

### Phase 4: Battle with NFT Characters / NFTキャラクターでバトル

```
1. Player connects wallet
2. Game fetches owned character NFTs
3. Player selects team from owned characters
4. Battle proceeds with owned characters' stats
5. Win rewards: BMAGIA tokens sent to player
```

---

## 9. Technical Constraints / 技術的制約

| Constraint | Detail |
|-----------|--------|
| Mpurse required | Players must install browser extension / プレイヤーはブラウザ拡張が必要 |
| CORS | Mpchain API may have CORS restrictions for direct fetch / 直接fetchはCORS制限の可能性 |
| Mobile | Mpurse is desktop-only; mobile needs Monapalette / Mpurseはデスクトップのみ |
| Testnet | Development should use testnet first / 開発はテストネットで先行 |
| Token issuance | Requires Monapalette or Counterwallet-mona (not Mpurse) / トークン発行はMpurse不可 |

---

## 10. Testnet Information / テストネット情報

| Item | Detail |
|------|--------|
| Testnet Mpurse | Available at Chrome Web Store (separate extension) |
| Testnet address prefix | Starts with lowercase `m` (mainnet: uppercase `M`) |
| Get testnet XMP | Send 1 MONA to `msVB7uMdzAwgQuph5pL8Zb7aiYgjYoFH1q` → receive 1494 XMP |
| Testnet Monapalette | https://testnet-monapalette.komikikaku.com/ |
| Testnet Mpchain | https://testnet.mpchain.info/ |

---

## 11. References / 参考資料

| Resource | URL |
|----------|-----|
| Monaparty Portal | https://www.monaparty.me/ |
| Mpurse GitHub | https://github.com/tadajam/mpurse |
| Mpchain API Doc | https://mpchain.info/doc |
| Monacard API | https://card.mona.jp/api_explain |
| Monapalette | https://monapalette.komikikaku.com/ |
| MonaParty開発環境まとめ (Axell) | https://medium.com/axell-corporation/monaparty%E3%81%AE%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83%E3%81%BE%E3%81%A8%E3%82%81-ad56ff7de4c9 |
| Token Battle (example DApp) | http://blog.utyuu.space/tokenbattle_in/ |
| モナーアイランド (example DApp) | https://monamonbattle.mona.jp/ |
