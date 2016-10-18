This is a page contains all parameters in LightGBM.

## Parameters format

The parameter format is ```key1=value1 key2=value2 ... ``` . And parameters can be set both in config file and command line. By using command line, parameters should not have spaces before and after ```=```. By using config files, one line can only contains one parameter. you can using ```#``` to comment. If one parameter appears in both command line and config file, LightGBM will use the parameter in command line.

## Core Parameters

* ```config```, default=```""```, type=string, alias=```config_file```
  * path of config file
* ```task```, default=```train```, type=enum, options=```train```,```prediction```
  * ```train``` for training
  * ```prediction``` for prediction.
* ```application```, default=```regression```, type=enum, options=```regression```,```binary```,```lambdarank```, alias=```objective```,```app```
  * ```regression```, regression application
  * ```binary```, binary classification application 
  * ```lambdarank```, lambdarank application
* ```data```, default=```""```, type=string, alias=```train```,```train_data```
  * training data, LightGBM will train from this data
* ```valid```, default=```""```, type=multi-string, alias=```test```,```valid_data```,```test_data```
  * validation/test data, LightGBM will output metrics for these data
  * support multi validation data, separate by ```,```
* ```num_iterations```, default=```10```, type=int, alias=```num_iteration```,```num_tree```,```num_trees```,```num_round```,```num_rounds```
  * number of boosting iterations/trees
* ```learning_rate```, default=```0.1```, type=double, alias=```shrinkage_rate```
  * shrinkage rate
* ```num_leaves```, default=```127```, type=int, alias=```num_leaf```
  * number of leaves for one tree
* ```tree_learner```, default=```serial```, type=enum, options=```serial```,```feature```,```data```
  * ```serial```, single machine tree learner
  * ```feature```, feature parallel tree learner
  * ```data```, data parallel tree learner
  * Refer to [Parallel Learning Guide](https://github.com/Microsoft/LightGBM/wiki/Parallel-Learning-Guide) to get more details.
* ```num_threads```, default=OpenMP_default, type=int, alias=```num_thread```,```nthread```
  * Number of threads for LightGBM. 
  * For the best speed, set this to the number of **real CPU cores**, not the number of threads (most CPU using [hyper-threading](https://en.wikipedia.org/wiki/Hyper-threading) to generate 2 threads per CPU core).
  * For parallel learning, should not use full CPU cores since this will cause poor performance for network.

## Learning control parameters

* ```min_data_in_leaf```, default=```100```, type=int, alias=```min_data_per_leaf``` , ```min_data```
  * Minimal number of data in one leaf. can use this to deal with over-fit.
* ```min_sum_hessian_in_leaf```, default=```10.0```, type=double, alias=```min_sum_hessian_per_leaf```, ```min_sum_hessian```, ```min_hessian```
  * Minimal sum hessian in one leaf. Like ```min_data_in_leaf```, can use this to deal with over-fit.
* ```feature_fraction```, default=```1.0```, type=double, ```0.0 < feature_fraction < 1.0```, alias=```sub_feature```
  * LightGBM will random select part of features on each iteration if ```feature_fraction``` smaller than ```1.0```. For example, if set to ```0.8```, will select 80% features before training each tree.
  * Can use this to speed up training
  * Can use this to deal with over-fit
* ```feature_fraction_seed```, default=```2```, type=int
  * Random seed for feature fraction.
* ```bagging_fraction```, default=```1.0```, type=double, , ```0.0 < bagging_fraction < 1.0```, alias=```sub_row```
  * Like ```feature_fraction```, but this will random select part of data
  * can use this to speed up training
  * Can use this to deal with over-fit
  * Note: To enable bagging, should set ```bagging_freq``` to a non zero value as well
* ```bagging_freq```, default=```0```, type=int
  * Frequency for bagging, ```0``` means disable bagging. ```k``` means will perform bagging at every ```k``` iteration.
  * Note: To enable bagging, should set ```bagging_fraction``` as well
* ```bagging_seed``` , default=```3```, type=int
  * Random seed for bagging.


## IO parameters

* ```max_bin```, default=```255```, type=int
  * max number of bin that feature values will bucket in. Small bin may reduce training accuracy but may increase general power (deal with over-fit).
  * LightGBM will auto compress memory according ```max_bin```. For example, LightGBM will use ```uint8_t``` for feature value if ```max_bin=255```.
* ```data_random_seed```, default=```1```, type=int
  * random seed for data partition in parallel learning(not include feature parallel).
* ```output_model```, default=```LightGBM_model.txt```, type=string, alias=```model_output```,```model_out```
  * file name of output model in training.
* ```input_model```, default=```""```, type=string, alias=```model_input```,```model_in```
  * file name of input model.
  * for prediction task, will prediction data using this model.
  * for train task, will continued train from this model.
* ```output_result```, default=```LightGBM_predict_result.txt```, type=string, alias=```predict_result```,```prediction_result```
  * file name of prediction result in prediction task.
* ```is_sigmoid```, default=```true```, type=bool
  * Set to ```true``` will use sigmoid(if needed, only effect for ```binary``` now) transform for prediction result.
  * Set to ```false``` will only predict the raw scores.
* ```init_score```, default=```""```, type=string, alias=```input_init_score```
  * file name of initial score file. LightGBM will use this score to start training.
  * only support train task.
  * each line contains one score corresponding with data
* ```is_pre_partition```, default=```false```, type=bool
  * used for parallel learning(not include feature parallel).
  * ```true``` if training data is pre-partitioned, and different machines using different partition.
* ```is_sparse```, default=```true```, type=bool, alias=```is_enable_sparse```
  * used to enable/disable sparse optimization. Set to ```false``` to disable sparse optimization.
* ```two_round```, default=```false```, type=bool, alias=```two_round_loading```,```use_two_round_loading```
  * by default, LightGBM will map data file to memory and load features from memory. This will provide faster data loading speed. But it may out of memory when data file is very big.
  * set this to ```true``` if data file is too big to fit in memory.
* ```save_binary```, default=```false```, type=bool, alias=```is_save_binary```,```is_save_binary_file```
  * set this to ```true``` will save data set(include validation data) to binary file. speed up the data loading speed for the next time.

## Objective parameters

* ```sigmoid```, default=```1.0```, type=double
  * parameter for sigmoid function. Will be used in binary classification and lambdarank.
* ```is_unbalance```, default=```false```, type=bool
  * used in binary classification. Set this to ```true``` if training data is unbalance.
* ```max_position```, default=```20```, type=int
  * used in lambdarank, will optimize NDCG at this position.
* ```label_gain```, default=```{0,1,3,7,15,31,63,...}```, type=multi-double
  * used in lambdarank, relevant gain for labels. For example, the gain of label ```2``` is ```3``` if using default label gains.
  * Separate by ```,```

## Metric parameters

* ```metric```, default={```l2``` for regression}, {```binary_logloss``` for binary classification},{```ndcg``` for lambdarank}, type=multi-enum, options=```l1```,```l2```,```ndcg```,```auc```,```binary_logloss```,```binary_error```
  * ```l1```, absolute loss
  * ```l2```, square loss
  * ```ndcg```, [NDCG](https://en.wikipedia.org/wiki/Discounted_cumulative_gain#Normalized_DCG)
  * ```auc```, [AUC](https://en.wikipedia.org/wiki/Area_under_the_curve_(pharmacokinetics))
  * ```binary_logloss```, [log loss](https://www.kaggle.com/wiki/LogarithmicLoss)
  * ```binary_error```. For one sample ```0``` for correct classification, ```1``` for error classification.
  * Support multi metrics, separate by ```,```
* ```metric_freq```, default=```1```, type=int
  * frequency for metric output
* ```is_training_metric```, default=```false```, type=bool
  * set this to true if need to output metric result for training
* ```ndcg_at```, default=```{1,2,3,4,5}```, type=multi-int, alias=```ndcg_eval_at```
  * NDCG evaluation position, separate by ```,```

## Network parameters

Following parameters are used for parallel learning, and only used for base(socket) version. It is not need to set them for MPI version. 

* ```num_machines```, default=```1```, type=int, alias=```num_machine```
  * Used for parallel learning, the number of machines for parallel learning application
* ```local_listen_port```, default=```12400```, type=int, alias=```local_port```
  * TCP listen port for local machines.
  * Should allow this port in firewall setting before training.
* ```time_out```, default=```120```, type=int
  * Socket time-out in minutes.
* ```machine_list_file```, default=```""```, type=string
  * File that list machines for this parallel learning application
  * Each line contains one ip and one port for one machine. Format is ```ip port```, separate by space.

## Tuning Parameters

### For faster speed

* Use bagging by set ```bagging_fraction``` and ```bagging_freq``` 
* Use feature sub-sampling by set ```feature_fraction```
* Use small ```max_bin```
* Use ```save_binary``` to speed up data loading in future learning
* Use parallel learning, refer to [parallel learning guide](https://github.com/Microsoft/LightGBM/wiki/Parallel-Learning-Guide).

### For better accuracy

* Use large ```max_bin``` (may slower)
* Use small ```learning_rate``` with large ```num_iterations```
* Use large ```num_leave```(may over-fitting)
* Use bigger training data

### Deal with over-fitting

* Use small ```max_bin```
* Use small ```num_leave```
* Use ```min_data_in_leaf``` and ```min_sum_hessian_in_leaf```
* Use bagging by set ```bagging_fraction``` and ```bagging_freq``` 
* Use feature sub-sampling by set ```feature_fraction```
* Use bigger training data

## Others

### Weight data
LightGBM support weighted training. It use an additional file to store weight data, like the following:

```
1.0
0.5
0.8
...
```

It means the weight of first data is ```1.0```, second is ```0.5```, and so on. The weight file is corresponded with training data file line by line, and has per weight per line. And if the name of data file is "train.txt", the weight file should be named as "train.txt.weight" and in same folder as the data file. And LightGBM will auto load weight file if it exists.

### Query data

For LambdaRank learning, it needs query information for training data. LightGBM use an additional file to store query data. Following is an example:

```
27
18
67
...
```

It means the first ```27``` data belong one query and next ```18``` belong another query, and so on.(**Note: data should order by query**) If name of data file is "train.txt", the query file should be named as "train.txt.query" and in same folder as the data file. And LightGBM will auto load query file if it exists.