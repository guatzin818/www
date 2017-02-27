update 02/21/2017: add xgboost_hist result. 


## Comparison Experiment


For the detailed experiment scripts and output logs, please refer to this [repo](https://github.com/guolinke/boosting_tree_benchmarks). 

### Experiment Data

We use 4 datasets to conduct our comparison experiments. Details of data are listed in the following table:

| Data     |      Task     |  Link | #Train_Set | #Feature| Comments|
|----------|---------------|-------|-------|---------|---------|
| Higgs    |  Binary classification | [link](https://archive.ics.uci.edu/ml/datasets/HIGGS) |10,500,000|28| use last 500,000 samples as test set  | 
| Yahoo LTR|  Learning to rank      | [link](https://webscope.sandbox.yahoo.com/catalog.php?datatype=c)  	|473,134|700|   set1.train as train, set1.test as test |
| MS LTR   |  Learning to rank      | [link](http://research.microsoft.com/en-us/projects/mslr/) |2,270,296|137| {S1,S2,S3} as train set, {S5} as test set |
| Expo     |  Binary classification | [link](http://stat-computing.org/dataexpo/2009/) |11,000,000|700| use last 1,000,000 as test set |
| Allstate |  Binary classification | [link](https://www.kaggle.com/c/ClaimPredictionChallenge) |13,184,290|4228| use last 1,000,000 as test set |


### Environment

We use one Linux server as experiment platform, details are listed in the following table:

| OS     |      CPU     |  Memory | 
|--------|--------------|---------|
| Ubuntu 14.04 LTS  |  2 * E5-2670 v3 | DDR4 2133Mhz, 256GB|

### Baseline

We use [xgboost](https://github.com/dmlc/xgboost) as baseline.

Both xgboost and LightGBM are built with OpenMP support.


### Settings

We set up total 3 settings for experiments, the parameters of these settings are listed in the following:

1. xgboost:
 ```
 eta = 0.1
 max_depth = 8
 num_round = 500
 nthread=16
 tree_method=exact
 min_child_weight=100
 ```

2. xgboost_hist (using histogram based algorithm):
 ```
 eta = 0.1
 num_round = 500
 nthread=16
 tree_method=approx
 min_child_weight=100
 tree_method=hist
 grow_policy=lossguide 
 max_depth=0 
 max_leaves=255
 ```

3. LightGBM:
 ```
 learning_rate = 0.1
 num_leaves = 255
 num_trees = 500
 num_threads = 16
 min_data_in_leaf=0
 min_sum_hessian_in_leaf=100
 ```

xgboost grows tree depth-wise and controls model complexity by ```max_depth```. LightGBM uses leaf-wise algorithm instead and controls model complexity by ```num_leaves```. So we cannot compare them in the exact same model setting. For the tradeoff, we use xgboost with ```max_depth=8```, which will have max number leaves to 255, to compare with LightGBM with ```num_leves=255```. 

Other parameters are default values. 

### Result

#### Speed

For speed comparison, we only run the training task, which is without any test or metric output. And we don't count the time for IO.

The following table is the comparison of time cost:



| Data      |  xgboost | xgboost_hist |  LightGBM|
|----|  ----| ----- | ----|
| Higgs|3794.34 s |551.898 s |238.505513 s |
| Yahoo LTR|674.322 s |265.302 s |150.18644 s |
| MS LTR|1251.27 s |385.201 s |215.320316 s |
| Expo|1607.35 s |588.253 s |138.504179 s |
| Allstate|2867.22 s |1355.71 s |348.084475 s |



We found LightGBM is faster than xgboost on all experiment data sets. 

#### Accuracy

For accuracy comparison, we use the accuracy on test data set to have a fair comparison.


Higgs's AUC:

| Metric      |  xgboost| xgboost_hist |  LightGBM|
|----|  ----| ---- | ----|
| AUC|0.839593|0.845605|0.845154|


ndcg at Yahoo LTR:

| Metric      |  xgboost | xgboost_hist |  LightGBM|
|----|  ----| ---- | ----|
| ndcg@1|0.719748|0.720223|0.732466|
| ndcg@3|0.717813|0.721519|0.738048|
| ndcg@5|0.737849|0.739904|0.756548|
| ndcg@10|0.78089|0.783013|0.796818|


ndcg at MS LTR:

| Metric      |  xgboost | xgboost_hist |  LightGBM|
|----|  ----| ---- | -----|
| ndcg@1|0.483956|0.488649|0.524255|
| ndcg@3|0.467951|0.473184|0.505327|
| ndcg@5|0.472476|0.477438|0.510007|
| ndcg@10|0.492429|0.496967|0.527371|


auc at Expo:

| Metric      |  xgboost | xgboost_hist |  LightGBM|
|----|  ----| ---- |  ----|
| auc|0.756713|0.777777|0.777543|


auc at Allstate:

| Metric      |  xgboost | xgboost_hist |  LightGBM|
|----|  ----| ---- |  ----|
| auc|0.607201|0.609042|0.609167|


#### Memory consumption

We monitor ```RES``` while running training task. And we set ```two_round=true``` (Will increase data-loading time, but reduce peak memory usage, not affect training speed or accuracy) in LightGBM to reduce peak memory usage. 

| Data      | xgboost | xgboost_hist| LightGBM|  
|-----------|---------|-------------- |---------|
| Higgs     | 4.853GB  | 3.784GB | **0.868GB** | 
| Yahoo LTR | 1.907GB  | 1.468GB | **0.831GB** | 
| MS LTR    | 5.469GB  | 3.654GB | **0.886GB** |
| Expo      | 1.553GB  | 1.393GB | **0.543GB** |
| Allstate  | 6.237GB  | 4.990GB | **1.027GB** |


## Parallel Experiment

### Data

We use a terabyte click log dataset to conduct parallel experiments. Details are listed in following table:

| Data     |      Task     |  Link | #Data | #Feature|
|----------|---------------|-------|-------|---------|
| Criteo    |  Binary classification | [link](http://labs.criteo.com/downloads/download-terabyte-click-logs/) |1,700,000,000|67|

This data contains 13 integer features and 26 category features of 24 days click log. We statistic the CTR and count for these 26 category features from the first ten days, then use next ten days’ data, which had been replaced the category features by the corresponding CTR and count, as training data. The processed training data hava total 1.7 billions records and 67 features.


### Environment
We use 16 windows servers as experiment platform, details are listed in following table:

| OS     |      CPU     |  Memory | Network Adapter |
|--------|--------------|---------|-----------------|
| Windows Server 2012 |  2 * E5-2670 v2 | DDR3 1600Mhz, 256GB| Mellanox ConnectX-3, 54Gbps, RDMA support |

### Settings

```
learning_rate = 0.1
num_leaves = 255
num_trees=100
num_thread=16
tree_learner=data
```
We use data parallel here, since this data is large in #data but small in #feature.

Other parameters are default values.

### Result

|  #machine | time per tree | memory usage(per machine) |
|-----------|---------|--------------|
| 1   | 627.8s  | 176GB |
| 2   | 311s    | 87GB  |
| 4   | 156s  | 43GB  |
| 8   | 80s   | 22GB  |
| 16  | 42s   | 11GB  |

From the results, we find LightGBM perform linear speed up in parallel learning. 