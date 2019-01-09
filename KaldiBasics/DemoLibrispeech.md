# Librispeech note 

### chain model
chain model是在nnet3框架下，本质上是参数的调整。主要特性：
1， 采样率是原来的1/3.
2， 模型是目标函数是the log probability of the correct sequence.（对序列，本质是计算MMI，但是实际应用上规避了lattice生成的步骤 
by doing a full forward-backward on a decoding graph derived from a phone n-gram language model.）
3， Because of the reduced frame rate, we need to use unconventional HMM topologies (allowing the traversal of the HMM in one state).
4， We use fixed transition probabilities in the HMM, and don't train them (we may decide train them in future; but for the most part the neural-net output probabilities can do the same job as the transition probabilities, depending on the topology).
5， Currently, only nnet3 DNNs are supported (see The "nnet3" setup), and online decoding has not yet been implemented (we're aiming for April to June 2016).
6， Currently the results are a bit better then those of conventional DNN-HMMs (about 5% relative better), but the system is about 3 times faster to decode; training time is probably a bit faster too, but we haven't compared it exactly.
