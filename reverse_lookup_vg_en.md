# VG Reverse Lookup Cheat Sheet

## The objectives

There are a variety of informative tutorials for `vg` such as ([Main Wiki](https://github.com/vgteam/vg/wiki/Basic-Operations) and [Workshop in Portuguese](https://github.com/Pfern/PANGenomics)). However when we do not know right vg subcommand and options for what we want to do, it is not easy to find them from these tutorials. To address this, we have organized command line examples of vg in temrs of what we want to do.

The version is `v1.10.0 "Rionero"`. The static binary and Docker image are available from [here](https://github.com/vgteam/vg/releases/tag/v1.10.0).




## Constricuting a graph

### Constructing a graph from a sequence

#### Constructing a graph from a reference genome sequence and valiant call information

```
vg construct -r ref.fa -v valiant.vcf > graph.vg
```



#### Constructing a graph from a multiple sequence alignment from all sequences

```
vg msga -f multi.fa > graph.vg
```

- [X-drop DP algorithm was added](https://github.com/vgteam/vg/pull/1752)


#### Converting a graph from a multiple sequence alignment 

```
clustalo -i multi.fa > msa.fa  # Use a multiple aligner instead of vg msga
vg construct -F fasta -M msa.fa > graph.vg
```


## Format conversion

#### Converting vg format into a human readable format

```
vg view graph.vg > graph.gfa # Convert vg to GFA
vg view -j graph.vg > convert graph.json # vg to JSON
```

- By converting into JSON format, it can be visualized the genome graph by using a genome graph browser such as MoMIG.
  - Example:[MoMIG](http://viewer.momig.tokyo/demo3/#force_layout=false&sankey=false&path=chr12:80,851,974-80,853,202)
    - Note: path information is mandately.



#### Converting GFA format to vg format

```
vg view -Fv hoge.gfa > graph.vg
```

- When using assembly graph for downstream analysis, this
  - If you use `minia` as an assembler, you can use [here](https://github.com/Pfern/PANGenomics/blob/5923c991962396f30ce8adef9eef4d0a1ecd68b8/exercises/bacteria/README.md#gfa-input-to-vg-from-minia-sand-bcalm)



#### Converting an assembly graph by SPAdes into vg

```
grep -v ^P assembly_graph_with_scaffolds.gfa | vg view -Fv - | vg mod -X 1000 - > graph.vg
```

- Above example is confirmed by the SPAdes v3.11.1



#### Visualizing vg format with Graphviz

```
vg view -d graph.vg | dot -Tpng -o vis.png # Converting vg format into dot format
vg view -dnp graph.vg | dot -Tpng -o vis.png # Highliting each path on the vg graph
```



#### Converting vg format into xg format

```
vg index -x index.xg graph.vg
```



#### Converting GAM format into JSON format

```
vg view -a mapped.gam > mapped.json
```





## Using graphs

### Showing statistics of a graph

#### Showing the total number of bases in a graph

```
vg stats -l graph.vg
```



#### Showing the number of nodes and edges in a graph

```
vg stats -z grpah.vg
```



#### Showing the number of paths in a graph

```
vg view graph.vg | grep ^P | wc -l
```



#### Retrieving the coodinate of a user-specified node on a user-specified path

```
vg find -n 10 -P chr1 -x index.xg # Where is the coodinate of the node ID 10 on the path chr 1.
```

#### Showing the number of heads and tails

```
vg stats -H graph.vg  # heads
vg stats -T graph.vg  # tails
```

- Confirm if the graph has heads or tails by this command
  - [Reference](https://github.com/vgteam/vg/wiki/visualization#visualizing-bidirectional-sequence-graphs)


#### Showing the information of subgraphs

```
vg stats -s graph.vg > subgraph.tsv
```

- Column 1 is the list of node ids, and column2 is the size of the subgraph.



### Edit a graph

#### Dividing each node into the length of N bases or less

```
vg mod - X 1000 graph.vg > graph.1000.vg # Divinding each node into the 1000 bases or less.
```

- [When creating GCSA index](https://github.com/vgteam/vg/issues/337)



#### Merging multiple nodes without path branch into one node

```
vg mod -u graph.vg > merged.graph.vg
```



#### Reassigning the node IDs

```
vg mod -c graph.vg > fixed.graph.vg # Sort node ids keeping current set of node ids
vg ids -s graph.vg > rename.vg  # Reassign node ids from 1-origin
```



#### Extracting a graph consisting of nodes within N steps from a user-specified node


```
vg find -n 5 -c 10 -x index.xg> node5.dis10.vg # Extract the graph from the node whose ID is 5 to the node whose distance is 10
```


#### Extracting a graph consisting of nodes whose distances of bases from a user-specified node are less than N

```
vg find -n 5 -c 10 -L -x index.xg > node5.dis10.vg # Extracting the graph consisting of nodes whose distances of bases from the node 5 are less than 10(bp)
```


#### Extracting a graph consisting of nodes whose number is less than or equal to N from the specified path, e

```
vg find -n 5 -c 10 -p chr 1:50000-55520 -x index.xg > chr1:50000-55520.vg # chr1:50000-55520 and the nodes that are away from it by 10 Extract graph of
```


#### Merging multiple vg format files into one

```
vg ids -j 1.vg 2.vg # Aligning node IDs of 1.vg and 2.vg
cat 1.vg 2.vg > merged.vg
```

- This command modifies the original file in-place although other commands output modified graph to STDOUT

#### Extending the reference graph by adding the mutation information of the query sequence

```
vg augment -a direct grpah.vg aln.gam > aug.vg
```

- From v1.10.0 or later, the default of the option `-a` is `direct` instead of` pileup` [Reference](https://github.com/vgteam/vg/pull/1824).
- Unlike `vg mod -i`, it does not put path information. For the difference between these two, please refer [here](https://github.com/vgteam/vg/issues/1801).


#### Break a graph into connected components in their own files in the given directory

```
vg explode graph.vg subgraph_dir
```



#### Accumulate the graph touched by the alignments in the GAM

```
vg find -G aln.gam graph.vg > aligned_region.vg
```



#### Remove subgraphs by length

```
vg mod -l 1000 -S graph.vg > graph.1000.vg  # Remove subgraphs which are shorter than 1,000 bp
```

- The graph without heads cannot be removed so manually filter is required (i.e. by `vg explode` ).
  - [Reference](https://github.com/vgteam/vg/blob/4f4e5516abe873e1e6322014597ede500849cef3/src/vg.cpp#L6910-L6950)


#### Make spacific paths or nodes in a graph circular

```
vg circularize -p chr1 graph.vg > circularized.vg
```

- Circularize the path by connecting its head/tail node


### Mapping

#### Creating a GCSA index

```
vg index -g index.gcsa -k 16 -b . graph.vg # Option -b specifies the directory where the temporary file is to be placed

# When memory consumption is too large,
vg prune graph.vg > prune.vg # Firstly simplifying the graph
vg index -g index.gcsa -k 16 -b . prune.vg # Then the foregoing commnad can be executed with less memory
rm prune.vg
```

- Because many large intermediate files are yielded in `TMPDIR` on generating k-mers, you should confirm the following two points.
  - Location of TMPDIR
  - Enough disk space
    - [Reference](https://github.com/vgteam/vg/wiki/working-with-a-whole-genome-variation-graph)
      - > **important**: The location of the temporary files created for this process is specified using the TMPDIR environment variable. Make sure it is set to a volume a couple of terabytes of free space


- Although the file extention is `.gcsa`, the implementation is not GCSA but GCSA2. 
  - [Reference](https://github.com/vgteam/vg/blob/65cef16db927008bfbee50c072907c25a3853bf1/src/subcommand/index_main.cpp#L61)

#### Mapping paired-end reads

```
#It is assumed that xg and gcsa files exist
vg map -x index.xg -g index.gcsa -t 1 -f 1 fq -f 2.fq > mapped.gam
```


- [X-drop DP algorithm was added](https://github.com/vgteam/vg/pull/1752)

#### Calculating the read coverage of each base

```
vg pack -x index.xg -g mapped.gam -d > coverage.tsv

# If you want something like pileup, set the -e flag
vg pack -x index.xg -g mapped.gam -d -e > coverage.edit.tsv
```



#### Excluding unmapped reads

```
vg view -a mapped.gam | jq - cr 'select (.score > 0)' | vg view - aJG - > filtered.gam
```



#### Filtering mapping results by the percent identity (sequence similarity)

```
vg view - a mapped.gam | jq - cr 'select (.identity> = 0.95)' | vg view - aJG - > filtered.id95.gam
```



#### Showing statistis of mapping results

```
vg stats -a mapped.gam graph.vg
```

- It is not used for the calculation, but position argument is necessary



#### Among mapping results, projects corresponding to linear paths are projected onto a bam/sam file

```
vg surject -x index.xg -t 1 -b mapped.gam > mapped.bam

#You can extract only the mappings for the path specified with -p option
vg surject -x index.xg -t 1 -s -p chr1 mapped.gam > mapped.sam
```



#### Projection of bam/sam mapping results for reference to gam on genome graph with same path.

```
vg inject -x index.xg -t 1 mapped.sam > mapped.gam
```



### Adding gene annotations into a graph as a path

To put the gene annotation as a path on the vg graph, first you need to convert the gene annotation into an alignment for the genome graph, then create a path and merge it into the vg graph.

#### Converting annotations in the BED or GFF format into the alignment file

```
vg annotate -b input.bed -x index.xg > annotation.gam
vg annotate -g input.gff -x index.xg > annotation.gam
```


#### Adding an alignment file to a graph as vg path

```
# Embed GAM alignments into a graph, and then merge the paths implied by alignments into the graph
vg mod -i annotation.gam graph.vg > mod.vg

# Don't edit with -i alignments, just use them for labeling the graph
vg mod -P -i annotation.gam graph.vg > graph.include_path.vg
```

### WIP: Extracting subgraph related information from a graph

Please be aware that there are uncertain points at present


#### Showing a list of nodes which are forming snarls.

```
vg snarls -m 1000 -r list.st graph.vg > snarls.pb
vg view -E list.st | jq '.visit[1: -1][].node_id | select (.! = null) | tonumber' | sort -n | uniq > node_list_in_ultra_bubble.txt

# Showing nodes in core regions (i.e. hub structures in a graph)
vg view graph.vg | grep ^S | cut -f 2 | grep -vwf node_list_in_ultra_bubble.txt > node_list_of_core_region.txt
```

- A Snarl is a generalization of the superbubble which is a subgraph of a genome graph. For the definition of terms, please refer to [Paten et al.](https://www.biorxiv.org/content/early/2017/01/18/101493)
- Note: there is an inconsistency (as of Aug 27, 2018) that the `-m` option of `vm snarls` only compute traversals for snarls with `<=` N nodes in the help message, but `<` according to the [SourceCode](https://github.com/vgteam/vg/blob/02a085c1f9902d94a25e8cdffafc16eb7ff8a4a2/src/subcommand/snarls_main.cpp#L228) -> [Fixed in v1.10.0](https://github.com/vgteam/vg/pull/1840)

## TODO

- How to call variants
