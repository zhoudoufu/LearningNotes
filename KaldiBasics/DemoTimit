
# Demo Timit

### 1. Intro

Timit数据是有speaker标注，性别标注的。训练基本过程是GMM到DNN， 并且有使用说话人的信息优化训练效果。
主要目的是熟悉kaldi的代码风格，文件格式。以及timit的训练过程，迭代优化方式。
### 2. Workflow



### 3. Details

#### 未整理顺序部分 tbd

* step1 整理数据

### Kaldi的数据格式

kaldi中数据索引有两种indexed by utterance（按音频文件索引） 和 indexed by segment（按时间戳索引，方便一个音频中有若干句的情况，支持丢弃一段音频中的部分内容）。

- .stm 文件（indexed by segment）

> file-name ? speaker-name segment-begin segment-end label text segment

>fadg0_si1279 1 fadg0 0.0 1.817625 <O,F>  sil b r ih cl k s aa r er n ao l cl t er n ix cl t ih v sil

- .mdl 文件 hmm的模型文件

gmm文件一般保存成二进制的，需要用命令转化成txt 方便查看：

> gmm-copy ‐‐binary=false binary.mdl text.mdl

打开后 网络的拓扑结构保存为：

> &lt;Topology>  
&lt;TopologyEntry>  
&lt;ForPhones> 1 2 3 4 5 6 7 8 &lt;/ForPhones>  
&lt;State> 0 &lt;PdfClass> 0  
&lt;Transition> 0 0.5  
&lt;Transition> 1 0.5  
**&lt;/State>  
&lt;State> 1 &lt;PdfClass> 1  
&lt;Transition> 1 0.5  
&lt;Transition> 2 0.5  
&lt;/State>**  
&lt;State> 2 &lt;PdfClass> 2  
&lt;Transition> 2 0.5  
&lt;Transition> 3 0.5  
&lt;/State>  
&lt;State> 3  
&lt;/State>  
&lt;/TopologyEntry>  
&lt;/Topology>  

拓扑结构Topology只有一个， 有8个phone，8个phone公用一个拓扑结构

包含pdf的state为有发射发射概率的state。如加粗所示，包含一个自环，和一个转移概率。
这里定义的的转移概率用于模型初始化。

初始网络demo中命名为0.mdl 训练完成后最终网络链到 final.mdl


### Kaldi的代码风格

kaldi的文件输入输出流向是用rxfilename和wxfilename描述的， 这两个不是类，而是读和写的扩展名，eg.

> copy-feats ark:$filename.ark ark,t:-
这个命令一般用来查看特征文件，他的格式是copy-feats [options] feature-rspecifier feature-wspecifier

> std::string rspecifier1 = "scp:data/train.scp"; // script file.

> std::string rspecifier2 = "ark:-"; // archive read from stdin.

> // write to a gzipped text archive.

> std::string wspecifier1 = "ark,t:| gzip -c > /some/dir/foo.ark.gz";

> std::string wspecifier2 = "ark,scp:data/my.ark,data/my.ark";


|    标记     | 说明           | 例子  |
| :------------- |:-------------:| -----:|
| "-" means the standard input  | r w 通用 |    |
|  "b" (binary) means write in binary mode (currently unnecessary as it's always the default).   |    w  写入
|    |
|   "t" (text) means write in text mode.     |    w    |    |
|  "f" (flush) means flush the stream after each write operation.      |    w    |  "ark,t,f:data/my.ark"  |
|   "nf" (no-flush) means don't flush the stream after each write operation (would currently be pointless, but calling code can change the default).     |     w   |    |
| "p" means permissive mode, which affects "scp:" wspecifiers where the scp file is missing some entries: the "p" option will cause it to silently not write anything for these files, and report no error.   |    w    |    |
| "o" (once) is the user's way of asserting to the RandomAccessTableReader code that each key will be queried only once. 这个选项使读入的内容用完后就不再保存。 |  r 读入 |   |
| "p" (permissive) 读入时检查到即使有问题也不报错，invalid的数据会跳过，跳不过的会停在最后卡住的地方 | r |   |
| "s" (sorted) 读入未排序的文件会报错 | r |   |
| "cs" (called-sorted) 读入序列排队，当一段数据被数个读入操作调取时，会自动忽略有 lower-numbered keys的请求实体 | r |   |

[Ref Kaldi](http://www.kaldi-asr.org/doc/io.html)
* cmvn 文件

* MLLT 特征变换

最大似然linear trainsform，是作用在提取出的mfcc特征上的， 主要目的是把协方差矩阵变成对角阵，初始化要是随机一个对角阵，试验多次取最好的。

* SAT training
Speaker adaptive training

[Ref paper](http://www.cs.cmu.edu/~ymiao/satdnn.html)
