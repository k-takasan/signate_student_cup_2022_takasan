# signate_student_cup_2022_takasan

# はじめに
こちらは2022年にSIGNATEにて開催されたコンペティション「SIGNATE Student Cup 2022（予測部門）」の解法および使用コードになります。

URL：https://signate.jp/competitions/735

# タスクの概要
求人情報のテキストデータからその職種（DS, ML Engineer, Software Engineer, Consultant）を分類する。  
評価指標はMacro F1 Scoreとする。

# 解法解説
今回さまざまなアプローチを検証したのですが、結果的に後述のyama09さんの解法が最も高いスコアとなりました。
以下はその実装手順となります。

1. ベースモデルの訓練 (train.ipynb)　
    - EfficientNet-B3をベースに、K-fold法による最適化を行った。 
        - ベースネットワークはCNN系を網羅的に探索して決定した (他にはEfficientNet-v2, SEResNext50が同等であった)。
       
2. 擬似ラベルデータの作成 (pseudo_label.ipynb)
    - 1で最適化したモデルから、テストデータの確度が高いものに対して擬似ラベルを作成した。
    - 作成した擬似ラベルデータを用いてモデルの再学習を行った。
3. モデルアンサンブルによる推論 (inference_ensemble.ipynb)
    - 1,2の手順をEfficientNet-v2, SEResNext50でも行い、合計3つのモデルでアンサンブル(Blending)を行った。

# 有効だった手法

- モデルアンサンブル (Hard Voting)
    - ただし重みの最適化は行っていない。
    - Blendingの他にVotingなども候補になるが試していない。

- Focal Loss
    - 不均衡データに対して有効とされる損失。
    - 今回はCross Entropyとの重み付き平均とした (単体ではあまり効果がなかった)。
- Cosine Annealing
    - 学習率のスケジューリング方法の一つ。
    - ただしスケジューリング方法の最適化は行なっていない。

# あまり有効でなかった手法
- 擬似ラベル
    - コンペでは定番の手法であるが、あまり効果はなかった。
- Label Smoothing
    - 過学習の抑制につながるとされるが、あまり効果はなかった。
- Class weight
    - Cross Entropyにおいて少数クラスの重み付けを行ったが、あまり効果はなかった。

- 再翻訳によるData Augmentation
    - フランス語とドイツ語
- TF-IDF

# 今回試せなかった手法
- Optunaによるハイパラ最適化
- 少数クラス（ML Engineer）への対処
    - ML Engineerのデータが少なく、精度が低くなっていた。
        - DSかそれ以外の2値分類 → 3クラス分類などの工夫をする。
        - ダウンサンプリングを行う。


# 反省点
- BERT系のチューニングやアンサンブル方法をしっかり検証する
- 少数クラスへの対策をさらに検証する

# おわりに
今まで馴染みのなかった自然言語処理に関してのノウハウを学ぶことができました。  
今回の反省点を踏まえて、また新たなコンペにも取り組んでいこうと思います。  
最後になりましたが、関係者各位に感謝申し上げます。

# 参考
- 同コンペ参加者yama09さんの解説記事  
https://qiita.com/yama09/items/9a396e184591b500cb0a
https://qiita.com/yama09/items/2eb188f62cdfb1d23fa0
- 同コンペ参加者colum2131さんの解説記事
https://qiita.com/colum2131/items/b2e5e86f1b98330cb851
- SIGNATE Student Cup 2020 解法まとめ
https://takaito0423.hatenablog.com/entry/2020/09/13/014014