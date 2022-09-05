# signate_student_cup_2022_takasan

# はじめに
こちらは2022年にSIGNATEにて開催されたコンペティション「SIGNATE Student Cup 2022（予測部門）」の解法および使用コードになります。

URL：https://signate.jp/competitions/735

# タスクの概要
日本画のデータセット「KaoKore」を使って、各画像から登場人物の身分を分類する（４クラス分類）。  
評価指標はMacro F1 Scoreとする。

<p align="center">
  <img src="images/label_example.png" width='700'>

  <br>
  データセットの画像とラベルの例。今回は身分のみの分類となる。
</p>



# 解法解説
1. ベースモデルの訓練 (train.ipynb)　
    - EfficientNet-B3をベースに、K-fold法による最適化を行った。 
        - ベースネットワークはCNN系を網羅的に探索して決定した (他にはEfficientNet-v2, SEResNext50が同等であった)。
        - 最適化にはAugmentationの手法が有効であった。
2. 擬似ラベルデータの作成 (pseudo_label.ipynb)
    - 1で最適化したモデルから、テストデータの確度が高いものに対して擬似ラベルを作成した。
    - 作成した擬似ラベルデータを用いてモデルの再学習を行った。
3. モデルアンサンブルによる推論 (inference_ensemble.ipynb)
    - 1,2の手順をEfficientNet-v2, SEResNext50でも行い、合計3つのモデルでアンサンブル(Blending)を行った。

# 有効だった手法
- 擬似ラベル
- モデルアンサンブル (Blending)
    - ただし重みの最適化は行っていない。
    - Blendingの他にVotingなども候補になるが試していない。
- Trivial Augment
    - 1つのパラメータのみ (画像加工の強さmの最大値) でData Augmentationを行うことができる手法。
    - ミニバッチごとにランダムに画像加工の種類とその強さを選択する。
- Rand Augment
    - 2つのパラメータでData Augmentationを行う。
    - 使用する画像加工の種類nとその強さmを指定する。
    - ミニバッチごとにランダムに画像加工の種類を選択する (mは固定)。
- Focal Loss
    - 不均衡データに対して有効とされる損失。
    - 今回はCross Entropyとの重み付き平均とした (単体ではあまり効果がなかった)。
- Cosine Annealing
    - 学習率のスケジューリング方法の一つ。
    - ただしスケジューリング方法の最適化は行なっていない。

# あまり有効でなかった手法
- Label Smoothing
    - 過学習の抑制につながるとされるが、あまり効果はなかった。
- Class weight
    - Cross Entropyにおいて少数クラスの重み付けを行なったが、あまり効果はなかった。
- Fine tuning
    - 学習済みモデルの前段をフリーズさせて学習を行ったが、あまり効果はなかった。

# 今回試せなかった手法
- Vision Transformer, Swin Transformer
    - 今回上位入賞者が使用していた。
- 深層距離学習
    - 検討はしたが実装できなかった。
    - 顔認識などに用いられる。
- Optunaによるハイパラ最適化
- AdamW
    - Adamの改良版。
- 少数クラス（平民）への対処
    - 平民のデータが少なく、精度が低くなっていた。
        - 貴族かそれ以外の2値分類 → 3クラス分類などの工夫をする。
        - GANによるData Augmentationを行う。
        - ダウンサンプリングを行う。
- ノイズとなるデータの除去
    - 顔以外の画像などがノイズとなっていた？
- TTA (Test-Time Augmentation)
    - 推論時にもAugmentationを行う手法。

# 反省点
- 実験管理をきちんとする
    - 序盤は場当たり的に行っていたので、終盤に何をすべきかわかりにくくなってしまった。
- LBに最適化しすぎない（LBスコアとCVスコアと照らし合わせる）
    - Public 11位 → Private 22位となってしまった。

# おわりに
画像分類やコンペ一般に関してのノウハウを学ぶことができました。  
今回の反省点を踏まえて、また新たなコンペにも取り組んでいこうと思います。  
最後になりましたが、チームメンバーと関係者各位に感謝申し上げます。

# 参考
- 1st place solution  
https://github.com/bysk2/probspace_kaokore_status_1st_solution
- 1st place solution @Nishikaでの類似コンペ  
https://github.com/3017218062/Ancient-Portrait-Classification#5