# GS Quantify 2016
Questions attempted in GS Quantify conducted by Goldman Sachs

--------

### Question 1 : Job Scheduler

#### Assumptions:
1. Priority of no 2 jobs will ever be equal (given).

#### Data Structures Used:
The system has the following data structures:
1. A map of `unique_id` to each job.
	* Efficient lookup by `unique_id` (assigned by us).
	* An binary tree was used instead of a hash-map to avoid performance degradation due to the nature of memory (paging, cache locality).

2. A Bi-directional map to store instructions and originating string and an identifier integer, for efficient access from both key and value.
   	* A hash-map is used instead of a binary tree map to avoid overheads and because the size of the tree is given to not be too large. 
3. A priority queue of jobs currently waiting in the queue.
	* Build-in abstract priority_queue is a heap, sorted in the order of increasing priority.
 	* Best data structure for a priority queue, O(1) lookup, O(lg n) insertion & deletion.
4. A priority queue of timestamps of busy CPUs (the timestamp of when they are supposed to be finished processing), another min-heap.
5. A vector of (`time_of_arrival`, `unique_id`) corresponding to each job.
 	* Insertion is performed when a new job is added.
 	* The vector is sorted by `timestamp`, search can be done in O(log n).
6. A vector of (`timestamp`, `state_of_queue`), which stores the state of queue corresponding to the timestamp at each "assign" operation.
 	* The state is stored only if the time difference between the new assign statement and the last assign is greater than a fixed interval of time.
 	* Since the insertion is done in an ordered manner according to timestamp, lookup can be done in O(log n). 
7. A vector of (`timestamp`, `num_free_CPUs`).
 	* It stores the number of free CPUs available at the time of each assign statement call.
8. Vectors are used here because the order of timestamp of "assign" and "job" commands is guranteed to be non-deceasing, therefore insertion can be done is O(1) and lookup in O(log n) which is very-efficient due to having almost no overhead due to container use.

The (`timestamp`, `unique_id`) and (`timestamp`, `num_free_cpus`) is merged into one vector for convenience of processing queries.

#### Algorithm:
	 "job" command:    the job is inserted into the map, the job vector and the job queue.
	 "assign" command: the free CPUs are collected, min(k, f, s) jobs are popped from the job queue and assigned. The state of the job queue at this timestamp is stored if the difference of timestamp exceeds a given limit. Otherwise, only the number of free CPUs is stored.
	 "query" command:  for a given timestamp, the job queue history vector is searched for the timestamp just lesser than the one required, then the state is evaluated and the required jobs is calculated based on how many jobs lie in the obtained time interval.

#### Space Complexity:
	 No. of jobs = j
	 No. of assign commands = k
	 No. of CPUs = n
	
Space Complexity = **O(n + kj)**

#### Time Complexity:
	 No. of jobs = j
	 No. of assign commands = k
	 No. of CPUs = n

"job" : **O(log j)**

"assign" : **O(n + jlog j)**

"query \<k\>" : **O(log k + jlog j)**

"query \<orig\>" : **O(log k + jlog j)**

--------

### QUestion 2 : Bond Liquidity Prediction

Historical data for 3 month time period is given. But, upon analyzing the data, we realized that the minimum date for each transaction takes place is 16th March 2016 and maximum date for which it happens is 9th June 2016. So, effectively time duration for which data is given is 86 days.

* First of all, we converted nominal features to integer values, i.e., **industryGroup10** was
converted to 10 as it should be evaluated by the machine learning algorithm we later apply.
* All the values containing **Nan** were replaced with 0.
* We have removed features containing dates like **issueDate**, **maturity**, **ratingAgency1EffectiveDate**.
* We have categorized features into two parts: ***Static Features*** and ***Time-Dependent Features***. Static Features contains features which are time invariant like **issuer**, **market**, **amtOutstanding**, **collateralType** etc. Time-Dependent features contains features that are vary on daily basis.
* For each bond, we have calculated the sum of buy volumes for a given day and considered it as a time-dependent feature for that bond. This way, we come up with 86 time-dependent features for each bond which contain sum of values of buy volumes for each day. Similarly, we can calculate 86 time-dependent features for sell volumes. We aim to evaluate the buy and sell volumes separately i.e., train a regressor separately for buy and sell volumes.
* We have assumed that sequence of buy/sell volumes for each day follows a time series i.e., the volumes are dependent on values of buy/sell volumes of last few days. We have taken 85 days into account. So, value for buy/sell volumes for 86th day will depend on values of last 85 days.
* After predicting the value of buy/sell volumes for 87th day using 2nd to 86th day, we multiplied it by factor of 3 assuming average will be same on 3 days. This assumption has been taken due to lack of accurate answers, i.e., 87th day has been predicted by our algorithm. So, we do not want to use it to predict for 88th day as it would lead to erroneous prediction, Rather, we considered it to be same and multiplied the result of 87th day by 3.

After preparing and preprocessing the data, we generated 6 Data Frames:





