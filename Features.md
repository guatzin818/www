This is a short introduction for the features and algorithms used in LightGBM. 

This page doesn't contain detailed algorithms, please refer to cited paper or source code if you are interested.

## Optimization in speed and memory usage

Many boosting tools use pre-sorted based algorithms<sup>[1][2]</sup>(e.g. default algorithm in xgboost) for decision tree learning. It is a simple solution, but not easy to optimize.

LightGBM uses the histogram based algorithms<sup>[3][4]</sup>, which bucketing continuous feature(attribute) values into discrete bins, to speed up training procedure and reduce memory usage. Following are advantages for histogram based algorithms:

* **Reduce calculation cost of split gain**
    * Pre-sorted based algorithms need O(#data) times calculation
    * Histogram based algorithms only need to calculate O(#bins) times, and #bins is far smaller than #data
        * It still needs O(#data) times to construct histogram, which only contain sum-up operation
* **Only need to split data one time after finding best split point**
    * Pre-sorted based algorithms need to split data O(#features) times (since different features access data in different order)
* **Use histogram subtraction for further speed-up**
    * To get one leaf's histograms in a binary tree, can use the histogram subtraction of its parent and its neighbor 
    * So it only need to construct histograms for one leaf (with smaller #data than its neighbor), then can get histograms of its neighbor by histogram subtraction with small cost( O(#bins) )
* **Reduce Memory usage**
    * Can replace continuous values to discrete bins. If #bins is small, can use small data type, e.g. uint8_t, to store training data
    * No need to store additional information for pre-sorting feature values
* **Reduce communication cost for parallel learning**

## Sparse optimization
  * Only need O(#non_zero_data) to construct histogram for sparse features

## Optimization in accuracy

Most decision tree learning algorithms grow tree by level(depth)-wise, like the following image:

[[image/level_wise.png]]

LightGBM grows tree by leaf-wise. It will choose the leaf with max delta loss to grow. 
When growing same #leaf, Leaf-wise algorithm can reduce more loss than level-wise algorithm.

[[image/leaf_wise.png]]

## Optimization in network communication

It only needs to use some collective communication algorithms, like "All reduce", "All gather" and "Reduce scatter", in parallel learning of LightGBM. LightGBM implement state-of-art algorithms described in this [paper](http://wwwi10.lrr.in.tum.de/~gerndt/home/Teaching/HPCSeminar/mpich_multi_coll.pdf)<sup>[5]</sup>. These collective communication algorithms can provide much better performance than point-to-point communication.

## Optimization in parallel learning

LightGBM provides following parallel learning algorithms. 

### Feature Parallel

#### Traditional algorithm
Feature parallel aim to parallel the "Find Best Split" in the decision tree. The procedure of traditional feature parallel is:

1. Partition data vertically (different machines have different feature set)
2. Workers find local best split point {feature, threshold} on local feature set
3. Communicate local best splits with each other and get the best one
4. Worker with best split to perform split, then send the split result of data to other workers
5. Other workers split data according received data

The shortage of traditional feature parallel:

* Has computation overhead, since it cannot speed up "split", whose time complexity is O(#data). Thus, feature parallel cannot speed up well when #data is large.
* Need communication of split result, which cost about O(#data/8) (one bit for one data). 

#### Feature parallel in LightGBM

Since feature parallel cannot speed up well when #data is large, we make a little change here: instead of partitioning data vertically, every worker holds the full data. Thus, LightGBM doesn't need to communicate for split result of data since every worker know how to split data. And #data won't be larger, so it is reasonable to hold full data in every machine.

The procedure of feature parallel in LightGBM:

1. Workers find local best split point{feature, threshold} on local feature set
2. Communicate local best splits with each other and get the best one
3. Perform best split

However, this feature parallel algorithm still suffers from computation overhead for "split" when #data is large. So it will be better to use data parallel when #data is large.

### Data Parallel

#### Traditional algorithm
Data parallel aim to parallel the whole decision learning. The procedure of data parallel is:

1. Partition data horizontally 
2. Workers use local data to construct local histograms 
3. Merge global histograms from all local histograms.
4. Find best split from merged global histograms, then perform splits

The shortage of traditional data parallel:

* High communication cost. If using point-to-point communication algorithm, communication cost for one machine is about O(#machine * #feature * #bin). If using collective communication algorithm (e.g. "All Reduce"), communication cost is about O(2 * #feature * #bin) ( check cost of "All Reduce" in chapter 4.5 at this [paper](http://wwwi10.lrr.in.tum.de/~gerndt/home/Teaching/HPCSeminar/mpich_multi_coll.pdf) ).

#### Data parallel in LightGBM

We reduce communication cost of data parallel in LightGBM:

1. Instead of "Merge global histograms from all local histograms", LightGBM use "Reduce Scatter" to merge histograms of different(non-overlapping) features for different workers. Then workers find local best split on local merged histograms and sync up global best split. 
2. As aforementioned, LightGBM use histogram subtraction to speed up training. Based on this, we can communicate histograms only for one leaf, and get its neighbor's histograms by subtraction as well.

Above all, we reduce communication cost to O(0.5 * #feature* #bin) for data parallel in LightGBM.



## Applications and metrics

Support following application:

* regression, the objective function is L2 loss
* binary classification, the objective function is logloss
* multi classification
* lambdarank, the objective function is lambdarank with NDCG

Support following metrics:

* L1 Loss
* L2 Loss
* Log loss
* Classification Error rate
* AUC
* NDCG
* Multi class log loss
* Multi class error rate

For more details, please refer to [Configuration](https://github.com/Microsoft/LightGBM/wiki/Configuration).

## Other features
* Limit max_depth of tree while grows tree leaf-wise
* [DART](https://arxiv.org/abs/1505.01866)
* L1/L2 regularization
* Bagging
* Column(feature) sub-sample
* Continued train with input GBDT model
* Continued train with the input score file
* Weighted training
* Validation metric output during training
* Multi validation data
* Multi metrics
* Early stopping
* Prediction for leaf index

For more details, please refer to [Configuration](https://github.com/Microsoft/LightGBM/wiki/Configuration).

## Future Plan

* More languages (e.g. Python, R) support
* More platforms (e.g. Hadoop, Spark) support
* Directly use for categorical features (Finished 12/5/2016)

## References
[1] Mehta, Manish, Rakesh Agrawal, and Jorma Rissanen. "SLIQ: A fast scalable classifier for data mining." International Conference on Extending Database Technology. Springer Berlin Heidelberg, 1996.

[2] Shafer, John, Rakesh Agrawal, and Manish Mehta. "SPRINT: A scalable parallel classifier for data mining" Proc. 1996 Int. Conf. Very Large Data Bases. 1996.

[3] Ranka, Sanjay, and V. Singh. "CLOUDS: A decision tree classifier for large datasets." Proceedings of the 4th Knowledge Discovery and Data Mining Conference. 1998.

[4] Machado, F. P. "Communication and memory efficient parallel decision tree construction." (2003).

[5] Thakur, Rajeev, Rolf Rabenseifner, and William Gropp. "Optimization of collective communication operations in MPICH." International Journal of High Performance Computing Applications 19.1 (2005): 49-66.
