## Comparison Experiment

For the detailed experiment scripts and output logs, please refer to this [repo](https://github.com/guolinke/boosting_tree_benchmarks). 

### Experiment Data

We use 3 data set to conduct our comparison experiments. Details of data are listed in the following table:

| Data     |      Task     |  Link | #Train_Set | #Feature| Comments|
|----------|---------------|-------|-------|---------|---------|
| Higgs    |  Binary classification | [link](https://archive.ics.uci.edu/ml/datasets/HIGGS) |10,500,000|28| use last 500,000 samples as test set  | 
| Yahoo LTR|  Learning to rank      | [link](https://webscope.sandbox.yahoo.com/catalog.php?datatype=c)  	|473,134|700|   set1.train as train, set1.test as test |
| MS LTR   |  Learning to rank      | [link](http://research.microsoft.com/en-us/projects/mslr/) |2,270,296|137| {S1,S2,S3} as train set, {S5} as test set |

### Environment

We use one Linux server as experiment platform, details are listed in the following table:

| OS     |      CPU     |  Memory | 
|--------|--------------|---------|
| Ubuntu 14.04 LTS  |  2 * E5-2680 v2 | DDR3 1600Mhz, 256GB|

### Baseline

We use [xgboost](https://github.com/dmlc/xgboost) as baseline, and build version is latest version at 27 OCT 2016 [016ab89](https://github.com/dmlc/xgboost/tree/016ab89484e7a6313a7de11160d0c3370fc5c35d).

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

2. xgboost_approx (using histogram based algorithm):
 ```
 eta = 0.1
 max_depth = 8
 num_round = 500
 nthread=16
 # num_bins = (1/sketch_eps)
 sketch_eps=0.004
 tree_method=approx
 min_child_weight=100
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

xgboost grows tree depth-wise and controls model complexity by ```max_depth```. LightGBM uses leaf-wise algorithm instead and controls model complexity by ```num_leaves```. So we cannot compare them in the exact same model setting. For the tradeoff, we use xgboost with ```max_depth=8```, which will have max number leaves to 255, to compare with LightGBM with ```num_leves=255```. And xgboost_approx with ```sketch_eps=0.004``` will have #bins to 250, which is similar to default(255) in LightGBM.

Other parameters are default values. 

### Result

#### Speed

For speed comparison, we only run the training task, which is without any test or metric output. And we don't count the time for IO.

The following table is the comparison of time cost:

| Data      |  xgboost| xgboost_approx |  LightGBM|
|----|  ----| ---- |  ----|
| Higgs|4604.09s |2142.72s |**360.77s** |
| Yahoo LTR|704.925s |497.467s |**173.39s**|
| MS LTR|1338.28s |1046.48s |**263.51s**|

[[image/time_cost.png]]

We found LightGBM is faster than xgboost on all experiment data sets. 

#### Accuracy

For accuracy comparison, we use the accuracy on test data set to have a fair comparison.

Higgs's AUC:

| Metric      |  xgboost| xgboost_approx |  LightGBM|
| ----------- |  -------| -------------- |  --------|
| AUC|0.839528|0.840533|**0.845123**|

NDCG at Yahoo LTR:

| Metric      |  xgboost| xgboost_approx |  LightGBM|
| ----------- |  -------| -------------- |  --------|
| NDCG@1|0.721213|0.720535|**0.731828**|
| NDCG@3|0.721025|0.719632|**0.738793**|
| NDCG@5|0.740316|0.739363|**0.756369**|
| NDCG@10|0.782226|0.781775|**0.796572**|


NDCG at MS LTR:


| Metric      |  xgboost| xgboost_approx |  LightGBM|
| ----------- |  -------| -------------- |  --------|
| NDCG@1|0.488128|0.480611|**0.521854**|
| NDCG@3|0.472706|0.470275|**0.504671**|
| NDCG@5|0.476245|0.474441|**0.510153**|
| NDCG@10|0.495091|0.493346|**0.52666**|

We found LightGBM has better accuracy than xgboost on all experiment data sets.

#### Memory consumption

We monitor ```RES``` while running training task. And we set ```two_round=true``` (Will increase data-loading time, but reduce peak memory usage, not affect training speed or accuracy) in LightGBM to reduce peak memory usage. 

| Data      | xgboost | xgboost_approx| LightGBM|  
|-----------|---------|-------------- |---------|
| Higgs     | 4.853GB  | 4.875GB | **0.822GB** | 
| Yahoo LTR | 1.907GB  | 2.221GB | **0.831GB** | 
| MS LTR    | 5.469GB  | 5.600GB | **0.745GB** |

LightGBM benefits from its histogram optimization algorithm, so it consumes much lower memory.

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