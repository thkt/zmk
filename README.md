# zmk-config-moNa2-v2

moNa2 split keyboard の ZMK ファームウェア設定。

元プロジェクト: [sayu-hub/zmk-config-moNa2.0](https://github.com/sayu-hub/zmk-config-moNa2.0)

## ハードウェア

| パーツ | 仕様 |
| -------- | ------ |
| MCU | Seeeduino XIAO BLE (nRF52840) x2 |
| トラックボール | PMW3610 センサー（右側） |
| ロータリーエンコーダ | EC11（左側） |
| キーマトリクス | 4x11 (col2row) |
| LED | RGB LED ウィジェット（バッテリー・レイヤー表示） |

**注意**: 右半分が Central（USB 接続・トラックボール側）。通常の ZMK split keyboard とは逆。

## 機能

- DYA Studio 対応（[cormoran/zmk](https://github.com/cormoran/zmk) フォーク使用）
  - BLE 管理 / Settings RPC / Runtime Input Processor / Runtime Sensor Rotate
  - 診断: フリーズ・クラッシュ・予期しないリセットの記録（Watchdog）/ スタック使用量（Devtool）
- トラックボール（PMW3610）+ スクロールレイヤー（初期値は Layer 2、対象レイヤーと速度を DYA Studio から変更可能）
- ロータリーエンコーダ（左側、レイヤーごとの回転動作を DYA Studio から変更可能）
- ZMK Studio 対応
- ディープスリープ: 電池駆動で 15 分間操作がないと各半分が個別にスリープし、その半分のキーを押すと復帰する

## ビルド

GitHub Actions による自動ビルド（push / PR 時）。ファームウェアは Actions の Artifacts からダウンロード。

### ローカルビルド（任意）

```bash
west init -l config
west update
west build -d build/right -s zmk/app -b xiao_ble/nrf52840/zmk -- -DSHIELD="mona2_r rgbled_adapter" -DSNIPPET=studio-rpc-usb-uart
west build -d build/left -s zmk/app -b xiao_ble/nrf52840/zmk -- -DSHIELD="mona2_l rgbled_adapter"
```

## フラッシュ

1. XIAO BLE のリセットボタンを素早く2回押す → USB ドライブとしてマウント
2. ビルドした `.uf2` ファイルをドライブにコピー
3. 右側（mona2_r）→ 左側（mona2_l）の順にフラッシュ

## BLE ペアリング

1. 右側を USB で PC に接続
2. PC の Bluetooth 設定から「mona2」を検索・ペアリング
3. 最大3つのホストプロファイル + DYA Studio 接続に対応

## Settings Reset

ペアリング情報をリセットする場合は `settings_reset` ファームウェアを両側にフラッシュしてから通常ファームウェアを再フラッシュ。

## COROPIT 版の設定

COROPIT を使用する場合、`boards/shields/mona2/mona2_r.overlay` の trackball_listener で軸反転を調整してください。

input-processors の変更例:

```dts
input-processors =
    <&zip_xy_transform (INPUT_TRANSFORM_X_INVERT | INPUT_TRANSFORM_Y_INVERT)>,
    <&mouse_runtime_input_processor>;
```

COROPIT 版ではトラックボールの取り付け方向が異なるため、必要に応じて `INPUT_TRANSFORM_XY_SWAP` を追加してください。

## モジュール構成

| カテゴリ | モジュール | 用途 |
| ---------- | ----------- | ------ |
| Core | cormoran/zmk | DYA Studio 対応 ZMK フォーク（v0.4 系） |
| Widget | zmk-rgbled-widget (caksoylar) | RGB LED によるバッテリー・接続状態・レイヤー表示 |
| Driver | zmk-pmw3610-driver (badjeff) | PMW3610 トラックボールドライバー |
| DYA Studio | zmk-module-ble-management | BLE 管理 UI |
| DYA Studio | zmk-feature-custom-settings | 設定の保存基盤 |
| DYA Studio | zmk-module-settings-rpc | アイドル/スリープ設定 |
| DYA Studio | zmk-module-runtime-input-processor | トラックボールとスクロールの設定 UI |
| DYA Studio | zmk-behavior-runtime-sensor-rotate | エンコーダの回転動作の設定 UI |
| DYA Studio | zmk-feature-watchdog | フリーズ・クラッシュ・予期しないリセットの記録 |
| DYA Studio | zmk-module-devtool | スタック使用量の表示 |
