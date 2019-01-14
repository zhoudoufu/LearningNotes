# Librispeech note 

### chain model
####chain model是在nnet3框架下，本质上是参数的调整。主要特性：
1， 采样率是原来的1/3.
2， 模型是目标函数是the log probability of the correct sequence.（对序列，本质是计算MMI，但是实际应用上规避了lattice生成的步骤 
by doing a full forward-backward on a decoding graph derived from a phone n-gram language model.）
3，因为采样率降低, 需要改变 HMM topologies， (允许HMM一个state就通过).
4，目前使用固定的转移概率，没有对转移矩阵作训练，目前NN输出概率作用和转移概率一样 
5，目前只支持nnet3
6，三倍速解码，且性能有5%的相对提升，训练时间也缩短 

####nnet3框架的功能：

nnet3既支持简单的前向传播DNN，而且支持时延神经网络tdnn，以及递归的拓扑结构（RNN， LSTM ，BLSTM等），因此nnet3的数据结构“知道”时间轴。


