This is a quick start guide for LightGBM.

Follow the [Installation Guide](https://github.com/Microsoft/LightGBM/wiki/Installation-Guide) to install LightGBM first.

## Training data format 

LightGBM supports input data file with [CSV](https://en.wikipedia.org/wiki/Comma-separated_values), [TSV] (https://en.wikipedia.org/wiki/Tab-separated_values) and [LibSVM](https://www.csie.ntu.edu.tw/~cjlin/libsvm/) formats.

Label is the data of first column, and there is no header in the file.

LightGBM also support weighted training, it needs an additional [weight data](https://github.com/Microsoft/LightGBM/wiki/Configuration#weight-data). And it needs an additional [query data](https://github.com/Microsoft/LightGBM/wiki/Configuration#query-data) for ranking task.

update 11/3/2016:

1. support input with header now
2. can specific label column, weight column and query/group id column. Both index and column are supported
3. can specific a list of ignored columns

For the detailed usage, please refer to [Configuration](https://github.com/Microsoft/LightGBM/wiki/Configuration).


## Parameter quick look

The parameter format is ```key1=value1 key2=value2 ... ``` . And parameters can be in both config file and command line.

Some important parameters:

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
  * number of leaves in one tree
* ```tree_learner```, default=```serial```, type=enum, options=```serial```,```feature```,```data```
  * ```serial```, single machine tree learner
  * ```feature```, feature parallel tree learner
  * ```data```, data parallel tree learner
  * Refer to [Parallel Learning Guide](https://github.com/Microsoft/LightGBM/wiki/Parallel-Learning-Guide) to get more details.
* ```num_threads```, default=OpenMP_default, type=int, alias=```num_thread```,```nthread```
  * Number of threads for LightGBM. 
  * For the best speed, set this to the number of **real CPU cores**, not the number of threads (most CPU using [hyper-threading](https://en.wikipedia.org/wiki/Hyper-threading) to generate 2 threads per CPU core).
* ```min_data_in_leaf ```, default=```100```, type=int
  * number of minimal data for one leaves, an important parameter to avoid over-fit

For all parameters, please refer to [Parameters](https://github.com/Microsoft/LightGBM/wiki/Configuration).


## Run LightGBM

For Windows:
```
lightgbm.exe config=your_config_file other_args ...
```

For unix:
```
./lightgbm config=your_config_file other_args ...
```

Parameters can be both in the config file and command line, and the parameters in command line have higher priority than in config file.
For example, following command line will keep 'num_trees=10' and ignore same parameter in config file.
```
./lightgbm config=train.conf num_trees=10
```

## Examples

* [Binary Classifiaction](https://github.com/Microsoft/LightGBM/tree/master/examples/binary_classification)
* [Regression](https://github.com/Microsoft/LightGBM/tree/master/examples/regression)
* [Lambdarank](https://github.com/Microsoft/LightGBM/tree/master/examples/lambdarank)
* [Parallel Learning](https://github.com/Microsoft/LightGBM/tree/master/examples/parallel_learning)
