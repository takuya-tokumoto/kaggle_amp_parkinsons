
# はじめに
## 目的
* パーキンソン病患者さんの進行度を測るMDS-UPDRスコアを予測することが目的
* Movement Disorder Society-Sponsored Revision of the Unified Parkinson's Disease Rating Scale（MDS-UPDRS）は、パーキンソン病に関連する運動症状と非運動症状の両方を総合的に評価するもの
* パーキンソン病の被験者と年齢をマッチさせた正常な対照被験者のタンパク質およびペプチドレベルの経時的なデータから学習させたモデルを作成することがミッション

## 提出物について
### 評価方法
#### 評価関数
> Symmetric mean absolute percentage error(SMAPE)  

![image](https://user-images.githubusercontent.com/58675697/235550613-dd5066b6-5718-4e18-af84-330ba4cd94e9.png)

`AtとFtの差の絶対値は、実際の値Atと予測値Ftの絶対値の和の半分で割られる。この計算の値は、すべての適合点tについて合計され、適合点の数nで再び割られる。`
* スケールに依存しない: SMAPEは、予測誤差の相対的な大きさを評価するため、予測対象のスケールや単位に依存しない評価指標です。これにより、異なるスケールの時系列データ間で予測性能を比較することが容易になります。
* 対称性: SMAPEは、過大予測と過小予測に対して同等のペナルティを与えます。このため、予測モデルが過大予測に傾くか過小予測に傾くかを区別せず、予測誤差全体の大きさを評価することができます。
* 0%から100%の範囲: SMAPEの値は0%から100%の範囲で評価され、0%は完全な予測精度を示し、100%は最も低い予測精度を示します。この範囲により、予測精度を直感的に理解しやすくなります。
* 実際の値が0に非常に近い場合、SMAPEは不安定になり、誤差が過大評価されることがあります。
* SMAPEは予測誤差の相対的な大きさを評価するため、絶対的な誤差を評価したい場合は他の指標（例：Mean Absolute Error）を使用することが望ましいです。

### 観点
* タンパク質/ペプチドサンプルが採取された各患者の来院時に、その来院時のUPDRSスコアと、6ヶ月後、12ヶ月後、24ヶ月後の来院時のスコアを予測する必要があります。
* 最終的に実施されなかった診察の予測は無視されます。
* pythonの時系列APIを使用して提出する必要があり

# EDA
## 参照
* https://www.kaggle.com/code/craigmthomas/amp-eda-models
* https://www.kaggle.com/code/takuyatokumoto/amp-eda-models/edit
# ベースラインの構築
## ①
https://www.kaggle.com/code/takuyatokumoto/amp-pdpp-baseline/edit
* 学習データ'train_clinical_data'のvisit_monthとupdrs_[1-4]の情報からベースラインを作成
* visit_monthの月初から現時点までのupdrs_[1-4]スコア最大値を予測に当てはめる方法
## ②
https://www.kaggle.com/code/takuyatokumoto/baseline-70-train-inference-randomforest
* 学習の縦幅はvisit_month=0のvisit_idベース ➡ ×（visit_month=3の時のスコアを特徴量にしてるっぽい）
* 特徴量はprotein, peprtideの存在量をvisit_id単位でgroupbyして統計量(平均、分散..)を利用
* random forestモデルでgrit serch機能を利用。
* updrs_[1-4]別に4通りのモデルを作成
## ③
https://www.kaggle.com/code/takuyatokumoto/eda-linearregression/edit
* proteinとpeptideの存在量をprotein/peptideID単位で横持ちに変換し、特徴量として作成
* モデルはLinearRegressionとxggregressのアンサンブル
* updrs_[1-4]別に4通りのモデルを作成
* max_depth=3とかなり浅い構造
* score 58.4
## ④
https://www.kaggle.com/code/takuyatokumoto/simple-linear-model-with-only-clinical-data/edit
* テストでは薬の服用状況が確認できないので？（要確認）CVも服用状況をunkownに置換して評価
* visit_monthとupd23b_clinical_state_on_medication軸でgroupbyして特徴量を加工
* LinearSVR/PoissonRegressor/SVR/LinearRegression/LGBMRegressorをアンサンブルしたモデルを利用
* コードがごちゃついているのが気になる。モデル構造はシンプルであるが予測性能は良い。特徴量エンジニアリングがメインのコンペの傾向が見える
## ⑤
https://www.kaggle.com/code/takuyatokumoto/using-feature-selection-xgboost-trend/edit
