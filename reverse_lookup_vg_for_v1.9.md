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



#### マルチプルアラインメントされた配列セットをグラフにする

```
clustalo -i multi.fa > msa.fa  # vg msga の代わりに汎用的なマルチプルアライナーを使う
vg construct -F fasta -M msa.fa > graph.vg
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
    - 詳細な使い方は[ここ](https://github.com/genomegraph/workshop/blob/master/browser_tutorial/browser_tutorial.md)を参照



#### GFAをvgに変換する

```
vg view -Fv hoge.gfa > graph.vg
```

- [v1.9.0 でオーバーラップつきGFAパーサーのバグがとれた(らしい)](https://github.com/vgteam/vg/pull/1765)
- アセンブリグラフを下流の解析に用いるときはこれ
  - アセンブラとして `minia` を使うなら、[ここ](https://github.com/Pfern/PANGenomics/blob/5923c991962396f30ce8adef9eef4d0a1ecd68b8/exercises/bacteria/README.md#gfa-input-to-vg-from-minia-and-bcalm)を参照



#### SPAdesのアセンブリグラフをvgに変換する

```
grep -v ^P assembly_graph_with_scaffolds.gfa | vg view -Fv - | vg mod -X 1000 - > graph.vg
```

- SPAdesはv3.11.1を使用



#### vgをGraphvizで簡単に可視化

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



#### 分岐はないが、複数のノードに分離されている領域をマージする

```
vg mod -u graph.vg > merged.graph.vg
```



#### ノードIDを整理して、振り直す

```
vg mod -c graph.vg > fixed.graph.vg
```



#### 指定したノードIDからノードの個数の距離N以下のノードを含むグラフを抽出する

```
vg find -n 5 -c 10 -x index.xg > node5.dis10.vg  # IDが5のノードから距離が10まで離れているノードまでのグラフを抽出する
```



#### 指定したノードIDからノードに含まれる塩基配列の距離N以下のノードを含むグラフを抽出する

```
vg find -n 5 -c 10 -L -x index.xg > node5.dis10.vg # IDが5のノードから距離が10bpまで離れているノードまでのグラフを抽出する
```



#### 指定したパス上の座標から、ノードの個数N以下のノードを含むグラフを抽出する

```
vg find -n 5 -c 10 -p chr1:50000-55520 -x index.xg > chr1:50000-55520.vg # chr1:50000-55520に対応するノードおよびそこから距離が10まで離れているノードまでのグラフを抽出する
```



#### 複数のvgファイルを1つにまとめる

```
vg ids -j 1.vg 2.vg  # ノードIDを揃える
cat 1.vg 2.vg > merged.vg
```



#### クエリ配列の変異情報をリファレンスに加えてグラフを拡張する

```
vg augment -a direct grpah.vg aln.gam > aug.vg
```

- v1.10.0以降では、 `-a` のデフォルトは `pileup` ではなく`direct` になるのでは？→ [参考](https://github.com/vgteam/vg/pull/1824)
-  `vg mod -i` と違ってパスの情報は載せない。この2つの違いは[ここ](https://github.com/vgteam/vg/issues/1801)を参照





### マッピング

#### GCSAインデックスを作成する

```
vg index -g index.gcsa -k 16 -b . graph.vg  # -bでtmpファイルをおくディレクトリを指定

# メモリ消費量がしんどい場合は、
vg prune graph.vg > prune.vg  # グラフの簡略化
vg index -g index.gcsa -k 16 -b . prune.vg  # メモリ消費量を減らすことができる
rm prune.vg
```



#### paired-endリードをマッピングする

```
# xgとgcsaがあることは前提として
vg map -x index.xg -g index.gcsa -t 1 -f 1.fq -f 2.fq > mapped.gam
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



#### マッピング結果の統計情報をみる

```
vg stats -a mapped.gam graph.vg
```

- 計算に使わないが、位置引数が必要



#### マッピング結果のうち、リニアなパスに対応するものをbam/samファイルに射影する

```
vg surject -x index.xg -t 1 -b mapped.gam > mapped.bam

# -p でパス名を指定すると、そのパスに対するマッピングだけを抽出できる
vg surject -x index.xg -t 1 -s -p chr1 mapped.gam > mapped.sam
```



#### リファレンスに対するbam/samのマッピング結果を、同じパスをもつゲノムグラフ上のgamに射影する

```
vg inject -x index.xg -t 1 mapped.sam > mapped.gam
```



### 遺伝子アノテーション

遺伝子アノテーションをvgのグラフ上にパスとして載せるには、一度遺伝子アノテーションをゲノムグラフに対するアラインメントに変換し、それから
パスを生成して、vgのグラフにマージするという作業を行います。



#### bed/gffフォーマットのアノテーションをアラインメントに変換する

```
vg annotate -b input.bed -x index.xg > annotation.gam
vg annotate -g input.gff -x index.xg > annotation.gam
```



#### アラインメントをvgのパスとして追加する

```
vg mod -P --include-aln annotation.gam graph.vg > mod.vg

# アノテーションの切れ目でノードを分割したい場合は、-P を外す
vg mod -P --include-aln annotation.gam graph.vg > mod.vg
```



### WIP: グラフからの情報抽出

怪しいところもあるので注意



#### BubbleになっているノードIDのリストを抽出する

```
vg snarls -m 1000 -r list.st graph.vg > snarls.pb
vg view -E list.st | jq '.visit[1:-1][].node_id | select(. != null) | tonumber' | sort -n | uniq > node_list_in_ultra_bubble.txt

# コア領域(=ハブになっている領域)のノードIDのリストを抽出には
vg view graph.vg | grep ^S | cut -f 2 | grep -vwf node_list_in_ultra_bubble.txt > node_list_of_core_region.txt
```

- 用語の定義は[Paten et al.](https://www.biorxiv.org/content/early/2017/01/18/101493)を参照
-  `vg snarls` の `-m` は helpでは `<=` だが、 [ソースコード](https://github.com/vgteam/vg/blob/02a085c1f9902d94a25e8cdffafc16eb7ff8a4a2/src/subcommand/snarls_main.cpp#L228)では `<` となっているので注意




## TODO

- valiant callの話
