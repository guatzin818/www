## Comparison Experiment

### Experiment Data

We use 3 data set to conduct our comparison experiments. Details of data are list in the following table:

| Data     |      Task     |  Link | #Data | #Feature| Comments|
|----------|---------------|-------|-------|---------|---------|
| Higgs    |  Binary classification | [link](https://archive.ics.uci.edu/ml/datasets/HIGGS) |10,000,000|28| use last 500,000 samples as test set  | 
| Yahoo LTR|  Learning to rank      | [link](https://webscope.sandbox.yahoo.com/catalog.php?datatype=c)  |~2,000,000|137| 	   set1.train as train, set1.test as test |
| MS LTR   |  Learning to rank      | [link](http://research.microsoft.com/en-us/projects/mslr/)|~473,000|700| {S1,S2,S3} as train set, {S5} as test set |

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
 ```

3. LightGBM:
 ```
 learning_rate = 0.1
 num_leaves = 255
 num_trees = 500
 num_threads = 16
 ```

xgboost with ```max_depth=8``` will have max number leaves to 255. This has same model complexity as LightGBM with ```num_leves=255```. And xgboost_approx with ```sketch_eps=0.004``` will have #bins to 250, which is similar to default(255) in LightGBM.

Other parameters are default values.

### Result

#### Speed

For speed comparison, we only run the training task, which is without any test or metric output. And we don't count the time for IO.

Following table is the comparison of time cost:

| Data      |  xgboost| xgboost_approx |  LightGBM|  
|-----------|---------|----------------|----------|
| Higgs     | 4445s   |     2206s      | **386s** | 
| Yahoo LTR | 844s    |     591s       | **176s** | 
| MS LTR    | 1374s   |     1233s      | **268s** |

[[image/time_cost.png]]

We found LightGBM is faster than xgboost on all experiment data sets. 

#### Accuracy

For accuracy comparison, we use the accuracy on test data set to have a fair comparison.

Higgs's AUC:

| Metric  | xgboost | xgboost_approx | LightGBM|  
|---------|---------|----------------|---------|
| AUC     | 0.8393  | 0.8402 | **0.8450**  | 

NDCG at Yahoo LTR:

| Metric    | xgboost | xgboost_approx| LightGBM|  
|-----------|---------|-------------- |---------|
| NDCG@1    | 0.7247  | 0.7272 | **0.7323**  | 
| NDCG@3    | 0.7282  | 0.7278 | **0.7368**  | 
| NDCG@5    | 0.7463  | 0.7465 | **0.7560**  | 
| NDCG@10   | 0.7878  | 0.7879 | **0.7969**  | 

NDCG at MS LTR:

| Metric    | xgboost | xgboost_approx| LightGBM|  
|-----------|---------|-------------- |---------|
| NDCG@1    | 0.4994  | 0.4959 | **0.5182**  | 
| NDCG@3    | 0.4817  | 0.4813 | **0.5042**  | 
| NDCG@5    | 0.4860  | 0.4861 | **0.5085**  | 
| NDCG@10   | 0.5051  | 0.5037 | **0.5265**  | 

We found LightGBM has better accuracy than xgboost on all experiment data sets.

#### Memory consumption

We monitor ```RES``` while running training task. And we set ```two_round=true``` in LightGBM to reduce peak memory usage.

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

