# Librispeech note 

### chain model
#### chain model是在nnet3框架下，本质上是参数的调整。主要特性：
1， 采样率是原来的1/3.
2， 模型是目标函数是the log probability of the correct sequence.（对序列，本质是计算MMI，但是实际应用上规避了lattice生成的步骤 
by doing a full forward-backward on a decoding graph derived from a phone n-gram language model.）
3，因为采样率降低, 需要改变 HMM topologies， (允许HMM一个state就通过).
4，目前使用固定的转移概率，没有对转移矩阵作训练，目前NN输出概率作用和转移概率一样 
5，目前只支持nnet3
6，三倍速解码，且性能有5%的相对提升，训练时间也缩短 

#### nnet3框架的功能：

nnet3既支持简单的前向传播DNN，而且支持时延神经网络tdnn，以及递归的拓扑结构（RNN， LSTM ，BLSTM等），因此nnet3的数据结构“知道”时间轴。


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

拓扑结构Topology定义了两种， 一种是所有发音因素公用一个拓扑结构，另一种是所有静音因素公用一个拓扑结构。

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
