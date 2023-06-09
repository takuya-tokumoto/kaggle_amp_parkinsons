# AMP parkinsonsコンペ
![image](https://github.com/takuya-tokumoto/kaggle_amp_parkinsons/assets/58675697/1758e941-b06d-438f-a7ec-1965c03aed4e)
[(AMP®-Parkinson's Disease Progression Prediction](https://www.kaggle.com/competitions/amp-parkinsons-disease-progression-prediction/overview)

# 結果
最終順位：281位

# 上位解法
## [1位](https://www.kaggle.com/competitions/amp-parkinsons-disease-progression-prediction/discussion/411505)
- サマリ
  - LGB（LightGBM）とNN（Neural Network）の2つのモデルの単純な平均
  - 両モデルは同じ特徴に基づいて学習（NNにはスケーリング/バイナリゼーションが追加されています）：
    - 訪問月（Visit month）
    - 予測期間（Forecast horizon）
    - 目標予測月（Target prediction month）
    - 訪問時に血液が採取されたかどうかの指標（Indicator whether blood was taken during the visit）
    - 補足データセットの指標（Supplementary dataset indicator）
    - 6ヶ月目、18ヶ月目、48ヶ月目に患者の訪問が行われたかどうかの指標（Indicators whether a patient visit occurred on 6th, 18th and 48th month）
    - 前の「非年次」訪問（6ヶ月目または18ヶ月目）の数のカウント（Count of number of previous “non-annual” visits）
    - 目標のインデックス（単一の目標列を持つようにデータセットをピボット）（Index of the target）
  - また、勝つ解決策は血液検査の結果を完全に無視していることが言及されています。血液検査のデータで何かしらの情報を見つけ出そうと試みましたが、どのアプローチやモデルも血液検査の特徴から十分な利益を得ることができず、ランダムな変動から区別することができないと結論づけられました。最終的なモデルは、臨床データと補足データセットの結合のみで訓練されました。
- LGB
  - CV上では、回帰出力を整数に丸めたときに、どのモデルの性能も常に向上することに気付いた
  - 87のターゲットクラス（0～最大ターゲット値）とlogloss目標を持つ分類モデルを構築。ターゲットクラスの予測分布が与えられたら、SMAPE+1を最小化する値を選びとる方式で予測
- NN
  - 回帰予測を目的としたNN
  - NNが負の予測でスタックしないように最終層にleakeyLeluを設けた  
- CV
  -  患者IDによる層化：データセットが小さい場合や、特定の特徴に偏りがある場合、ランダムな分割だけではうまく機能しないことがあります。そのため、彼らは患者IDに基づいて層化したクロスバリデーションを試みています。つまり、同一の患者からのデータが訓練データと検証データに混在しないようにしています。
  - 複数のクロスバリデーションスキームの試行：様々なクロスバリデーションスキームを試し、それらが公開リーダーボードよりも互いに良好に相関していることを確認しました。これは、異なるバリデーションスキームが同じ結論に達するという意味で、モデルのロバスト性を示しています。
  - leave-one-(patient)-out スキーム：最終的に採用したのは、「leave-one-(patient)-out」または「グループk分割交差検証」です。これは、各患者（患者IDごと）に対して一度だけ、その患者のデータをテストデータとする学習・テストプロセスを行う。
  - クロスバリデーションとプライベートリーダーボードの相関：選んだクロスバリデーションスキームがプライベートリーダーボード（最終評価のスコア）と良好に相関していたため、これを採用しました。この一致は、モデルの性能評価が適切であったことを示しています。また、選んだ提出が最終的に最良のプライベートリーダーボードスコアを出したと述べています。
- 試したこと
  -  6ヶ月目の訪問の指標：6ヶ月目に患者が診察を受けたかどうかは、UPDRSの目標値（特にパート2と3）と薬の使用頻度と強く相関しているとされています。この訪問が行われた患者は、平均してUPDRSスコアが高い傾向にあるため、この情報はモデルの学習にとって重要だったようです。同様の傾向が18ヶ月目の訪問にも見られましたが、これら2つの特徴は相関していたため、これらの変数の存在/非存在がモデル間でのパフォーマンスの違い（プライベートリーダーボード上の"cliff effect"）を引き起こしている可能性が示唆されています。
  - 訪問月=0の予測：訪問月が0のときの予測は、12ヶ月後と24ヶ月後の予測よりも6ヶ月後の予測が一貫して低いという興味深い効果が観察されました。これは数学的には理解可能で、6ヶ月目に診察を受ける患者は平均的にUPDRSスコアが高い傾向にあるからです。しかし、臨床的な観点からは、このモデルの挙動は合理的とは言えません。
  - 訓練データとテストデータの違い：訓練データとテストデータの間の違いに注意を払うことも重要でした。例えば、30ヶ月目の訪問を示す特徴を追加すると、クロスバリデーションのスコアは改善しますが、リーダーボードのスコアは低下する可能性があると説明されています。これは訓練データとテストデータの分布が異なるために起こる現象です。

## [4位](https://www.kaggle.com/competitions/amp-parkinsons-disease-progression-prediction/discussion/411398)

- Protein/Peptideデータ
  - Kaggleは227のProtein NXP特徴と968のPeptide PeptideAbundance特徴を提供しました。これは17の可能な訪問日ごとに1195の特徴を意味します。しかし訓練患者は248人しかいません。次元の呪いは特徴の数が訓練サンプルの数/10を超えたときに始まるため、彼は我々は実際には25の特徴を訓練するだけのデータがあるに過ぎないと指摘しています。
- 患者の訪問日からのシグナルを探しました。多くのKagglersが患者の訪問日から特徴をエンジニアリングできることを見落としていたと彼は言います。例えば、患者の最初の血液検査がいつ行われたか、患者が最初の医師訪問で血液検査を受けたかどうか、患者が医師を何回訪れたか、患者の最後の訪問がどれくらい前だったか、などの特徴を作成することができます。
- 特徴エンジニアリング
  - 患者の訪問日にシグナルがあることを示しています。このシグナルを最もよく抽出する方法は何でしょうか？彼は何百もの特徴を生成し、RAPIDS cuML SVRを用いてどの特徴が最も多くのシグナルを抽出するかを見つけるためにfor-loopを使用しました。

## [5位](https://www.kaggle.com/competitions/amp-parkinsons-disease-progression-prediction/discussion/411388)
- 初回非ゼロ来院月数によって、コントロールグループと実際の患者を区別することができる
  - 最初の非ゼロ来院月数が12未満であれば、実際の患者であり、最初の非ゼロ来院月数が12に等しければ、その人は健康な対照群に属している。この区別は、updrsスコアに対して高い予測値を持つ。
- 利用した情報は以下のみ
  - 患者が所属するグループ
  - 予測の月
- これらの特徴量に基づいて、患者がどのグループに属しているかにより、モデルは線形回帰か等温回帰を予測します。
  - 線形回帰（Linear Regression）: 線形回帰は予測変数と目的変数との間に線形関係（すなわち一次関数的な関係）があると仮定する統計的モデリング手法です。この場合、予測は直線（または高次元空間では平面や超平面）で表現されます。
  - 等温回帰（Isotonic Regression）: 等温回帰は目的変数と予測変数間に単調（増加または減少）な関係が存在するという前提のもとで、目的変数を予測するための手法です。この場合、予測は単調な曲線（または高次元空間では曲面）で表現されます。
- 具体的には、ある患者グループでは病状の進行が時間（月）とともに線形に増減すると想定し、別のグループでは病状の進行が単調に（つまり一定の方向に、でも必ずしも一定の速度でではなく）増減すると想定していると解釈できます。どのグループに対してどの回帰手法を用いるかは、おそらく訓練データの解析結果に基づいています。
- つまり、このモデルは患者のグループによって予測の方法を切り替え、線形回帰または等温回帰を用いて病状の進行を予測するというものです。
