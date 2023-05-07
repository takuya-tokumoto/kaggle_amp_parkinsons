
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
## [AMP® - PDPP - Baseline](https://www.kaggle.com/code/takuyatokumoto/amp-pdpp-baseline/)
* 学習データ'train_clinical_data'のvisit_monthとupdrs_[1-4]の情報からベースラインを作成
* visit_monthの月初から現時点までのupdrs_[1-4]スコア最大値を予測に当てはめる方法
## [[Baseline : 70]| Train + Inference RandomForest](https://www.kaggle.com/code/takuyatokumoto/baseline-70-train-inference-randomforest)
* 学習の縦幅はvisit_month=0のvisit_idベース ➡ ×（visit_month=3の時のスコアを特徴量にしてるっぽい）
* 特徴量はprotein, peprtideの存在量をvisit_id単位でgroupbyして統計量(平均、分散..)を利用
* random forestモデルでgrit serch機能を利用。
* updrs_[1-4]別に4通りのモデルを作成
## [EDA+LinearRegression](https://www.kaggle.com/code/takuyatokumoto/eda-linearregression)
* proteinとpeptideの存在量をprotein/peptideID単位で横持ちに変換し、特徴量として作成
* モデルはLinearRegressionとxggregressのアンサンブル
* updrs_[1-4]別に4通りのモデルを作成
* max_depth=3とかなり浅い構造
* score 58.4

## [Simple Linear model with only clinical data](https://www.kaggle.com/code/takuyatokumoto/simple-linear-model-with-only-clinical-data)
* テストでは薬の服用状況が確認できないので？（要確認）CVも服用状況をunkownに置換して評価
* visit_monthとupd23b_clinical_state_on_medication軸でgroupbyして特徴量を加工
* LinearSVR/PoissonRegressor/SVR/LinearRegression/LGBMRegressorをアンサンブルしたモデルを利用
* コードがごちゃついているのが気になる。モデル構造はシンプルであるが予測性能は良い。特徴量エンジニアリングがメインのコンペの傾向が見える

## [Using Feature selection, XGBoost, Trend](https://www.kaggle.com/code/takuyatokumoto/using-feature-selection-xgboost-trend)
https://www.kaggle.com/code/takuyatokumoto/using-feature-selection-xgboost-trend/edit
* ベースラインに流用

## [Only Trends](https://www.kaggle.com/code/takuyatokumoto/only-trends)
* 最適化手法によるLBスコアの改善をトライ
* template_v2との差分点は `if target == 'updrs_4': pred_month = pred_month.clip(54, None)`で54でクリップしているか？
* なぜ54なのかは解説されていない

## [PDPP Quadratic Trends](https://www.kaggle.com/code/ambrosm/pdpp-quadratic-trends)
* 

## [Explain Dataset, Test API, Cross-Validation Tips](https://www.kaggle.com/code/takuyatokumoto/explain-dataset-test-api-cross-validation-tips)
* 
* 

## [Protein NPX Groups + Trend](https://www.kaggle.com/code/vitalykudelya/protein-npx-groups-trend/notebook)
* タンパク質のNPX値をいくつかのグループに分け、各グループの月次トレンド予測から最適なシフトを求める。
* 月次推移と対応するNPXグループシフトからの予測値の合計

# Discussion
## [Dealing with visits of a patient with NaN proteins](https://www.kaggle.com/competitions/amp-parkinsons-disease-progression-prediction/discussion/403073)
* 同じ患者のタンパク質やペプチド発現の将来の値が欠落した場合、どのように対処するのが適切でしょうか？(例えば同じ患者（例：Patient_id=55）に対して、ある訪問月（例：visit_month=0）にタンパク質のNPX値があり、その次（visit_month=3）にNPXのNaNがある場合)
* ➡これといった正解はない。タンパク質やペプチドの表現が可能な後続の月があれば、fill forwardかbackfillのどちらか、もしくは両月間の中央値を求めるという手もあります。

## [Patient-level Time Series Feature Engineering](https://www.kaggle.com/competitions/amp-parkinsons-disease-progression-prediction/discussion/388521)

時系列における特徴量エンジニアリングの方法  
このコンペでは、1人あたりの時系列測定値（経時的な測定値）を取得します。  
単純な機能としては、差分ベースの機能（時間の経過に伴う値の変化）をチェックするとよいかもしれません。  

### Group stats features  
* この特徴量加工は、時系列変数を時間の経過とともに集計する必要がある変換を行うためのもの。
* グループ（この場合、患者）に沿った傾向を見つけるのに便利です。

```python
keys = ... 
val = ...
df.groupby(keys).transform(lambda x: x.mean())
df.groupby(keys).transform(lambda x: x.median())
df.groupby(keys).transform(lambda x: x.std())
df.groupby(keys).transform(lambda x: x.count())
df.groupby(keys).transform(lambda x: x.sum())
```
### Lag features
* 時系列の値を一定の時間単位でシフトさせることに基づいています。
* モデルに情報を追加して、より良い予測をするのに役立ちます。特に、対象変数のラグ値を特徴量として投入し、予測したい場合に有効です。

```python
keys = ...
val = ...
lag = 1
df.groupby(keys)[val].transform(lambda x: x.shift(lag))
```

### Rolling Window Stats features
* この特徴量加工は、ローリングウィンドウを使用した経時的な集計を必要とする変換で、時系列変数を変換するためのもの。
* 時系列におけるトレンドや季節性を見つけるのに便利
```python
keys = ...
val = ...
window = 7
df.groupby(keys)[val].transform(lambda x: x.rolling(window=window, min_periods=3, win_type="triang").mean())
df.groupby(keys)[val].transform(lambda x: x.rolling(window=window, min_periods=3).std())
```

### Expanding stats features
* 累積平均および累積標準偏差を計算
* ここでのexpanding(2) は、2番目のデータポイントから累積計算を開始することを意味します。
```python
keys = ... 
val = ...
df.groupby(keys)[val].transform(lambda x: x.expanding(2).mean())
df.groupby(keys)[val].transform(lambda x: x.expanding(2).std())
```

### Trend features
* グループごとに差分（隣接する要素間の差）を計算し特徴量を作成する。
```python
keys = ... 
val = ...
df.groupby(keys)[val].transform(lambda x: x.diff())
```

### Exponentially Weighted Average
* 過去の観測値に対して指数関数的に減衰する重みを付けて平均を計算します。
* 最新のデータポイントには高い重みが付けられ、古いデータポイントには低い重みが付けられるため、EWAは時系列データの最近の動きをより強調します。
```python
keys = ... 
val = ...
lag = 1
alpha=0.95
df_temp.groupby(keys)[val].transform(lambda x: x.shift(lag).ewm(alpha=alpha).mean())
```

## [Peptide sequence that may help in detecting PD (from internet)](https://www.kaggle.com/competitions/amp-parkinsons-disease-progression-prediction/discussion/394983)

### Alpha-synuclein
* α-シヌクレインは、脳内に多く存在するペプチドで、パーキンソン病の特徴的な病理学的特徴であるレビー小体の主要構成要素である。
* α-シヌクレインをコードするSNCA遺伝子の変異は、家族性のパーキンソン病と関連しています。

### Beta-amyloid
* ベータアミロイドは、アルツハイマー病に最もよく関連するペプチドですが、パーキンソン病の人の脳内でも発見されています。
* βアミロイドは、脳内の神経炎症と酸化ストレスに関与していると考えられており、これらはいずれも神経変性につながる可能性があります。

### Urocortin
* ウロコルチンは、パーキンソン病の動物モデルにおいて、神経保護作用を有することが確認されたペプチドです。
* ウロコルチンは、パーキンソン病で変性する細胞であるドーパミン作動性ニューロンの炎症、酸化ストレス、アポトーシス（プログラムされた細胞死）を抑えることができます。

### Glucagon-like peptide-1 (GLP-1)
* GLP-1は、グルコース調節に関与するペプチドホルモンで、パーキンソン病の動物モデルにおいて神経保護作用があることが判明しています。
* GLP-1は、ミトコンドリア機能を改善し、ドーパミン作動性ニューロンの炎症を抑えることで、神経変性を遅らせたり、予防したりする効果が期待されます。

`これらのトレーニングデータでは、完全な配列は利用できないようです。配列の一部を用いて特徴量を構築することができる`

