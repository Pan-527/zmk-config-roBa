# zmk-config-roBa

<img src="keymap-drawer/roBa.svg" >

## トラックボール機能

### 慣性スクロール

SCROLL レイヤー (K 長押し) でボールを弾いて離すと、iOS のようにスクロールが減衰しながら続きます。
[mjmjm0101/zmk-input-processor-scroll-inertia](https://github.com/mjmjm0101/zmk-input-processor-scroll-inertia) を使用しています。

- 縦横どちらにもスクロールできます。[kot149/zmk-scroll-snap](https://github.com/kot149/zmk-scroll-snap) がボールの動きを縦か横の強い方に寄せてしばらくロックするので、縦に転がしているときに斜めに流れることはありません。横に転がせば横スクロールになります (境界は約 32°)。慣性は両軸に効きます。
- SCROLL レイヤーでは左親指の Cmd 位置が素の Cmd になっているので、K + Cmd + ボールで Cmd+ホイール (Figma やブラウザのズーム) になります。
- スクロール量は 20 カウントで 1 ノッチ (以前のドライバ内蔵スクロールは 16)。慣性の発動条件はモジュール既定値よりかなり緩くしてあり、弱いフリックでも滑ります。
- 調整値は `config/roBa.keymap` の `&scroll_inertia` にまとめてあります。
  - スクロール速度: `&trackball_listener` 内の `&zip_scroll_scaler 1 20` の第 2 引数を変更 (小さいほど速い)。変更したら `&scroll_inertia` の `scale-div` も同じ値にする
  - 慣性が付きにくい: `start` / `move` を下げる。逆にゆっくりスクロールしただけで慣性が付くなら上げるか、`decel-ratio` を下げる
  - 滑りすぎる / 止まりが遅い: `decay-*` を下げる (例 `980`) か `friction` を上げる (例 `100`)
  - 末尾がカクつく: `stop` を上げる
  - 縦方向が逆: `&zip_y_scaler (-1) 1` を `&zip_y_scaler 1 1` にする

### 中クリック (I 長押し)

I を長押しすると中ボタン (ホイールクリック) を押しっぱなしにします。そのままボールを転がすと中ボタンドラッグになるので、Figma などでキャンバスを掴んで移動できます。タップは通常通り i です。直前 125ms 以内に別のキーを打っていた場合は必ず文字として扱われます。

### トラックボールジェスチャー

GESTURE レイヤー (J 長押し) でボールを弾くと、方向に応じたキーが送られます。
[zettaface/zmk-input-processor-keybind](https://github.com/zettaface/zmk-input-processor-keybind) を使用しています。

| 方向 | 動作 (macOS) |
| --- | --- |
| 右 | `Ctrl + ←` 前のデスクトップ (トラックパッドのスワイプと同じ向き) |
| 左 | `Ctrl + →` 次のデスクトップ |
| 上 | `Ctrl + ↑` Mission Control |
| 下 | `Ctrl + ↓` アプリケーション Exposé |

- 左右は J を押している間は何度でも発火します (右→左の往復ができます)。上下は J を押している間に 1 回だけ発火します。Mission Control はトグルなので、1 回のフリックで 2 回発火すると開いてすぐ閉じてしまうためです。発火後は GESTURE_DONE レイヤー (8) が上下だけを無視し、J を離すと解除されます。
- 左右の割り当ては `config/roBa.keymap` の `&trackball_gesture` の `bindings`、上下は `macros` 内の `gesture_up` / `gesture_down` で変更できます。上下が逆なら `&trackball_gesture_v` の 3 つ目と 4 つ目を入れ替えてください。
- 感度は `&trackball_gesture` の `tick` で調整します (小さいほど少ない動きで発火)。親指は縦に大きく動かしにくいので、判定前に `&zip_y_scaler 2 1` で縦方向だけ 4 倍にしており、縦は横の 4 分の 1 の動きで発火します。縦の効きを変えるときはこの倍率を変えてください。
- ジェスチャー中とその直後にカーソルが動かないように、ボールの移動量を `&zip_xy_scaler 0 1` で 0 に書き換えています。ZMK の input-listener はレイヤー別チェーンでプロセッサが STOP を返しても無視してイベントを流してしまうため、「捨てる」のではなく「0 にする」必要があります。
- J を離した瞬間に通常のカーソル操作へ戻ります。ボールは指を離した後も少し回り続けるので、弾いた直後に J を離すとその分カーソルが流れることがあります。気になる場合はボールが止まってから J を離してください。
- J は layer-tap なので、押してから 200ms 経つまではレイヤーが有効になりません。この間のボール操作はカーソル移動になるため、J を押してひと呼吸おいてから弾いてください。また、直前 125ms 以内に別のキーを打っていた場合は必ず文字として扱われるので (`require-prior-idle-ms`)、文章を打っている最中に j が落ちることはありません。

### 構成

- `config/west.yml`: 上記 2 モジュールを追加 (動作確認したコミットに固定)
- `boards/shields/roBa/roBa.dtsi`: プロセッサのノード定義 (無効状態)
- `boards/shields/roBa/roBa_R.overlay`: 右手側 (central) でのみ有効化。左手側で有効になるとビルドが失敗するため
- `config/roBa.keymap`: 各パラメータとレイヤーごとの処理チェーン

スクロールはドライバの `scroll-layers` ではなく `trackball_listener` の `scroller` で処理しているため、
`CONFIG_PMW3610_SCROLL_TICK` と `CONFIG_PMW3610_INVERT_SCROLL_X` は使われません。
ZMK の `CONFIG_ZMK_POINTING_SMOOTH_SCROLLING` (HID Resolution Multiplier) は macOS がサードパーティ製マウスに対して無視するため有効にしていません。
レイヤーの並びを変えた場合は、`&scroll_inertia` の `layer` と `&trackball_listener` 内の `layers` の番号も合わせて変更してください。
