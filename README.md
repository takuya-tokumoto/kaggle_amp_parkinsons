# AMP parkinsonsコンペ
![image](https://github.com/takuya-tokumoto/kaggle_amp_parkinsons/assets/58675697/1758e941-b06d-438f-a7ec-1965c03aed4e)
[(AMP®-Parkinson's Disease Progression Prediction](https://www.kaggle.com/competitions/amp-parkinsons-disease-progression-prediction/overview)

# 結果
最終順位：281位

# 上位解法
## [1位](https://www.kaggle.com/competitions/amp-parkinsons-disease-progression-prediction/discussion/411505)
- モデルはLGBとNNの組み合わせ
- 血液検査の結果を完全に無視ランダムな変動と区別できるほど重要な血液検査の特徴から利益を得ることはできないという結論に達しました。
- 最終的なモデルは、臨床データと補助的なデータセットの組み合わせでのみで学習
- モデル
  - LGB
    - CV上では、回帰出力を整数に丸めたときに、どのモデルの性能も常に向上することに気付いた
    - 87のターゲットクラス（0～最大ターゲット値）とlogloss目標を持つ分類モデルを構築。ターゲットクラスの予測分布が与えられたら、SMAPE+1を最小化する値を選びとる
  - NN
