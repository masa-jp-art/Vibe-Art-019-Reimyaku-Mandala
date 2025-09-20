# 霊脈曼荼羅 – p5.js Interactive Web Art

「霊脈曼荼羅（Reimyaku Mandala）」は、鑑賞者の **声・動き・操作** を「供物」として取り込み、
**反応拡散（Reaction-Diffusion）**・**Perlinフロー**・**Boids（群れ）**・**カレイド対称**・**ポスト的処理**
を統合したブラウザ体験型・生成アートです。

本リポジトリは、仕様書に基づく **鑑賞可能なモック実装** を p5.js で提供します。

---

## 🌐 体験（How to Run）

### 前提
- PC / Mac（モダンブラウザ）
- **HTTPS** または `localhost` でアクセス（マイク・カメラ許可のため）

### 手順
1. このフォルダ内で静的サーバを起動
   - Python: `python3 -m http.server 8000`
   - Node: `npx http-server` または `npx serve`
2. ブラウザで `http://localhost:8000/` を開く
3. 右下の **Mic** / **Cam** ボタンで必要に応じて入力を有効化
   ブラウザの許可ダイアログで「許可」を選択
4. **操作キー**：
   - `1..4`：テーマ切替（対称数・雰囲気）
   - `C`：カラーパレット循環
   - `R`：反応拡散フィールドを再初期化
   - `Space`：フラッシュ演出（注入半径が一時拡大）
   - `P`：PNG保存
5. **録画**：`⏺ 録画` → `⏹ 停止` で WebM をダウンロード

---

## 🎮 インタラクション仕様（要点）

- **マウス／ドラッグ**：反応拡散への「注入」位置。押し続けるほど変化が強まる。
- **マイク（FFT）**：
  - low（低音）→ 全体のスピン／脈動
  - mid（中音）→ 反応拡散の F/k（コントラスト）に反映
  - high（高音）→ 粒子輝度／微振動
  - 拍手等の立ち上がりで **フラッシュ** を自動発火
- **カメラ（動き量）**：画面前の動きが大きいほど、注入と **群れの結合（cohesion）** が強化
- **Boids**：セパレーション／アライメント／コヒージョン＋Perlinフローで滑らかに群舞
- **カレイド**：RD + Trails を回転対称合成し、曼荼羅的な統合像を描出

---

## 🛠 実装構成

- **レンダリング**：p5.js（`WEBGL`）
- **反応拡散**：GLSL フラグメントシェーダで **Gray-Scott** を実装
  → `createFramebuffer()` により **同一GLコンテキストの FBO ping-pong**
- **フロー／粒子**：CPU 側で Boids を更新、2D `createGraphics` に残像描画
- **合成**：最終パスで **カレイド（回転対称）** シェーダ合成＋簡易ビネット
- **音**：p5.sound の `FFT`（`fftSize=1024`, `smoothing=0.8`）
- **映像**：`getUserMedia` + 低解像オフスクリーンで**フレーム差分**→動き量だけ使用
- **録画**：`canvas.captureStream()` + `MediaRecorder`（WebM）

### ファイル一覧
- `index.html`：CDNで p5.js / p5.sound を読み込み、HUD とボタン配置
- `style.css`：HUD/トップバー等の軽量スタイル
- `sketch.js`：本体（シミュレーション・合成・入力処理）
- `.env.example`：外部APIを使う場合の**ダミー例**（本実装では不要）

---

## ⚙️ 主要パラメータ（抜粋）

- RD（Gray-Scott）
  `Du=0.16, Dv=0.08, F∈[0.038,0.072], k∈[0.04,0.07], dt=1.0`
  注入半径 `injectRadius=0.08`（フラッシュで一時拡大）
- Boids
  `count≈280, neigh=60px, w_sep=1.4, w_align=0.8, w_coh≈1.0(+モーション依存)`
- Kaleido
  `sides ∈ {8,10,12,16}`、`spin≈0.004 + low*0.02`

---

## 🔐 プライバシー／アクセシビリティ

- マイク／カメラの処理は **ローカルのみ**（サーバ送信なし）
- 許可しない場合でも**代替動作**（疑似入力）で鑑賞可能
- 点滅は控えめ・色弱でも識別しやすいパレットを同梱

---

## 📦 拡張（.env と外部API）

本実装は外部 API を使用しません。
将来的に「オンラインギャラリー保存」等を追加する場合は、`.env`（ビルド時に Vite などで注入）に API キーを定義し、クライアント側では `import.meta.env` 等から参照してください。
サンプル：`.env.example`

---

## 🧪 トラブルシュート

- **音が反応しない**：初回は **Mic ON** ボタンを押してから発声。ブラウザのマイク許可を確認。
- **カメラが映らない**：**Cam ON** ボタン後、許可ダイアログで「許可」を選択。
- **重い**：ウィンドウを小さくする／マイクやカメラをOFFにする。
- **録画できない**：一部ブラウザで `MediaRecorder` が未対応。Chrome 最新を推奨。

# v2.0-霊脈曼荼羅 – Psychedelic Edition (p5.js)

「霊脈曼荼羅」を **サイケデリックな色彩** に振り切ったブラウザ作品です。
反応拡散 × Boids × カレイド合成に、**コサイン・パレット**と**Sinebow**を用いた
**カラーサイクリング**、**ポスタライズ**、**色収差（RGBスプリット）**を重畳し、
音声・動き・操作に強く反応します。

- **Cosine Palettes**（Inigo Quilez）: `a + b*cos(2π*(c*t + d))` で色を滑らかに生成。
  → 軽量・連続な“波”で、虹色を回転させるのに適します。
- **Sinebow**: `sin(π*(h + φ))^2` をRGBに位相ずらしで適用。
  → 高彩度の虹色を素直に得られます。
- **Color Cycling**: パレット自体を時間や音で回す古典的手法。
- **Chromatic Aberration**: テクスチャをRGBで微小にサンプリングずらし。
  → 周辺で色がにじむ“幻視感”を演出。

参考: Quilezのパレット記事、Sinebowの実装例、カラーサイクリングの解説、
色収差のWebGL実装、シェーダ配色の基礎は下記を参照。

> Inigo Quilez, *Procedural Color Palette*（コサイン・パレット）
> Basecase, *On Rainbows*（Sinebow）
> Wikipedia, *Color cycling*（パレット回転）
> EffectGames, *Old School Color Cycling with HTML5*（HTML5実装例）
> Maxim McNair, *Chromatic Aberration*（RGBオフセット実装）
> Maxime Heckel, *Refraction, dispersion…*（色収差・分散の応用）
> The Book of Shaders, *Colors*（GLSLでの色操作）

---

## 体験方法

### 起動
1. 任意の静的サーバを起動（**HTTPS** または `localhost`）
   - Python: `python3 -m http.server 8000`
   - Node: `npx http-server` / `npx serve`
2. `http://localhost:8000/` を開く
3. 右下HUDから **Mic** / **Cam** を必要に応じて有効化
   ブラウザの許可ダイアログで「許可」を選択

### 操作
- **マウス**：反応拡散への注入（ドラッグで強め）
- **マイク**：
  - 低域（low）→ スピン／脈動
  - 中域（mid）→ RDのF/k（模様コントラスト）
  - 高域（high）→ RGBスプリット強度・粒子スパーク
- **カメラ**：動き量が大きいほど Cohesion↑ / 注入↑
- **キー**：
  - `1..4`：カレイド対称/テーマ
  - `C`：静的4色パレットの循環
  - `X`：**Palette Mode**（Cosine ↔ Sinebow）
  - `V`：**Psy強度**（0 → 0.6 → 1.2 → 2.0）
  - `B`：**ポスタライズ**（OFF → 8 → 16 階調）
  - `R`：RDフィールド再初期化
  - `Space`：フラッシュ（注入半径を一時拡大）
  - `P`：PNG保存
- **録画**：`⏺ 録画` → `⏹ 停止` で WebM ダウンロード

---

## 実装の要点

- **RD（Gray-Scott）**はFBO ping-pongでWebGL計算
- **色生成**は `fragKaleido` 内で
  - Cosine / Sinebow の**位相 `uHueBase`**を**時間 + 音量（mid） + Psy**で回転
  - **Posterize**で階調を量子化（`uPosterize`）
  - **RGB Split**は中心からの放射方向に `±uAberration` だけuvオフセットして
    **R/G/Bを別サンプル** → 周辺で色にじみ
- **HUD**で現在のモードと強度を可視化

---

## パフォーマンス / 注意

- ブラウザ：Chrome 最新推奨（他は最新2メジャー）
- 重いときはウィンドウを小さく／Mic/CamをOFF、`V`でPsyを下げる
- 点滅は控えめ、**低刺激モード**相当として `B` をOFFに

---

## 拡張（任意）

- **ml5.js** の Pose/Hand を加えて、特定ジェスチャで色相ジャンプ
- **高解像PNG**（2x/4xレンダリング）
- **キューブヘリックス**（Green 2011）の忠実実装を追加して、
  知覚輝度が単調増加する虹スケールも選択可

---

## Credits / References
- Inigo Quilez, *Procedural Color Palette*（コサイン・パレット）
- Basecase, *On Rainbows*（Sinebowの実装）
- Wikipedia, *Color cycling*（パレットアニメーション）
- EffectGames, *Old School Color Cycling with HTML5*
- Maxim McNair, *Chromatic Aberration*（RGBオフセット手法）
- Maxime Heckel, *Refraction, dispersion…*（色収差/分散）
- The Book of Shaders, *Colors*（GLSL色変換/合成）

