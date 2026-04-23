# Weston Shell 選定ガイド: Desktop Shell vs IVI Shell

## 1. 概要

Weston には複数のシェル実装があり、用途によって選択が大きく変わる。
誤った選択はアーキテクチャの再設計につながるため、初期段階での判断が重要。

---

## 2. 基本特性の比較

| 項目 | Desktop Shell | IVI Shell |
|------|:------------:|:---------:|
| 対象用途 | PC・汎用デスクトップ | 車載・産業用組み込み |
| ウィンドウ操作 | ユーザーが自由に操作 | システムが完全制御 |
| プロトコル | xdg-shell | ivi-application |
| レイヤー管理 | 限定的 | Layer Manager API で厳密制御 |
| 画面レイアウト固定 | 困難 | 容易 |
| マルチスクリーン制御 | 基本的なサポート | 複数ディスプレイの細粒度制御 |
| アプリ間の重なり順制御 | 限定的 | IVI ID + レイヤーで明示管理 |
| タッチ・ジェスチャー | 汎用入力 | 車載 HMI 向け入力制御 |
| 標準化規格 | freedesktop.org | GENIVI Alliance / AGL |
| OSS エコシステム | GTK, Qt (一般向け) | Qt (AGL), Wayland-IVI-Extension |
| 学習コスト | 低〜中 | 中〜高 |

---

## 3. 判断軸ごとの詳細

### 3.1 画面制御の方針

| 問い | Desktop Shell | IVI Shell |
|------|--------------|-----------|
| ユーザーがウィンドウを自由に動かせるか？ | Yes → Desktop | No → IVI |
| 画面レイアウトを仕様として固定したいか？ | No | Yes → IVI |
| 複数アプリの重なり順をコードで制御したいか？ | 困難 | Yes → IVI |

### 3.2 ターゲットデバイス

| デバイス種別 | 推奨シェル | 理由 |
|------------|-----------|------|
| カーナビ・車載ダッシュボード | **IVI** | GENIVI/AGL 標準、安全要件に対応しやすい |
| 産業用 HMI パネル | **IVI** | 固定レイアウト、外部からのレイアウト制御が必要 |
| 開発用ホスト PC | **Desktop** | 開発ツール・ブラウザなど汎用アプリを動かす |
| 汎用 Linux タブレット | **Desktop** | ユーザーインタラクションが主体 |
| デジタルサイネージ（固定表示） | **IVI** | コンテンツ配置をシステムが制御 |
| Raspberry Pi プロトタイプ（HMI付き） | 要件次第 | 下記フローで判断 |

### 3.3 開発・運用コスト

| 観点 | Desktop Shell | IVI Shell |
|------|--------------|-----------|
| 既存 Qt/GTK アプリの流用 | 容易 | ivi-application 対応が必要な場合あり |
| HMI デザイナーとの連携 | 標準的なウィンドウ概念で話しやすい | レイヤー・サーフェス概念の理解が必要 |
| AGL (Automotive Grade Linux) との統合 | 非推奨 | **標準サポート** |
| セーフティ要件 (ASIL等) | 非対応 | 対応実績あり |
| デバッグツール | 豊富 | 限定的（ivi-shell 専用ツールが必要） |

---

## 4. 意思決定フロー

フローチャートは `weston-shell-decision.drawio` を参照。

```
最終用途が車載 or 産業用 HMI?
  ├─ Yes → 画面レイアウトをシステムが制御する?
  │           ├─ Yes → IVI Shell
  │           └─ No  → Desktop Shell (ただし要再検討)
  └─ No  → ユーザーがウィンドウを自由操作する?
              ├─ Yes → Desktop Shell
              └─ No  → IVI Shell (組み込み固定UI)
```

---

## 5. ユースケース別推奨まとめ

| ユースケース | 推奨 | 根拠 |
|------------|:----:|------|
| 車載インフォテインメント (IVI) | **IVI Shell** | AGL 標準、GENIVI 準拠 |
| 車載クラスタ (メーター) | **IVI Shell** | 厳密なレイヤー制御、安全系統分離 |
| 産業用タッチパネル HMI | **IVI Shell** | 固定 UI、外部からレイアウト制御 |
| デジタルサイネージ | **IVI Shell** | コンテンツ管理システムからの制御が容易 |
| 開発・検証用デスクトップ | **Desktop Shell** | 汎用ツール、デバッグのしやすさ |
| 汎用 Linux タブレット | **Desktop Shell** | ユーザー操作前提 |
| Kiosk 端末 | どちらも可 | 固定UI度合いで判断（フロー参照） |

---

## 6. 移行コスト（後から切り替えた場合）

| 変更方向 | コスト | 影響範囲 |
|---------|:------:|---------|
| Desktop → IVI | **高** | アプリの Wayland プロトコル変更、レイアウト設計の見直し |
| IVI → Desktop | **中** | ivi-application 依存コードの除去、HMI 設計の変更 |

**結論: 初期段階で正しく選択することが最も低コスト。**

---

## 7. チェックリスト（最終判断用）

以下の質問に答えて、IVI が多ければ IVI Shell を選択。

- [ ] 車載・産業用途か？ → IVI
- [ ] 画面レイアウトを仕様書で固定するか？ → IVI
- [ ] 複数アプリの重なり順をコードで制御したいか？ → IVI
- [ ] AGL / GENIVI エコシステムを使うか？ → IVI
- [ ] ユーザーがウィンドウを自由に動かせる必要があるか？ → Desktop
- [ ] 開発ホスト PC でも同じバイナリを動かしたいか？ → Desktop
- [ ] 既存の汎用 Qt/GTK アプリをそのまま動かしたいか？ → Desktop

---

## 8. 参考資料

- [Weston 公式ドキュメント](https://wayland.freedesktop.org/weston.html)
- [GENIVI Alliance](https://www.genivi.org/)
- [AGL (Automotive Grade Linux)](https://www.automotivelinux.org/)
- [Wayland IVI Extension](https://github.com/GENIVI/wayland-ivi-extension)
- [ilmControl API リファレンス](https://github.com/GENIVI/wayland-ivi-extension/tree/master/ivi-layermanagement-api)
