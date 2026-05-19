# GemBrawl

4人同時対戦のブラウザベース2Dアクションゲーム。宝石7個を奪い合い、先に全部集めるか時間切れ時に最多保有者が勝利。

## 技術スタック

| フェーズ | 構成 |
|---|---|
| Phase 0（現在） | 単体 `index.html` + Phaser 3 CDN。ビルド不要 |
| Phase 1〜 | Vite + TypeScript + Phaser 3 npm |
| ネットワーク | WebRTC RTCDataChannel + PeerJS |

## ゲーム仕様

### ルール
- ステージに宝石7個
- 攻撃を当てると相手が宝石を1個ドロップ
- 勝利条件: 宝石7個獲得 / 時間切れ時に最多保有

### キャラクター（5人予定）
| キャラ | 必殺技 |
|---|---|
| 主人公 | 一定時間、移動速度2倍 |
| ライバル | 自分中心に円形AoE展開、内側の敵速度1/2 |
| キャラ3〜5 | 未定 |

### 操作（現在のキーバインド）
| 操作 | キー |
|---|---|
| 移動 | ← → / A D |
| ジャンプ | ↑ / W / Space |
| 段降り | ↓ / S |
| 攻撃 | Z |

## ステージ設計

- 固定1画面（800×480）、スクロールなし
- 左右に壁（幅24px）
- 一方通行プラットフォーム（下から飛び抜け可能、上に乗れる）
- 7段のプラットフォーム構成

## ネットワーク方針

外部サービス（PeerJSなど）に依存しない。

- **シグナリング**: ゲーム画面上でSDP（Base64）をコピペして交換。Discord/LINEなどで共有
- **トポロジー**: スター型（ホストが各プレイヤーと1対1接続）。ホストが全員の入力を中継
- **NAT越え**: GoogleのSTUNサーバー（`stun.l.google.com:19302`）のみ使用
- **接続手順**: ① ホストがルームコード生成 → ② 参加者がコードを貼り付け応答コードを生成 → ③ ホストが応答コードを貼り付けて完了

## 開発上の注意

- **ネットワーク同期は決定論的ロックステップ方式** を採用予定。`Math.random()` をゲームロジック内で使わない。乱数が必要な場合は `SeededRNG`（mulberry32）を使うこと
- **シミュレーションと描画を分離** する。`GameSimulation` は純粋関数として副作用なしで実装
- **Phaser 3の物理エンジン** はArcade Physicsを使用。一方通行プラットフォームはprocessコールバックで実装
- Phase 0は単体HTMLのまま機能検証を続ける。環境構築（Vite + TypeScript）はPhase 1で行う

## ディレクトリ構成（Phase 1以降）

```
GemBrawl/
├── src/
│   ├── game/
│   │   ├── GameSimulation.ts    # 決定論的ゲームループ（純粋関数）
│   │   ├── SeededRNG.ts         # Math.random()の代替
│   │   ├── GemSystem.ts
│   │   └── characters/
│   │       ├── CharacterRegistry.ts
│   │       ├── BaseCharacter.ts
│   │       └── SpecialMoveSystem.ts
│   ├── network/
│   │   ├── PeerJSAdapter.ts         # WebRTC/PeerJS分離層
│   │   └── LockstepCoordinator.ts   # フレーム入力収集・同期制御
│   ├── scenes/
│   │   ├── LobbyScene.ts
│   │   ├── GameScene.ts
│   │   └── ResultScene.ts
│   └── main.ts
├── public/
│   ├── maps/      # Tiled JSONマップ
│   └── assets/    # スプライト、タイルセット
├── index.html
├── CLAUDE.md
├── ROADMAP.md
└── package.json   # Phase 1以降
```
