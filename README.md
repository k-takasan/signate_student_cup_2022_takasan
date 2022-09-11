# signate_student_cup_2022_takasan

# はじめに
こちらは2022年にSIGNATEにて開催されたコンペティション「SIGNATE Student Cup 2022（予測部門）」の解法および使用コードになります。

URL：https://signate.jp/competitions/735

# タスクの概要
求人情報のテキストデータからその職種（DS, ML Engineer, Software Engineer, Consultant）を分類する。  
評価指標はMacro F1 Scoreとする。

# 解法解説
今回さまざまなアプローチを検証したのですが、結果的に下段URLのyama09さんの解法が最も高いスコアとなりました。
以下はその実装手順となります。

1. ベースモデルの訓練 (train.ipynb)　
    - BERTをベースに、K-fold法による最適化を行った。 
2. モデルアンサンブルによる推論 (train_inference_ensemble.ipynb)
    - 1と同様にRoberta, Debertaの学習を行い、合計3つのモデルでアンサンブル（Voting）を行った。

# 有効だった手法

- BERT系のNNによる分類
    - BERTの派生であるRobertaやDebertaは同等の精度が得られた。
- モデルアンサンブル (Hard Voting)
    - Votingの他にBlendingやStackingなども候補になるが試していない。

# あまり有効でなかった手法
- 擬似ラベル
    - テストデータの推論結果のうち、確信度が高いサンプルに対してラベルを付与し、そのサンプルも訓練データに含めるという手法。
    - コンペでは定番の手法であるが、あまり効果はなかった。
- Label Smoothing
    - 過学習の抑制につながるとされるが、あまり効果はなかった。
- Class weight
    - Cross Entropyにおいて少数クラスの重み付けを行ったところ、CVは上がったが、LBは下がった。
- 学習率のスケジューリング
    - 一通り試したわけではないが、Warm upはあまり効かなかった。
- 再翻訳によるData Augmentation
    - フランス語とドイツ語による再翻訳を行い、学習データを増やしたが、あまり効果はなかった。
- TF-IDFやBoWによる特徴量抽出からのGBDTによる分類
    - NNが主流になる前の文書分類手法。
    - NN系の分類器と比べて大幅に精度が低かった。

# 今回試せなかった手法
- Optunaによるハイパラ最適化
    - バッチサイズやMax lengthのほか、Schedulerの最適化も含める。
- 少数クラス（ML Engineer）への対処
    - ML Engineerのデータが少なく、精度が低くなっていた。
        - DSかそれ以外の2値分類 → 3クラス分類などの工夫をする。
        - ダウンサンプリングを行う。
- Focal Loss
    - 不均衡データに対して有効とされる損失。
- Stacking, Blendingなどのアンサンブル手法

# 反省点
- BERT系のチューニングやアンサンブル手法をしっかり検証する
    - ハイパーパラメータやモデルの重み付けにまだまだ改善の余地があったと思われる。
- 少数クラスへの対策を講じる
    - 今回決定的な対処法は見つからなかった。
- より多くの＆多様なモデルを組み合わせる
    - 上位入賞者は10以上のモデルのアンサンブルをしていたと思われる。
        - 優勝者は同じネットワークでもSeedを変えたものをアンサンブルに取り入れていた。

# おわりに
今まで馴染みのなかった自然言語処理に関してのノウハウを学ぶことができました（まだ不十分ですが）。  
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