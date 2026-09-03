# zmk-config-roBa

<img src="keymap-drawer/roBa.svg" >

## トラックボール機能

### 慣性スクロール

SCROLL レイヤー (K 長押し) でボールを弾いて離すと、iOS のようにスクロールが減衰しながら続きます。
[mjmjm0101/zmk-input-processor-scroll-inertia](https://github.com/mjmjm0101/zmk-input-processor-scroll-inertia) を使用しています。

- 縦スクロール専用 (`axis = <1>`)。Shift を押している間は横スクロールになります。
- スクロール量は 20 カウントで 1 ノッチ (以前のドライバ内蔵スクロールは 16)。慣性の発動条件はモジュール既定値よりかなり緩くしてあり、弱いフリックでも滑ります。
- 調整値は `config/roBa.keymap` の `&scroll_inertia` にまとめてあります。
  - スクロール速度: `&trackball_listener` 内の `&zip_scroll_scaler 1 20` の第 2 引数を変更 (小さいほど速い)。変更したら `&scroll_inertia` の `scale-div` も同じ値にする
  - 慣性が付きにくい: `start` / `move` を下げる。逆にゆっくりスクロールしただけで慣性が付くなら上げるか、`decel-ratio` を下げる
  - 滑りすぎる / 止まりが遅い: `decay-*` を下げる (例 `980`) か `friction` を上げる (例 `100`)
  - 末尾がカクつく: `stop` を上げる
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
- ジェスチャー中とその直後にカーソルが動かないように、ボールの移動量を `&zip_xy_scaler 0 1` で 0 に書き換えています。ZMK の input-listener はレイヤー別チェーンでプロセッサが STOP を返しても無視してイベントを流してしまうため、「捨てる」のではなく「0 にする」必要があります。
- J を離した瞬間に通常のカーソル操作へ戻ります。ボールは指を離した後も少し回り続けるので、弾いた直後に J を離すとその分カーソルが流れることがあります。気になる場合はボールが止まってから J を離してください。
- J は layer-tap なので、押してから判定時間が経つまではレイヤーが有効になりません。この間のボール操作はカーソル移動になるため、J 専用の `lt_gesture` で判定時間を 120ms に短くしています (通常の `&lt` は 200ms)。それでも動く場合は J を押してひと呼吸おいてから弾くか、`tapping-term-ms` をさらに短くしてください。

### 構成

- `config/west.yml`: 上記 2 モジュールを追加 (動作確認したコミットに固定)
- `boards/shields/roBa/roBa.dtsi`: プロセッサのノード定義 (無効状態)
- `boards/shields/roBa/roBa_R.overlay`: 右手側 (central) でのみ有効化。左手側で有効になるとビルドが失敗するため
- `config/roBa.keymap`: 各パラメータとレイヤーごとの処理チェーン

スクロールはドライバの `scroll-layers` ではなく `trackball_listener` の `scroller` で処理しているため、
`CONFIG_PMW3610_SCROLL_TICK` と `CONFIG_PMW3610_INVERT_SCROLL_X` は使われません。
ZMK の `CONFIG_ZMK_POINTING_SMOOTH_SCROLLING` (HID Resolution Multiplier) は macOS がサードパーティ製マウスに対して無視するため有効にしていません。
レイヤーの並びを変えた場合は、`&scroll_inertia` の `layer` と `&trackball_listener` 内の `layers` の番号も合わせて変更してください。
