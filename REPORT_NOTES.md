# レポート作成メモ（実データに基づく結果まとめ）

LaTeX を書くときに参照するための、確定済みの数値・結論・使い方のメモ。
すべて `ai-interview-coach/eval/` の解析スクリプト出力に基づく。

## 作成したファイル

| ファイル | 用途 |
|---|---|
| `results_section.tex` | Results + Discussion セクション丸ごと。`main.tex` で `\input{results_section}` するか、内容をコピー。 |
| `revised_snippets.tex` | Abstract / Conclusion / Methodology の修正版テキスト（コピペ用、コメント内）。 |
| `REPORT_NOTES.md` | このメモ。 |

## ⚠️ 最重要：現 main.tex は実データと矛盾している
- Abstract が "significantly improves" と書いているが、**有意差はない**。
- Table 1 の baseline が 14.7 など、**実際の値（16.86）と違う**プレースホルダ。
- DPO の結果が `[Placeholder]` のまま。

→ `results_section.tex` で Results 全体を差し替え、`revised_snippets.tex` で Abstract / Conclusion を差し替える。

## 確定した結果の数値（1〜20スケール、N=200）

| 指標 | Baseline (SD) | SFT+DPO (SD) | Δ | 対t (p) | Wilcoxon p | Cohen's d |
|---|---|---|---|---|---|---|
| Technical Correctness | 16.79 (2.40) | 16.92 (1.92) | +0.14 | 0.63 (0.53) | 0.73 | 0.044 |
| Specificity | 16.88 (2.37) | 17.06 (1.82) | +0.19 | 0.88 (0.38) | 0.65 | 0.062 |
| Helpfulness | 16.88 (2.37) | 17.05 (1.85) | +0.18 | 0.83 (0.41) | 0.69 | 0.058 |
| Actionability | 16.89 (2.38) | 17.05 (1.84) | +0.17 | 0.78 (0.44) | 0.81 | 0.055 |
| Interview Coaching Quality | 16.88 (2.37) | 17.06 (1.82) | +0.19 | 0.88 (0.38) | 0.69 | 0.062 |
| **Overall** | **16.86 (2.37)** | **17.03 (1.85)** | **+0.17** | **0.81 (0.42)** | **0.70** | **0.057** |

- Win/Loss/Tie（overall）：**new 48 / baseline 40 / tie 112**、符号検定 p=0.46。
- **どの指標も α=0.05 で有意でない。効果量はすべて negligible (d<0.07)。**

## 結論（レポートの主張）
**「有意な改善は得られなかった（ネガティブ結果）」** を正直に主張し、その理由を分析する論文構成にする。

理由3点（Discussion）：
1. **天井効果** — baseline の 65% が既に ≥18/20、191/200 件で両モデル ≥15。伸びしろがない。
2. **判定モデルの解像度不足** — スコアが 15 と 18 に二極集中、5観点がほぼ同値。ノイズが大きい。
3. **改善と退行の相殺** — 改善48件 vs 退行40件。
   - 大勝ち：`test_0001/0073/0119` が 1→18（baseline の幻覚を修正）。
   - 大負け：`test_0194` 18→1（`@Override` が実行時 `java.lang.Error` を投げる、という虚偽）、`test_0196` 18→10、`test_0103` 18→12.8。
   - 退行40件のうち技術的誤りは4件、残りは判定ノイズ範囲の軽微なもの。

## データセットの限界（Limitations に必須）
- **test/train 全件が `student_answer_type = partially_correct`**。方法論で謳う5タイプが実データに無い。
- **category / difficulty が 189/200 件 `Unknown`**（difficulty: Medium 7, Hard 4 のみ）。
- → 評価が入力分布の狭い1スライスしか見ていない。天井効果の一因でもある。
- 発生源ファイル：`ai-interview-coach/data/test.jsonl`（および `data/train.jsonl`）。

## 数値の再生成方法（データを更新した場合）
```bash
cd ai-interview-coach
python eval/compare_scores.py        # 平均・Δ・勝敗 -> eval/comparison_summary.json
python eval/statistical_tests.py     # t検定/Wilcoxon/符号検定/Cohen d -> eval/statistical_tests.json
python eval/failure_analysis.py      # 退行/改善の分類・代表例 -> eval/failure_*.{json,csv}
```
退行・改善の代表例の本文（フィードバック全文＋判定理由）は
`eval/failure_regressions.csv` / `eval/failure_gains.csv` から引用できる。

## 図表の作成候補（任意）
- スコア分布のヒストグラム（baseline vs new、15と18への集中＝天井効果が視覚的に伝わる）。
  - baseline 分布: 1:3, 14:2, 15:44, 16:16, 17:5, 18:127, 19:3
  - new 分布: 1:1, 10:1, 13:1, 14:1, 15:42, 16:18, 18:130, 19:6
- 退行/改善の要因カテゴリの棒グラフ。

## 引用について
現 `egbib.bib` には RLHF / RLAIF / DPO / 教育応用 / LLM judge の引用が既にある。
「LLM judge が飽和して評価器として弱い」「LLM-judge と人手選好の整合」の議論には
既存の `shankar2024validators` を流用できる。
