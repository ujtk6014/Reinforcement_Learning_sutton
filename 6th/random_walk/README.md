# 6th/random walk
このフォルダでは強化学習のTD学習の基本について説明します

# Outline
1. TD学習とは
2. ランダムウォークの問題

# TD学習とは
前回、前々回と、動的計画法、モンテカルロ法を学んできましたがどちらにも問題点がありました

- 動的計画法

環境のモデルが必要

- モンテカルロ法

ランダムに，すべての状態をたくさん経験する必要がある

画像でみるとこんな感じですね

![image.png](https://qiita-image-store.s3.amazonaws.com/0/261584/3ce68a5a-7fbc-780a-ff03-3de6e9f98150.png)

ちなみに動的計画法はすべての状態に対して予測して価値を計算していくので完全バックアップと言われます．
この話は本の最後あたりにでてきます
つまり，環境のモデルがあるのでそこからはじめようとなにしようと関係ないわけです

しかし，モデルがない場合モンテカルロ法のようにエピソード単位で学習を行う必要があります．

エピソード単位かどうか，実際のシナリオを考えるかどうかが大きな違いとも言えます

## モチベーション
ここで，TD学習のモチベーションは，

- 環境のモデルを使いたくない
- でもエピソードがすべて終わるまで待ちたくない

という2つのモチベーションです．
なので式としてはこんな感じになります

<img src="https://latex.codecogs.com/gif.latex?V(s_t)&space;&plus;&space;\alpha[r_{t&plus;1}&space;&plus;&space;\gamma&space;V(s_{t&plus;1})&space;-&space;V(s_t)]">

さて，このままではよくわからないので
モンテカルロ法を価値関数に置き換えた場合と比較してみます
モンテカルロ法は今まで行動価値関数Qでいろいろと考えていましたが，ここでは価値関数でいきしょう（本質的には変わりません）
（その状態の価値か，その状態で行動aをとった価値なのか，どちらを評価するのかの違いです）

![image.png](https://qiita-image-store.s3.amazonaws.com/0/261584/3d5af2f6-7d3e-81c4-1d5c-ef5b79b9a74a.png)

上に示したようにモンテカルロ法はエピソード終了後の確定収益で更新します．
一方で，TD法は動的計画法を融合させ，その後の状態価値でも更新します．

この違いを図で確認します

![image.png](https://qiita-image-store.s3.amazonaws.com/0/261584/df531920-e2de-929e-e1cf-befdc378c963.png)

この図をみると一目瞭然かと思います．
ちなみに動的計画法のようにすべての状態をバックアップするわけではないことに注意してください
つまり，モデルはないので，動的計画法（完全バックアップ）のようにすべての状態に対してこれはできません

**基本的にはエピソードの中での話であるということを軸においておきます！**
（サンプルバックアップです）

なのでどちらかというとモンテカルロ法的な発想です

まとめると

### モンテカルロ法のように確定収益ではなく，報酬r+次状態の予測状態価値で更新をしていくということです．

では違いをどう考えればよいでしょうか

教科書の例がすごく分かりやすかったので紹介します
（この例では，バッチ更新と呼ばれる手法を使っています（更新分はすべての学習データの増加分をひとまとめで更新するという手法です））

![image.png](https://qiita-image-store.s3.amazonaws.com/0/261584/f5f091d5-42c7-7726-b8a1-17bff914ccdd.png)

このような条件（もちろんこのモデルはないです，分かりやすいように描画しているだけです！）

この時以下のような，状態が観測されたとします．

A:0⇒B:0　（状態AからスタートBにいって報酬は0）
B:1（状態Bから報酬は1）
B:1
B:1
B:1
B:1
B:1
B:0


まず，モンテカルロ的に考えてみましょう

状態Aにいて，Bにいって，報酬が0でした
そして，それ以外で状態Aにはいってないので，状態Aの価値は0になります

一方！

TDの場合
Bが状態価値3/4になりますね（8回中6回成功）
でしかも，A⇒Bの状態遷移が100％おきてます
よってAの価値も3/4になるというわけです

つまりTDはマルコフ過程でのもっともよい推定値（1ステップ前にのみ状態が依存する）を求めていて

モンテカルロ法は訓練データでのもっともよい推定値を求めていると言えます

どっちがいいのよという問題ですが
マルコフ過程をおくのであれば，TDのほうがよさそうです
過去の1ステップで未来を表現できそうなこの状態では．．
なぜなら，未来のデータに対して有効だからです

ただ，モンテカルロは現在のデータへの平均を取れます
なので今までの結果をより強く反映させる場合はモンテカルロの方がよいかと．．．

はっきりとはわかりませんが

では教科書の例にあげられているランダムウォークの問題をやってみましょう．

# ランダムウォークの問題
これは以下のようなノードがあり，右端にいくと報酬が1
左端にいくと報酬が0です

![image.png](https://qiita-image-store.s3.amazonaws.com/0/261584/c79b0624-e86a-748a-d383-bdfda07d9bdf.png)

このとき，ランダム方策での状態価値関数を求めてくださいという問題
ランダム方策とは，右に行くか左にいくかをランダムにとるということです

先ほどの更新式を使えば簡単に実装できます
ちなみに真値は確率になるので
左から

A:1/6
B:2/6
C:3/6
D:4/6
E:5/6

になります

# ランダムウォークの問題(Usage)

## TD法の推定価値

```
$ python random_walk_1.py
```

## TDとモンテカルロの比較

```
$ python random_walk_2.py
```

# ランダムウォークの問題(結果)

まずTD法で学習させた後の結果をみてみましょう


![Figure_1.png](https://qiita-image-store.s3.amazonaws.com/0/261584/7f4535d3-ea4e-3910-a407-f1a68b6e1f08.png)

になります
教科書通りですね

さて，各状態での予測誤差をとったものをモンテカルロ法と比較します
ちなみに，たくさん学習回すと，どちらも最後は同じ結果になります
モンテカルロ法の状態価値推定は上の式にのっとってます
（今回は行動価値を求める問題ではないのでそこはご注意ください）

![Figure_1.png](https://qiita-image-store.s3.amazonaws.com/0/261584/cd8622b9-2b97-88d4-9624-849a0fb9be1b.png)


どちらがいいとかそういう話ではないそうですが．．．
確率的なタスクについては

データの平均を取るモンテカルロ法よりも

次の状態遷移などを考えるTDの方が収束がはやいことが証明されています