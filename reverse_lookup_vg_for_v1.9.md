# 逆引きvgコマンド

## 目的

`vg` のチュートリアルはいろいろある([本家のwiki](https://github.com/vgteam/vg/wiki/Basic-Operations)や[ポルトガルでの講習会](https://github.com/Pfern/PANGenomics))が、「〇〇をしたいときにどんなコマンドを使えばいいのかわからない」ときに、これらの資料を探すのはすごくしんどい。そこで、やりたいことベースで `vg` コマンドの使い方を探せるように、コマンドを整理してみた。  

バージョンは `v1.9.0 "Miglionico"` とする。static binary または docker imageは[ここ](https://github.com/vgteam/vg/releases/tag/v1.9.0) から入手可能。



## グラフを作る

### 配列からのグラフ構築

#### リファレンスゲノムとvaliant callの情報をグラフにする

```
vg construct -r ref.fa -v valiant.vcf > graph.vg
```



#### 複数の配列をマルチプルアラインメントして、グラフにする

```
vg msga -f multi.fa > graph.vg
```



## フォーマット変換

#### vgの中身を確認する

```
vg view graph.vg > graph.gfa  # vgをGFAに変換する
vg view -j graph.vg > graph.json  # vgをJSONに変換する
```

- JSONに変換すると、いわゆるゲノムグラフブラウザで可視化できる。
  - 例：[MoMIG](http://viewer.momig.tokyo/demo3/#force_layout=false&sankey=false&path=chr12:80,851,974-80,853,202)
    - ただし、パスの情報がないと可視化できない



#### GFAをvgに変換する

```
vg view -Fv hoge.gfa > graph.vg
```

- [v1.9.0 でオーバーラップつきGFAパーサーのバグがとれた(らしい)](https://github.com/vgteam/vg/pull/1765)
- アセンブリグラフを下流の解析に用いるときはこれ
  - アセンブラとして `minia` を使うなら、[ここ](https://github.com/Pfern/PANGenomics/blob/5923c991962396f30ce8adef9eef4d0a1ecd68b8/exercises/bacteria/README.md#gfa-input-to-vg-from-minia-and-bcalm)を参照



#### vgをGraphMLで簡単に可視化

```
vg view -d graph.vg | dot -Tpng -o vis.png  # vgをdot形式にする
vg view -dnp graph.vg | dot -Tpng -o vis.png  # 各パスがグラフのどこを通るのかを出す
```



#### vgをxgに変換する

```
vg index -x index.xg graph.vg
```



#### GAMの中身を確認する

```
vg view -a mapped.gam > mapped.json
```





## グラフを使う

### グラフの統計情報

#### グラフの総塩基数を出す

```
vg stats -l graph.vg
```



#### グラフのノード数とエッジ数を出す

```
vg stats -z grpah.vg
```



#### グラフのパスの数を出す

```
vg view graph.vg | grep ^P | wc -l
```



#### 指定したノードの座標が、グラフの特定のパスのどこに乗っているかを出す

```
vg find -n 10 -P chr1 -x index.xg  # IDが10のノードがchr1というパスでは、どこの座標に位置するのか
```



### グラフの編集

#### ノードの長さをN塩基以下に切る

```
vg mod -X 1000 graph.vg > graph.1000.vg  # ノードの大きさを最大で1000bpで切る
```

- [GCSAインデックスを作成するとき](https://github.com/vgteam/vg/issues/337)とかに使う



#### 指定したノードIDから距離N以下のノードを含むグラフを抽出する

```
vg find -n 5 -c 10 -x index.xg > node5.dis10.vg  # IDが5のノードから距離が10まで離れているノードまでのグラフを抽出する
```



#### 複数のvgファイルを1つにまとめる

```
vg ids -j 1.vg 2.vg  # ノードIDを揃える
cat 1.vg 2.vg > merged.vg
```





### マッピング

#### GCSAインデックスを作成する

```
vg index -g index.gcsa -k 16 -b . graph.vg  # -bでtmpファイルをおくディレクトリを指定

# メモリ消費量がしんどい場合は、
vg prune graph.vg > prune.vg  # グラフの簡略化
vg -g index.gcsa -k 16 -b . prune.vg  # メモリ消費量を減らすことができる
rm prune.vg
```



#### paired-endリードをマッピングする

```
# xgとgcsaがあることは前提として
vg map -x index.xg -g index.gcsa -t 1 -f 1.fq -2 2.fq > mapped.gam
```



#### 1塩基単位のカバレッジを計算する

```
vg pack -x index.xg -g mapped.gam -d > coverage.tsv

# pileup的なものがほしいときは、-eフラグをたてる
vg pack -x index.xg -g mapped.gam -d -e > coverage.edit.tsv
```



#### マップされなかったリードの情報を除く

```
vg view -a mapped.gam | jq -cr 'select(.score > 0)' | vg view -aJG - > filtered.gam
```



#### マッピング結果を%identity(配列類似度)でフィルタリングする

```
vg view -a mapped.gam | jq -cr 'select(.identity >= 0.95)' | vg view -aJG - > filtered.id95.gam
```

