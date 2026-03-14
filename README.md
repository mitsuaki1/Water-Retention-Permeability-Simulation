# 土質による保水力・透水力の比較シミュレーション

## 🌱 概要 / Overview

土質による保水力・透水力の比較シミュレーションは、**粘性土・砂質土・団粒構造**という3種類の土壌において、降雨時の水の浸透・保水・表面流出をリアルタイムで可視化するブラウザベースのシミュレーションです。外部ライブラリ不要・単一HTMLファイルで動作します。

This browser-based simulation visualizes water infiltration, retention, and surface runoff in real time across three soil types — **clayey soil, sandy soil, and aggregate structure**. No external libraries required; runs as a single HTML file.

粘性土では表面流出が卓越し、砂質土では高速浸透が起こり、団粒構造は「高透水性と高保水性の同時達成」という特異な性質を持つことを、粒子物理と土壌水文学の数式によって可視化します。

In clayey soils, surface runoff dominates. Sandy soils exhibit rapid deep infiltration. Aggregate structure uniquely achieves both high permeability and high water retention simultaneously — a property visualized here through particle physics and soil hydrological equations.

本プロジェクトは**醸す谷 Vallis Fermenti**の実践記録の一部であり、有機農業・大地の再生・発酵の交差点において「制御ではなく条件のデザイン」を探究するシリーズです。

This project is part of the **醸す谷 / Vallis Fermenti** practice series, exploring the design of conditions — not control — at the intersection of organic farming, land regeneration, and fermentation.

---

## 🎯 目的 / Purpose

慣行農業は土壌の水分挙動を「管理すべきもの」として扱います。しかし有機農業・大地の再生・団粒構造の育成といった実践は、土壌が自律的に水を浸透・保持・供給できる**条件を設計する**という原理に基づいています。

Conventional agriculture treats soil water behavior as something to be managed. Organic farming, land regeneration, and aggregate structure cultivation, however, are based on designing the **conditions** under which soil can autonomously infiltrate, retain, and supply water.

このシミュレーターの目的：

- 粘性土・砂質土・団粒構造の保水・透水挙動をリアルタイムに比較する
- van Genuchten・Green-Ampt・二重間隙モデルなどの土壌水文学的数式を視覚化する
- 団粒構造の「ミクロポアで数日間保水しながらマクロポアで速やかに浸透する」二重性を実証する
- 締固め圧・傾斜角・有機物量・粒度組成がKsatと保水性に与える影響を定量的に示す

Objectives of this simulator:

- Compare water retention and permeability across clay, sand, and aggregate soils in real time
- Visualize soil hydrological equations: van Genuchten, Green-Ampt, and dual-porosity models
- Demonstrate the dual nature of aggregate structure — rapid infiltration via macro-pores while retaining water in micro-pores for days
- Quantitatively show the effects of compaction, slope, organic matter, and particle size on Ksat and retention

---

## 🧬 シミュレーション設計 / Simulation Architecture

### 土質 3種 / Three Soil Types

| 土質 / Soil Type | 特徴 / Characteristics | 参考 Ksat |
|---|---|---|
| 粘性土 / Clayey Soil | 低間隙率・低透水・高保水・表面流出優勢 | 0.001–0.1 mm/h |
| 砂質土 / Sandy Soil | 大粒径・高透水・低保水・深部浸透速 | 100–720 mm/h |
| 団粒構造 / Aggregate Structure | マクロ+ミクロ二重間隙・高透水かつ高保水 | 10–100 mm/h |

### 粒子物理 / Particle Physics

各雨滴粒子は地表面に到達すると **浸透 or 表面流出** に確率的に分岐します。

Each raindrop particle probabilistically splits into **infiltration or surface runoff** upon reaching the soil surface.

```
p_infilt = Ksat / (Ksat + I + 0.5)
```

浸透した粒子は土質ごとの物理で運動します：

- **粘性土**: 極低速（Δvy ≈ 0.012 × por）、粒子に衝突して保持
- **砂質土**: 高速重力（Δvy ≈ 0.22 × por）、底部まで一気に抜ける
- **団粒構造**: マクロポア（高速）→ クラスター内（減速・保持判定）の二段階

### 団粒保水の時間スケール / Aggregate Retention Time Scale

| スケール | 値 |
|---|---|
| 1 frame の実時間 | 10 分 |
| 団粒内保水の時定数 τ | 864 frames ≈ **6 日** |
| マクロポア排水の時定数 | 6–36 frames ≈ **1–6 時間** |

団粒内ミクロポア（r ≈ 0.1–10 µm）の保水は Young-Laplace 式で説明される毛管力により維持され、実測では 2–7 日のスパンで保持されます（Gerke & van Genuchten 1993）。

---

## 📊 物理モデルと数式 / Physical Model & Equations

| # | モデル / Model | 数式 / Formula | 出典 / Reference |
|---|---|---|---|
| ① | 飽和透水係数 Ksat | `Ksat_clay = 0.04 × (1−f_clay)^3.5 × (1−σ)^0.85` | Schaap et al. 2001 |
| ② | Green-Ampt 浸透式 | `f(t) = Ksat × (1 + ψ_f·Δθ / F(t))` | Green & Ampt 1911 |
| ③ | van Genuchten モデル | `θ(h) = θ_r + (θ_s−θ_r) / [1+\|αh\|^n]^m` | van Genuchten 1980 |
| ④ | 二重間隙モデル | `∂θ_im/∂t = −α_w·(h_f−h_im) / (1−w_f)` | Gerke & van Genuchten 1993 |
| ⑤ | Young-Laplace 毛管力 | `ψ = −2σ·cosθ_c / r` | Marshall & Holmes 1988 |
| ⑥ | Darcy 則 / 締固め効果 | `k ∝ e³/(1+e)·C_k`, `v = −Ksat·(dh/dz)` | Taylor 1948 |

### van Genuchten パラメータ（Carsel & Parrish 1988）

| 土質 | θ_r | θ_s | α | n |
|---|---|---|---|---|
| 粘土 / Clay | 0.068 | 0.38 | 0.008 | 1.09 |
| 砂 / Sand | 0.045 | 0.43 | 0.145 | 2.68 |
| 団粒（壌土近似）| 0.078 | 0.46 | 0.036 | 1.56 |

---

## ⚙️ 実装技術 / Technical Implementation

- **単一HTMLファイル**: HTML5 Canvas + バニラ JavaScript、外部依存なし
- **オフスクリーン描画**: 各土壌を VIRT_H=440px で描画 → CROP_PX=130px をクロップして 220px に拡大表示（全3パネル同一画角）
- **60fps 描画**: `requestAnimationFrame` ループ
- **時間スケール**: 1 frame = 10 分（実時間対応）
- **最大粒子数**: 900 粒子（FIFO 管理）
- **団粒クラスター**: `retainedWater` 変数による指数減衰（τ = 864 frames）
- **透水係数 Ksat**: パラメータ変更時にリアルタイム再計算・表示

```
Single HTML file
  ├── Physics engine      浸透確率・重力・粒子衝突
  ├── Grain renderer      粘土（円形微粒）/ 砂（角形ポリゴン）/ 団粒（クラスター）
  ├── Aggregate system    クラスターごとのretainedWater管理
  ├── Ksat calculator     Rosettaモデル近似
  ├── Graph renderer      含水率・流出量・浸透速度のリアルタイムグラフ
  └── Time display        経過時間・団粒保水残存率タイムバー
```

---

## 🎛️ パラメータ / Parameters

### 共通パラメータ / Common Parameters

| スライダー | 範囲 | 説明 |
|---|---|---|
| 降水量 | 0–30 mm/h | Ksat との比で浸透/流出が決まる |
| 締固め圧 | 0–100% | 間隙比を低下させ Ksat を指数的に減少 |
| 傾斜角 | 0–15° | 表面流出の速度・方向に影響 |
| 有機物量 | 0–15% | 団粒形成・保水性・Ksat に正の影響 |

### 個別パラメータ / Per-soil Parameters

| 土質 | パラメータ | 範囲 | 影響 |
|---|---|---|---|
| 粘性土 | 粘土割合 | 50–100% | 高いほど Ksat 低下・保水増大 |
| 砂質土 | 砂割合 | 50–100% | 高いほど Ksat 急増・保水低下 |
| 団粒構造 | 団粒形成量 | 10–100% | 高いほどマクロポア増・Ksat 向上かつ保水も向上 |

---

## ▶️ 実行方法 / How to Run

```bash
# リポジトリをクローン
git clone https://github.com/mitsulab/Water-Retention-Permeability-Simulation.git

# ブラウザで開く（インストール不要）
open index.html        # macOS
start index.html       # Windows
xdg-open index.html    # Linux
```

インストール不要・依存ライブラリなし・ビルドステップなし。

No installation, no dependencies, no build steps.

### GitHub Pages での公開 / Deploy via GitHub Pages

Settings → Pages → Source: `main` branch / `/ (root)` → Save

https://mitsuaki1.github.io/Water-Retention-Permeability-Simulation/

---

## 🖥️ 推奨ブラウザ / Recommended Browsers

- Google Chrome（推奨 / Recommended）
- Microsoft Edge
- Firefox

---

## 📁 ファイル構成 / File Structure

```
Water-Retention-Permeability-Simulation/
├── index.html    # シミュレーション本体（単一ファイル・外部依存なし）
└── README.md     # 本ファイル
```

---

## 🌍 理論的背景 / Theoretical Background

### 団粒構造と二重間隙 / Aggregate Structure & Dual Porosity

団粒構造土壌が「保水性と透水性を同時に達成する」メカニズムは、間隙の二重構造にあります。

The mechanism by which aggregate soil achieves both water retention and permeability lies in its dual pore structure:

- **マクロポア（団粒間）**: r ≈ 50–500 µm、ψ ≈ −0.3 to −3 kPa → 重力排水・速い浸透（時定数 1–6 h）
- **ミクロポア（団粒内）**: r ≈ 0.1–10 µm、ψ ≈ −15 to −1500 kPa → 毛管保水・数日間保持（時定数 2–7 日）

この二重性により、降雨時には素早く浸透し流出を防ぎながら、植物が数日かけてゆっくりと水を吸収できる環境が形成されます。

This dual structure enables rapid infiltration during rainfall (preventing runoff), while providing a slow-release water reservoir that plants can access over several days.

### 大地の再生との接点 / Connection to Land Regeneration Practice

矢野智徳・高田宏臣らが実践する「大地の再生」の核心は「水脈と空気脈のつなぎ直し」です。本シミュレーターが示す締固め土壌での流出優勢・団粒構造での深部浸透は、その原理の計算科学的な表現です。

The core of the land regeneration practice advocated by Yano Tomonori and Takada Hiroomi is "reconnecting water and air pathways." The runoff dominance in compacted soil and deep infiltration in aggregate soil demonstrated by this simulator are computational expressions of that principle.

---

## 📄 参考文献 / References

- Schaap M.G. et al. (2001) ROSETTA: a computer program for estimating soil hydraulic parameters. *J. Hydrology* 251.
- van Genuchten M.Th. (1980) A closed-form equation for predicting the hydraulic conductivity of unsaturated soils. *SSSAJ* 44.
- Green W.H., Ampt G.A. (1911) Studies on soil physics. *J. Agricultural Science* 4(1).
- Gerke H.H., van Genuchten M.Th. (1993) A dual-porosity model for simulating the preferential movement of water and solutes in structured porous media. *Water Resources Research* 29(2).
- Carsel R.F., Parrish R.S. (1988) Developing joint probability distributions of soil water retention characteristics. *Water Resources Research* 24(5).
- Marshall T.J., Holmes J.W. (1988) *Soil Physics*. 2nd ed. Cambridge University Press.
- Taylor D.W. (1948) *Fundamentals of Soil Mechanics*. John Wiley & Sons.

---

## 🚧 現在の限界と今後の課題 / Limitations & Future Work

### 現在の限界 / Current Limitations

- 二次元シミュレーション（実際の土壌は三次元の細孔・団粒構造を持つ）
- パラメータは文献の平均値を使用（土壌タイプ・温度・pH の変動未反映）
- フィールドデータとの定量的検証が未実施

### 今後の展開 / Future Directions

- SOFIX・NGS フィールドデータとの比較検証
- 温度・pH・土壌タイプ別パラメータの実装
- 土壌生態系 ANT シミュレーターとの統合（水分動態 × 微生物ネットワーク）
- WebGL 対応による大規模粒子シミュレーション
- IoT センサー（土壌水分・CO₂・温度）連携によるデジタルツイン構築

---

## 🔗 関連プロジェクト / Related Projects

- [Soil Ecosystem ANT Simulator](https://github.com/mitsuaki1/Soil-Ecosystem-ANT-Simulator) — アクターネットワーク理論に基づく土壌生態系多種間相互作用シミュレーション
- 醸す谷 note — 有機農業 × 有機土木 × 発酵の実践記録（近日公開）
- Qualium — 土壌生態系センシング・可視化システム（開発中）

---

## 🌿 著者 / Author

**mitsulab**

有機農業 × 有機土木 × 発酵

微生物・水文・生態的相互作用による土地の自己組織化を探究——生命を制御するのではなく、生命が育つ条件を設計する。

Exploring land self-organization through microbiology, hydrology, and ecological interaction — designing the conditions for life to flourish, not controlling life itself.

---

## 📝 ライセンス / License

MIT License — 自由に改変・転用・共有していただけます。  
引用・転載の際は `mitsulab / 醸す谷 Vallis Fermenti` へのクレジットをお願いします。

Free to use, modify, and share. Please credit `mitsulab / 醸す谷 Vallis Fermenti` when citing or reproducing.

---

## 🏷️ キーワード / Keywords

`soil-water` `hydraulic-conductivity` `water-retention` `aggregate-structure` `van-genuchten` `green-ampt` `dual-porosity` `particle-simulation` `soil-physics` `organic-farming` `land-regeneration` `canvas-simulation` `mitsulab` `vallis-fermenti`
