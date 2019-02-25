<p align="center">
<img width=30% src="https://dai.lids.mit.edu/wp-content/uploads/2018/08/orion.png" alt=“Orion” />
</p>

<p align="center">
<i>Orion is a machine learning library built for data generated by satellites.</I>
</p>

<p align="center">
<i>An open source project from Data to AI Lab at MIT.</I>
</p>

# Orion
Orion is a machine learning library built for sensor data collected from Satellites. The library makes use of a number of automated machine learning tools developed under ["The human data interaction project"](https://github.com/HDI-Project)  within the [Data to AI Lab at MIT](https://dai.lids.mit.edu/). The focus, with the ready availability of automated machine leanring is on:

* domain expert interaction with the machine learning system
* learning from minimal labels
* explainability of model outputs
* model audit
* scalability

## License
- MIT license


# Install

To install the project, after cloning the repository and creating a virtualenv, execute

```
make install-develop
```

# Command Line Interface usage example

In the following example we load the `S-1` signal from NASA data,
execute the `lstm_dynamic_threshold` pipeline on it and store the
found anomalies in the database.

Afterwards, we use the CLI to insert and see some comments.

1. First, we reset the database to start from scratch.
   This command will ask the user to input the name of the database
   as a safe mesure.

```
$ orion reset
WARNING: This will drop the database!
Please enter the database name to confirm: orion
Dropping database orion
```

2. Afterwards we have no datasets and no pipelines registered, so we proceed to register
   one of the NASA signals and a couple of pipelines

```
$ orion list datasets
No datasets found
$ orion add dataset S-1
$ orion list datasets
                 dataset_id             insert_time name signal  timestamp_column  value_column
0  5c7468e76c1cea309ee45327 2019-02-25 22:15:03.560  S-1    S-1                 0             1
$ orion list pipelines
No pipelines found
$ orion add pipeline lstm orion/pipelines/lstm_dynamic_threshold.json
$ orion list pipelines
                pipeline_id             insert_time   name
0  5c7468ff6c1cea3156e67614 2019-02-25 22:15:27.172   lstm
```

3. Once we have a dataset and a pipeline registered, we can run an analysis.

```
$ orion run lstm S-1 -v
2019-02-25 17:15:45,672 - 12724 - INFO - explorer - Loading dataset S-1
2019-02-25 17:15:45,681 - 12724 - INFO - explorer - Loading pipeline lstm
Using TensorFlow backend.
2019-02-25 17:15:46,565 - 12724 - INFO - analysis - Fitting the pipeline
Epoch 1/1
2019-02-25 17:15:50.108763: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 FMA
9899/9899 [==============================] - 57s 6ms/step - loss: 0.0562 - mean_squared_error: 0.0562
2019-02-25 17:16:58,660 - 12724 - INFO - analysis - Finding events
/home/xals/.virtualenvs/Orion/lib/python3.6/site-packages/scipy/optimize/optimize.py:570: RuntimeWarning: invalid value encountered in subtract
  numpy.max(numpy.abs(fsim[0] - fsim[1:])) <= fatol):
2019-02-25 17:17:12,796 - 12724 - INFO - analysis - 3 events found in 0:01:26.231063
2019-02-25 17:17:12,796 - 12724 - INFO - analysis - Storing results
Datarun id: 5c7469686c1cea31b469a933
```

4. Once the pipeline has finished, we can see a summary of our datarun.

```
$ orion list dataruns
                 datarun_id                   dataset                end_time  events             insert_time                  pipeline              start_time
0  5c7469686c1cea31b469a933  5c7468e76c1cea309ee45327 2019-02-25 22:17:12.796       3 2019-02-25 22:17:12.797  5c7468ff6c1cea3156e67614 2019-02-25 22:15:46.565
```

5. And we can see the events that our datarun detected.

```
$ orion list events -d 5c7469686c1cea31b469a933
                   event_id                   datarun             insert_time     score  start_time   stop_time  comments
0  5c7469686c1cea31b469a934  5c7469686c1cea31b469a933 2019-02-25 22:17:12.845  0.121541  1222840800  1222862400         0
1  5c7469686c1cea31b469a935  5c7469686c1cea31b469a933 2019-02-25 22:17:12.890  0.098881  1222884000  1222905600         0
2  5c7469686c1cea31b469a936  5c7469686c1cea31b469a933 2019-02-25 22:17:12.893  0.219194  1398016800  1399507200         0
```

6. Using the `event_ids`, we can add some comments to individual events.

```
$ orion add comment 5c7469686c1cea31b469a934 "This does not seem right"
$ orion add comment 5c7469686c1cea31b469a935 "This does not seem right"
$ orion add comment 5c7469686c1cea31b469a936 'This matches the paper results. Great!'
$ orion list events
                   event_id                   datarun             insert_time     score  start_time   stop_time  comments
0  5c7469686c1cea31b469a934  5c7469686c1cea31b469a933 2019-02-25 22:17:12.845  0.121541  1222840800  1222862400         1
1  5c7469686c1cea31b469a935  5c7469686c1cea31b469a933 2019-02-25 22:17:12.890  0.098881  1222884000  1222905600         1
2  5c7469686c1cea31b469a936  5c7469686c1cea31b469a933 2019-02-25 22:17:12.893  0.219194  1398016800  1399507200         1
```

7. We can even add more than one comment for each event

```
$ orion add comment 5c7469686c1cea31b469a934 "This is another comment, just to demonstrate how it works"
$ orion list comments
                   event_id                   datarun             insert_time     score  start_time   stop_time                                               text
0  5c7469686c1cea31b469a934  5c7469686c1cea31b469a933 2019-02-25 22:17:12.845  0.121541  1222840800  1222862400                           This does not seem right
1  5c7469686c1cea31b469a934  5c7469686c1cea31b469a933 2019-02-25 22:17:12.845  0.121541  1222840800  1222862400  This is another comment, just to demonstrate h...
2  5c7469686c1cea31b469a935  5c7469686c1cea31b469a933 2019-02-25 22:17:12.890  0.098881  1222884000  1222905600                           This does not seem right
3  5c7469686c1cea31b469a936  5c7469686c1cea31b469a933 2019-02-25 22:17:12.893  0.219194  1398016800  1399507200             This matches the paper results. Great!
```

8. Optionally, each of these outputs can be exported as a CSV file:

```
$ orion list events -d 5c7469686c1cea31b469a933 -o found_events.csv
Storing results in found_events.csv
$ cat found_events.csv
event_id,datarun,insert_time,score,start_time,stop_time,comments
5c7469686c1cea31b469a934,5c7469686c1cea31b469a933,2019-02-25 22:17:12.845,0.12154059531278764,1222840800,1222862400,2
5c7469686c1cea31b469a935,5c7469686c1cea31b469a933,2019-02-25 22:17:12.890,0.09888124355086333,1222884000,1222905600,1
5c7469686c1cea31b469a936,5c7469686c1cea31b469a933,2019-02-25 22:17:12.893,0.21919396915313805,1398016800,1399507200,1
```
