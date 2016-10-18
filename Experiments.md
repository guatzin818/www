## Comparison Experiment

### Experiment Data

We use 3 data set to conduct our comparison experiments. Details of data are list in the following table:

| Data     |      Task     |  Link | #Train_Set | #Feature| Comments|
|----------|---------------|-------|-------|---------|---------|
| Higgs    |  Binary classification | [link](https://archive.ics.uci.edu/ml/datasets/HIGGS) |10,500,000|28| use last 500,000 samples as test set  | 
| Yahoo LTR|  Learning to rank      | [link](https://webscope.sandbox.yahoo.com/catalog.php?datatype=c)  |2,270,296|137| 	   set1.train as train, set1.test as test |
| MS LTR   |  Learning to rank      | [link](http://research.microsoft.com/en-us/projects/mslr/)|473,134|700| {S1,S2,S3} as train set, {S5} as test set |

### Environment

We use one Linux server as experiment platform, details are listed in the following table:

| OS     |      CPU     |  Memory | 
|--------|--------------|---------|
| Ubuntu 14.04 LTS  |  2 * E5-2680 v2 | DDR3 1600Mhz, 256GB|

### Baseline

we use [xgboost](https://github.com/dmlc/xgboost) as baseline, and build version is latest version at 8 OCT 2016 [f9648ac](https://github.com/dmlc/xgboost/tree/f9648ac320ba9d9fb77c1b9bf091406b9b6b4086).

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

xgboost grows tree depth-wise and controls model complexiy by ```max_depth```. LightGBM uses leaf-wise algorithm instead and controls model complexity by ```num_leaves```. So we cannot compare them in the exact same model setting. For the tradeoff, we use xgboost with ```max_depth=8```, which will have max number leaves to 255, to compare with LightGBM with ```num_leves=255```. And xgboost_approx with ```sketch_eps=0.004``` will have #bins to 250, which is similar to default(255) in LightGBM.

Other parameters are default values.

### Result

#### Speed

For speed comparison, we only run the training task, which is without any test or metric output. And we don't count the time for IO.

Following table is the comparison of time cost:

| Data      |  xgboost| xgboost_approx |  LightGBM|  
|-----------|---------|----------------|----------|
| Higgs     | 4543s   |     2321s      | **383s** | 
| Yahoo LTR | 702s    |     473s       | **176s** | 
| MS LTR    | 1323s   |     1024s      | **265s** |

[[image/time_cost.png]]

We found LightGBM is faster than xgboost on all experiment data sets. 

#### Accuracy

For accuracy comparison, we use the accuracy on test data set to have a fair comparison.

Higgs's AUC:

| Metric  | xgboost | xgboost_approx | LightGBM|  
|---------|---------|----------------|---------|
| AUC     | 0.8395  | 0.8405 | **0.8451**  | 

NDCG at Yahoo LTR:

| Metric    | xgboost | xgboost_approx| LightGBM|  
|-----------|---------|-------------- |---------|
| NDCG@1    | 0.7187  | 0.7180 | **0.7287**  | 
| NDCG@3    | 0.7175  | 0.7277 | **0.7381**  | 
| NDCG@5    | 0.7377  | 0.7380 | **0.7557**  | 
| NDCG@10   | 0.7804  | 0.7809 | **0.7961**  | 

NDCG at MS LTR:

| Metric    | xgboost | xgboost_approx| LightGBM|  
|-----------|---------|-------------- |---------|
| NDCG@1    | 0.4825  | 0.4824 | **0.5182**  | 
| NDCG@3    | 0.4662  | 0.4685 | **0.5041**  | 
| NDCG@5    | 0.4720  | 0.4735 | **0.5092**  | 
| NDCG@10   | 0.4914  | 0.4925 | **0.5259**  | 

We found LightGBM has better accuracy than xgboost on all experiment data sets.

#### Memory consumption

We monitor ```RES``` while running training task. And we set ```two_round=true``` (Will increase data-loading time, but reduce peak memory usage, not affect training speed or accuracy) in LightGBM to reduce peak memory usage. 

| Data      | xgboost | xgboost_approx| LightGBM|  
|-----------|---------|-------------- |---------|
| Higgs     | 4.853g  | 4.875g | **0.822g** | 
| Yahoo LTR | 1.907g  | 2.221g | **0.831g** | 
| MS LTR    | 5.469g  | 5.600g | **0.745g** |

LightGBM benefits from its histogram optimization algorithm, so it consumes much lower memories.

## Parallel Experiment

### Data

We use a terabyte click log dataset to conduct parallel experiments. Details are list in following table:

| Data     |      Task     |  Link | #Data | #Feature|
|----------|---------------|-------|-------|---------|
| Criteo    |  Binary classification | [link](http://labs.criteo.com/downloads/download-terabyte-click-logs/) |1,700,000,000|67|

This data contains 13 integer features and 26 category features of 24 days click log. We statistic the CTR and count for these 26 category features from first ten days, then use next ten days’ data, which had been replaced the category features by the corresponding CTR and count, as training data. The processed training data has total 1.7 billions records and 67 features.


### Environment
We use 16 windows servers as experiment platform, details are list in following table:

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