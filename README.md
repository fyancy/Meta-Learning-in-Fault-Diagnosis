# Meta-Learning-in-Fault-Diagnosis
The source codes for Meta-learning for few-shot cross-domain fault diagnosis.

# Instructions
* To run all models, the requirements of your python environmrnt are as: 1) pytorch 1.8.1+cu102; 2) tensorflow-gpu 2.4.0. Note that only `MANN` is implemented by tensorflow, all other methods are achieved by pytorch. Thus, with pytorch only, you can observe the performance of most methods on CWRU dataset.
* Some packages you have to install: 1) tensorflow_addons (for optimizer AdamW in tensorflow. Not really necessary); 2) learn2learn. The latter is an advanced API to achieve meta-learning methods, which is definitely compatible with pytorch. If you have problems when installing learn2learn, such as 'Microsoft Visual C++ 14.0 is required.', please refer to https://zhuanlan.zhihu.com/p/165008313; 3) Visdom (for visualization).
* The codes of these methods follow the idea of the original paper as far as possible, of course, for application in fault diagnosis, there are some modifications.

# Methods
```
1. CNN
2. CNN with fine-tuning (CNN-FT) [1]
3. CNN with Maximum Mean Discrepancy (CNN-MMD) [2]
4. Model Agnostic Meta-Learning (MAML) [3]
5. Reptile [4]
6. Memory Augmented Neural Network (MANN) [5]
7. Prototypical Networks (ProtoNet) [6]
8. Relation Networks (RelationNet) [7]
```
## Tasks on CWRU bearing dataset
```
T1: 10 ways, load 3 ==> 10 ways, load 0  
T2: 6 ways, load 0 ==> 4 ways, load 0  
Details can be found in `cwru_path.py` 
```
### CNN-based methods
* **CNN**

|Tasks|shots|Acc.(%)|Test time (s)|Trainging time (s)|Memory (KB)|
|:----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|T<sub>1</sub>|5|71.80|	1.183|	2.484|321|

* **CNN-FT**

|Tasks|shots|Acc.(%)|Test time (s)|Trainging time (s)|Memory (KB)|
|:----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|T<sub>1</sub>|5|	75.90|	3.995|	2.484|	321|
|T<sub>1</sub>|1|	48.00|	3.45|	-|		321|
|T<sub>2</sub>|5|	82.50|	5.72|	-|		225|
|T<sub>2</sub>|1|	68.00|	4.68|	-|		225|

* **CNN-MMD**

|Tasks|shots|Acc.(%)|Test time (s)|Trainging time (s)|Memory (KB)|
|:----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|T<sub>1</sub>|5|81.53|	1.164|	15.38|321|

### Meta-learning methods
* **MAML**

|Tasks|shots|Acc.(%)|Test time (s)|Trainging time (s)|Memory (KB)|
|:----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|T<sub>1</sub>|5	|95.80	|5.654	|720	|321|
|T<sub>1</sub>|1	|87.40	|4.494	|233	|321|
|T<sub>2</sub>|5	|91.95	|6.507	|312	|225|
|T<sub>2</sub>|1	|77.50	|4.455	|340	|225|

* **Reptile**

|Tasks|shots|Acc.(%)|Test time (s)|Trainging time (s)|Memory (KB)|
|:----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|T<sub>1</sub>|5	|94.6	|18.00	|1820	|321|
|T<sub>1</sub>|1	|bad	|-	|-	|-|
|T<sub>2</sub>|5	|91.50	|17.528	|585.6s	|225|
|T<sub>2</sub>|1	|55.15	|17.59	|532	|225|

* **ProtoNet**

|Tasks|shots|Acc.(%)|Test time (s)|Trainging time (s)|Memory (KB)|
|:----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|T<sub>1</sub>|5	|95.30	|0.290	|41	    |160|
|T<sub>1</sub>|1	|87.69	|0.121	|24		|160||
|T<sub>2</sub>|5	|89.18	|0.161	|-		|160|
|T<sub>2</sub>|1	|77.25	|0.104	|-		|160|

* **RelationNet**

|Tasks|shots|Acc.(%)|Test time (s)|Trainging time (s)|Memory (KB)|
|:----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|T<sub>1</sub>|5	|92.34	|0.3472|	304	| 1339|
|T<sub>1</sub>|1	|85.65	|0.15|	102|	 1339|
|T<sub>2</sub>|5	|93.22	|0.19|	275|	 1339|
|T<sub>2</sub>|1	|77.98	|0.129|	-|	 1339|


* **MANN**

|Tasks|shots|Acc.(%)|Test time (s)|Trainging time (s)|Memory (KB)|
|:----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|T<sub>1</sub>|1|	88.35|	0.12|	90|4134|

## Feature extractor
The backbone of these methods, i.e. feature extractor, consists of four convolution blocks, as follows
```python
import torch.nn as nn

def conv_block(in_channels, out_channels):
    return nn.Sequential(
        nn.Conv1d(in_channels, out_channels, kernel_size=3, padding=1),
        nn.BatchNorm1d(out_channels),
        nn.ReLU(),
        nn.MaxPool1d(kernel_size=2),
    )


class encoder_net(nn.Module):
    def __init__(self, in_chn, hidden_chn, cb_num=4):
        super().__init__()
        conv1 = conv_block(in_chn, hidden_chn)
        conv1_more = [conv_block(hidden_chn, hidden_chn) for _ in range(cb_num - 1)]
        self.feature_net = nn.Sequential(conv1, *conv1_more)  # (None, 64, 1024/2^4)

    def forward(self, x):
        return self.feature_net(x)
```


**References**  
```
[1] Li, F., Chen, J., Pan, J., & Pan, T. (2020). Cross-domain learning in rotating machinery fault diagnosis under various operating conditions based on parameter transfer. Measurement Science and Technology, 31(8), 085104.  
[2] Xiao, D., Huang, Y., Zhao, L., Qin, C., Shi, H., & Liu, C. (2019). Domain adaptive motor fault diagnosis using deep transfer learning. IEEE Access, 7, 80937-80949.
[3] Finn, C., Abbeel, P., & Levine, S. (2017, July). Model-agnostic meta-learning for fast adaptation of deep networks. In International Conference on Machine Learning (pp. 1126-1135). PMLR.  
[4] Nichol, A., Achiam, J., & Schulman, J. (2018). On first-order meta-learning algorithms. arXiv preprint arXiv:1803.02999.  
[5] Santoro, A., Bartunov, S., Botvinick, M., Wierstra, D., & Lillicrap, T. (2016, June). Meta-learning with memory-augmented neural networks. In International conference on machine learning (pp. 1842-1850). PMLR.  
[6] Snell, J., Swersky, K., & Zemel, R. (2017, December). Prototypical networks for few-shot learning. In Proceedings of the 31st International Conference on Neural Information Processing Systems (pp. 4080-4090).  
[7] Sung, F., Yang, Y., Zhang, L., Xiang, T., Torr, P. H., & Hospedales, T. M. (2018). Learning to compare: Relation network for few-shot learning. In Proceedings of the IEEE conference on computer vision and pattern recognition (pp. 1199-1208).  
```
