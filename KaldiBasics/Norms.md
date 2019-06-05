## 0，kaldi的一些术语定义
参考： http://kaldi-asr.org/doc/glossary.html

*  pdf-id : The zero-based integer index of a clustered context-dependent HMM state ，可以理解为hmm-gmm模型对应的一个state 的id，与dnn的输出节点一一对应。

*  transition-state：a one-based index that encodes the pdf-id (i.e. the clustered context-dependent HMM state), the phone identity, and information about whether we took the self-loop or forward transition in the HMM. 与 dnn模型文件里的triples的行号对应，表示一个state上的转移概率的条转移概率的集合。一个transition-id表示其中一条转移概率弧。


## 1, tree

决策树定义，转为txt后： /opt/kaldi/egs/librispeech/s5/exp/tri6b/tree.txt

phone的映射列表：  /opt/kaldi/egs/librispeech/s5/data/lang/phones.txt


In practice we generally let each tree-root correspond to a "real phone", meaning that we group together all word-position-dependent, tone-dependent or stress-dependent versions of each phone into one group that becomes a tree root. 
相同的central-phone作为一个tree root。

`ContextDependency 3 1 `

表示context width = 3 ， central phone的位置是1 （从零开始）

ToPdf 之后的内容是树的内容。

SE (for split  event map)表示一棵树，

CE (for constant event map)表示叶子节点，每个叶子结点后面只有一个数值，表示pdf-id

还可以嵌套其他映射表用（table event map ）目前没用到。

tree can ask questions about the pdf-class as well as about phonetic context. 树的划分可以通过两种问题： 1， pdf-class（hmm模型的状态编号）2，前后音素（triphone）
0.  EventMap := ConstantEventMap | SplitEventMap | TableEventMap | "NULL"
1.  ConstantEventMap := "CE" <numeric pdf-id>
2.  SplitEventMap := "SE" <key-to-split-on> "[" yes-value-list "]" "{" EventMap EventMap "}"
3.  TableEventMap := "TE" <key-to-split-on> <table-size> "(" EventMapList ")"

这里SE 里面 key-to-split-on在当前tree的定义里，表示决策树划分的问题，可以是 0，1，2. 表示triphone的左边 中间 或右边 的phone。可以是-1 表示 identity of the "pdf-class". Normally the value of the pdf-class is the same as the index of the HMM state，例如：

`SE -1 [ 0 1 2 ]
{ CE 282 CE 780 }`   *phn的pdf-class是0 1 2 的，叶子结点pdf-id是282 780。*

`SE 2 [ 227 228 ]
{ CE 1752 CE 2802 }` *central phn 的右侧音素编号是227 228的， 叶子结点pdf-id是1752 2802。*

画出决策树tree供参考：


## 2,  mdl

dnn 的模型可以分为两个部分：  转移概率模型 和 发射概率模型
/opt/kaldi/egs/librispeech/s5/exp/nnet2_online/nnet_a/final.mdl.txt


*  转移概率模型

 网络的拓扑结构保存为：

`<Topology>
<TopologyEntry>
<ForPhones>
11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 146 147 148 149 150 151 152 153 154 155 156 157 158 159 160 161 162 163 164 165 166 167 168 169 170 171 172 173 174 175 176 177 178 179 180 181 182 183 184 185 186 187 188 189 190 191 192 193 194 195 196 197 198 199 200 201 202 203 204 205 206 207 208 209 210 211 212 213 214 215 216 217 218 219 220 221 222 223 224 225 226 227 228 229 230 231 232 233 234 235 236 237 238 239 240 241 242 243 244 245 246 247 248 249 250 251 252 253 254 255 256 257 258 259 260 261 262 263 264 265 266 267 268 269 270 271 272 273 274 275 276 277 278 279 280 281 282 283 284 285 286 287 288 289 290 291 292 293 294 295 296 297 298 299 300 301 302 303 304 305 306 307 308 309 310 311 312 313 314 315 316 317 318 319 320 321 322 323 324 325 326 327 328 329 330 331 332 333 334 335 336 337 338 339 340 341 342 343 344 345 346
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.75 <Transition> 1 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.75 <Transition> 2 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 2 0.75 <Transition> 3 0.25 </State>
<State> 3 </State>
</TopologyEntry>
<TopologyEntry>
<ForPhones>
1 2 3 4 5 6 7 8 9 10
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.25 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 3 <PdfClass> 3 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 4 <PdfClass> 4 <Transition> 4 0.75 <Transition> 5 0.25 </State>
<State> 5 </State>
</TopologyEntry>
</Topology>`

拓扑结构Topology定义了两种， 一种是所有发音音素公用一个拓扑结构，另一种是所有静音音素公用一个拓扑结构。

包含pdf的state为有发射发射概率的state。  如加粗所示，包含一个自环，和一个转移概率。
这里定义的的转移概率用于模型初始化。 dnn使用的转移概率模型是来自gmm训练，经过训练后的转移概率在：

/opt/kaldi/egs/librispeech/s5/exp/tri6b/final.show_trans

`Transition-state 38151: phone = ZH_S hmm-state = 2 pdf = 4075
 Transition-id = 76381 p = 0.75 count of pdf = 9085 [self-loop]
 Transition-id = 76382 p = 0.25 count of pdf = 9085 [2 -> 3]`

这里的transition-state 与 dnn模型文件里的triples的行号对应，表示一个state上的转移概率的条转移概率的集合。一个transition-id表示其中一条转移概率弧。

`<Triples> 38151
1 0 0
1 1 41
1 2 56
1 3 42
...
346 2 4075
</Triples>
`

上面是mdl里面的Triples域 ，格式是 : phn-id pdf-class pdf-id 

* 发射概率模型

之后的部分是dnn的定义： 
提取结构：

`num-components 17
num-updatable-components 5
left-context 7
right-context 7
input-dim 140
output-dim 5704
parameter-dim 8141104
component 0 : SpliceComponent, input-dim=140, output-dim=700, context=-7 -6 -5 -4 -3 -2 -1 0 1 2 3 4 5 6 7 , const_component_dim=100
component 1 : FixedAffineComponent, input-dim=700, output-dim=700, linear-params-stddev=0.00303107, bias-params-stddev=0.461679
component 2 : AffineComponentPreconditionedOnline, input-dim=700, output-dim=3500, linear-params-stddev=0.997007, bias-params-stddev=2.33597, learning-rate=0.00133022, rank-in=20, rank-out=80, num_samples_history=2000, update_period=4, alpha=4, max-change-per-sample=0.075
component 3 : PnormComponent, input-dim = 3500, output-dim = 350, p = 2
component 4 : NormalizeComponent, input-dim=350, output-dim=350
component 5 : AffineComponentPreconditionedOnline, input-dim=350, output-dim=3500, linear-params-stddev=1.00035, bias-params-stddev=0.890683, learning-rate=0.00133022, rank-in=20, rank-out=80, num_samples_history=2000, update_period=4, alpha=4, max-change-per-sample=0.075
component 6 : PnormComponent, input-dim = 3500, output-dim = 350, p = 2
component 7 : NormalizeComponent, input-dim=350, output-dim=350
component 8 : AffineComponentPreconditionedOnline, input-dim=350, output-dim=3500, linear-params-stddev=1.00039, bias-params-stddev=0.870742, learning-rate=0.00133022, rank-in=20, rank-out=80, num_samples_history=2000, update_period=4, alpha=4, max-change-per-sample=0.075
component 9 : PnormComponent, input-dim = 3500, output-dim = 350, p = 2
component 10 : NormalizeComponent, input-dim=350, output-dim=350
component 11 : AffineComponentPreconditionedOnline, input-dim=350, output-dim=3500, linear-params-stddev=1.00028, bias-params-stddev=0.916529, learning-rate=0.00133022, rank-in=20, rank-out=80, num_samples_history=2000, update_period=4, alpha=4, max-change-per-sample=0.075
component 12 : PnormComponent, input-dim = 3500, output-dim = 350, p = 2
component 13 : NormalizeComponent, input-dim=350, output-dim=350
component 14 : AffineComponentPreconditionedOnline, input-dim=350, output-dim=5704, linear-params-stddev=0.646413, bias-params-stddev=0.0428529, learning-rate=0.00133022, rank-in=20, rank-out=80, num_samples_history=2000, update_period=4, alpha=4, max-change-per-sample=0.075
component 15 : FixedScaleComponent, input-dim=5704, output-dim=5704, scales-mean=1, scales-stddev=0.183266
component 16 : SoftmaxComponent, input-dim=5704, output-dim=5704
prior dimension: 5704, prior sum: 1, prior min: 1.49816e-06`

 **FixedAffineComponent** is the LDA-like(Linear discriminant analysis) decorrelating transform 

 **AffineComponentPreconditionedOnline** is a refinement of AffineComponent. An AffineComponent would consist of the standard (weight matrix plus bias term) that appears in neural networks.

 The **PnormComponent** is the nonlinearity

The **NormalizeComponent** is something we add to stabilize the training of p-norm networks. It is also a fixed, non-trainable nonlinearity

**FixedScaleComponent** applies a fixed per-element scale.


供参考的文件：

[phones.txt](/uploads/f9dc32992d9f5ecb2ba9dd9f7e8e06d0/phones.txt)

[final.mdl.trans.txt](/uploads/f99a5266aca0a5891010ee04a92aa8d2/final.mdl.trans.txt)

[tree.txt](/uploads/aab6978da6f561bf112726e4b1e1212a/tree.txt)


## 3, Forced alignment 输出结果

1， Forced alignment 参数

--transition-scale=1.0 

--self-loop-scale=0.1 

--acoustic-scale=0.1 

--beam=10 

--retry-beam=40

* 举例说明 transition-scale和self-loop-scale的用法：

假设一个state有三条弧，其中包括一个自环，两个非自环。在mdl的定义中，假设自环的概率是p，两个非自环的概率是q1，q2，且 p+q1+q2=1。FA时设定 --transition-scale=a 和 --self-loop-scale=b。经过参数调整后实际解码时的转移概率是：

自环 ： b * log(p)

非自环q1 : b * log(1-p) + a * log(q1/(q1+q2)) 

非自环q2 ： b * log(1-p) + a * log(q2/(q1+q2))

beam : Decoding beam used in alignment

retry-beam : Decoding beam for second try at alignment

>  SimpleDecoder is a straightforwardly implemented Viterbi beam search algorithm with only a single tunable parameter:

提取特征的帧长 25ms， 帧移10ms

音频长度（ms为单位）= （帧数-1）* 10ms + 25ms

FA的结果每帧对应一个transition-id， 

2, 输出结果解释

输出alignment是二值压缩包， 例如 ali.1.gz, 可以用show-alignments命令转换为可读文本：

`show-alignments phones.txt final.mdl ark:"gunzip -c ali.1.gz|" > ali.1.txt`

`1272-141231-0032  [ 4 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 16 18 17 ] [ 23414 23413 23413 23413 23413 23413 23413 23413 23413 23413 23413 23413 23413 23413 23413 23476 23475 23475 23564 23563 ] [ 26796 26795 26795 26936 26935 27020 27019 27019 ] [ 56016 56015 56015 56015 56156 56348 ] [ 23388 23496 23580 ] [ 8742 8741 8930 8929 9054 9053 ] [ 61174 61173 61234 61233 61233 61276 61275 61275 61275 61275 ] [ 16304 16303 16334 16344 ] [ 29876 29972 29971 29971 29971 29971 29971 29971 30046 30045 30045 ] [ 35290 35289 35289 35336 35335 35335 35335 35335 35398 ] [ 9230 9590 9942 ] [ 53176 53248 53247 53348 53347 53347 53347 ] [ 65442 65441 65520 65519 65519 65519 65822 65821 65821 65821 65821 65821 ] [ 74436 74508 74507 74507 74620 ] [ 42948 43038 43037 43037 43037 43272 ] [ 64308 64307 64307 64307 64307 64307 64307 64307 64366 64496 64495 64495 64495 ] [ 65918 65917 66110 66288 66287 ] [ 23404 23470 23596 23595 ] [ 9352 9351 9582 9581 9798 9797 9797 ] [ 65938 65937 66194 66408 ] [ 67770 67792 67791 67791 67791 67791 67820 67819 ] [ 62762 62761 62761 62944 63062 ] [ 11418 11594 11680 11679 11679 ] [ 64248 64247 64384 64383 64383 64496 64495 ] [ 66014 66013 66013 66013 66200 66372 66371 66371 ] [ 41842 42004 42148 ] [ 66032 66031 66228 66227 66404 ] [ 7930 8190 8189 8354 8353 8353 ] [ 64276 64275 64275 64275 64275 64410 64409 64409 64409 64409 64478 64477 64477 64477 ] [ 19056 19055 19055 19055 19055 19055 19055 19055 19055 19055 19055 19132 19131 19131 19131 19131 19148 19147 19147 19147 19147 19147 19147 ] [ 21984 21983 21983 21983 21983 21983 22082 22081 22081 22081 22081 22344 22343 22343 22343 22343 22343 22343 22343 22343 22343 22343 ] [ 4 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 16 18 ] [ 41784 41783 41783 41783 41783 41783 41783 41944 41943 41943 41943 41943 41943 42138 ] [ 55986 56192 56191 56420 ] [ 7830 7829 8072 8400 8399 ] [ 56432 56644 56838 ] [ 22010 22009 22009 22102 22224 22223 ] [ 10852 10952 10951 11008 ] [ 56460 56716 56902 ] [ 22402 22401 22560 22712 22711 ] [ 29580 29738 29737 29737 29806 29805 29805 ] [ 23436 23435 23435 23506 23558 ] [ 8624 8623 8623 8902 8901 9166 9165 ] [ 35624 35623 35648 35647 35647 35666 35665 35665 35665 ] [ 1708 1707 1707 1782 1781 1824 1823 1823 1823 1823 1823 ] [ 62758 62757 62757 62757 62757 62757 62757 62757 62757 62757 62757 62946 62945 63022 63021 ] [ 21892 21891 21891 21891 21891 22084 22083 22083 22318 22317 22317 22317 22317 22317 22317 22317 ] [ 4 1 1 1 1 1 1 1 1 16 18 ]
1272-141231-0032  SIL                                                                                                        DH_B                                                                                                                        EH1_I                                               N_E                                     DH_B                  AH0_E                             P_B                                                             AW1_I                       ER0_I                                                                 F_I                                                       AH0_I              L_E                                           T_B                                                                         W_I                               IH1_I                                   S_I                                                                               T_E                               DH_B                        AH0_I                                  T_E                         TH_B                                                R_I                               AH1_I                             S_I                                           T_E                                                 IH1_B                 T_E                               AH0_B                             S_I                                                                                     AY1_I                                                                                                                                         D_E                                                                                                                                     SIL                                               IH1_B                                                                                   N_E                         AH0_B                        N_I                   D_E                                     AH1_B                       N_I                   D_I                               ER0_E                                         DH_B                              AH0_E                                  G_B                                                       AA1_I                                                      R_I                                                                                           D_E                                                                                                 SIL`

上面是一个音频的FA结果：

第一部分是每帧一个transition-id作为输出，同一个音素用方括号括起来，transition-id 的个数对应时长。
第二部分是每个方括号内的部分对应的音素。
以上音频4.48s长，共有446个transition-id输出。
