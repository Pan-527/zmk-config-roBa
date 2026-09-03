# zmk-config-roBa

<img src="keymap-drawer/roBa.svg" >

## トラックボール機能

### 慣性スクロール

SCROLL レイヤー (K 長押し) でボールを弾いて離すと、iOS のようにスクロールが減衰しながら続きます。
[mjmjm0101/zmk-input-processor-scroll-inertia](https://github.com/mjmjm0101/zmk-input-processor-scroll-inertia) を使用しています。

- 縦スクロール専用 (`axis = <1>`)。Shift を押している間は横スクロールになります。
- 調整値は `config/roBa.keymap` の `&scroll_inertia` にまとめてあります。
  - 慣性が付きにくい: `start` / `move` を下げる
  - 滑りすぎる / 止まりが遅い: `decay-*` を下げる (例 `980`) か `friction` を上げる (例 `100`)
  - スクロール速度: `&trackball_listener` 内の `&zip_scroll_scaler 1 16` の第 2 引数を変更 (小さいほど速い)。変更したら `&scroll_inertia` の `scale-div` も同じ値にする
  - 縦方向が逆: `&zip_y_scaler (-1) 1` を `&zip_y_scaler 1 1` にする

### トラックボールジェスチャー

GESTURE レイヤー (J 長押し) でボールを弾くと、方向に応じたキーが送られます。
[zettaface/zmk-input-processor-keybind](https://github.com/zettaface/zmk-input-processor-keybind) を使用しています。

| 方向 | 動作 (macOS) |
| --- | --- |
| 右 | `Ctrl + →` 次のデスクトップ |
| 左 | `Ctrl + ←` 前のデスクトップ |
| 上 | `Ctrl + ↑` Mission Control |
| 下 | `Ctrl + ↓` アプリケーション Exposé |

- 割り当ては `config/roBa.keymap` の `&trackball_gesture` の `bindings` (右, 左, 上, 下 の順) で変更できます。上下が逆なら 3 つ目と 4 つ目を入れ替えてください。
- 感度は `tick` で調整します (小さいほど少ない動きで発火)。

### 構成

- `config/west.yml`: 上記 2 モジュールを追加 (動作確認したコミットに固定)
- `boards/shields/roBa/roBa.dtsi`: プロセッサのノード定義 (無効状態)
- `boards/shields/roBa/roBa_R.overlay`: 右手側 (central) でのみ有効化。左手側で有効になるとビルドが失敗するため
- `config/roBa.keymap`: 各パラメータとレイヤーごとの処理チェーン

スクロールはドライバの `scroll-layers` ではなく `trackball_listener` の `scroller` で処理しているため、
`CONFIG_PMW3610_SCROLL_TICK` と `CONFIG_PMW3610_INVERT_SCROLL_X` は使われません。
レイヤーの並びを変えた場合は、`&scroll_inertia` の `layer` と `&trackball_listener` 内の `layers` の番号も合わせて変更してください。
