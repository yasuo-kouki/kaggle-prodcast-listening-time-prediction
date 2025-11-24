# Kaggle Playground Series: Podcast Listening Time Prediction

ポッドキャスト番組の総視聴時間を予測する Playground コンペ（回帰, RMSLE 指標）の実験リポジトリです。  
XGBoost/LightGBM/CatBoost/RandomForest を中心に、外れ値処理とアンサンブルでスコアを更新しました。

## プロジェクト概要

- ターゲット: `Listening_Time`（分単位, 連続値）
- 指標: Root Mean Squared Log Error (RMSLE)
- 主な成果: 木系モデルを順に改善し、スタッキング/単純平均のアンサンブルで自己ベスト（12.86040）を記録

## ディレクトリ構成

```
kaggle-prodcast-listening-time-prediction/
├─ README.md
├─ data/                # Kaggle 配布 CSV（要ダウンロード）
├─ cat_project.ipynb    # CatBoost 実験
├─ lightgbm.ipynb
├─ radom_forest.ipynb
├─ xgb_project.ipynb
├─ hybrid_project.ipynb # スタッキング/重み最適化
├─ model/               # 保存済みモデル（cbm/json/pkl 等）
├─ catboost_info/       # CatBoost 学習ログ
├─ submit/              # 提出履歴 CSV
└─ reult.xlsx           # ローカル検証ログ
```

## セットアップ

1. Kaggle から `train.csv`, `test.csv`, `sample_submission.csv` を `data/` に配置
2. `jupyter lab` で各 Notebook を開き、EDA → 単体モデル → アンサンブルの順に実行
3. `submit/` に生成された CSV を Kaggle に提出

## 開発ログとスコア

| # | モデル | Public LB (RMSLE) | 備考 |
| --- | --- | --- | --- |
| 1 | XGBoost v1 | 12.88959 | 欠損平均補完 + 学習率 0.1 |
| 2 | XGBoost v2 | 12.83601 | 外れ値カット + 学習率微調整 |
| 3 | CatBoost | 13.23707 | デフォルトパラメータ, 今後改善余地 |
| 4 | Ensemble (XGB + LGBM + RF + Cat) | **12.86040** | 重み平均・スタッキング両方を検証 |

## 特徴量・前処理メモ

- `Podcast_Name`, `Episode_Title`: 極端なレアカテゴリは見られず、Label Encoding で対応
- `Episode_Length_minutes`: 150 分以上の長時間サンプルが外れ値として存在（train/test 双方に散見）
- `Host_Popularity_percentage`, `Guest_Popularity_percentage`: 0 や 100 超が点在するためクリッピング + log1p で安定化
- `Number_of_Ads`: 長い裾を持つため `min-max` ではなく `quantile clipping` を採用
- `Episode_Sentiment`, `Genre`, `Publication_Day`: train/test の分布差は小さく追加処理なし

## 進め方の目安

1. `consider_data.ipynb` でデータ概要と異常値を確認
2. `xgb_project.ipynb` / `lightgbm.ipynb` / `cat_project.ipynb` で単体モデルを検証
3. 予測結果を `submit/` に保存し、`hybrid_project.ipynb` で重み最適化や stacking ridge (`model/meta_model/stacking_ridge.pkl`) を再学習
4. 最終的な提出ファイルは `submit/hybrid/` に格納

## 今後のTODO

- CatBoost のパラメータチューニング（depth, l2_leaf_reg, border_count）
- LightGBM の monotone constraints / categorical_feature 指定
- Stacking 第二層に Lasso/ElasticNet を導入し汎化性能を比較
# kaggle-prodcast-listening-time-prediction
