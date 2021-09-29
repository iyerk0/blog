Add xgboost4j jars to the spark session via livy
```json
%%configure -f
{
	"conf": {
		"spark.pyspark.python": "python3",
		"spark.pyspark.virtualenv.enabled": "true",
		"spark.pyspark.virtualenv.type": "native",
		"spark.pyspark.virtualenv.bin.path": "/usr/bin/virtualenv",
		"spark.yarn.appMasterEnv.SPARK_HOME": "/usr/lib/spark/",
		"spark.yarn.appMasterEnv.PYSPARK_PYTHON": "/usr/bin/python3",
		"spark.pyspark.virtualenv.python_version": "3.7",
		"spark.jars.packages": "ml.dmlc:xgboost4j:0.72,ml.dmlc:xgboost4j-spark:0.72"
	}
}
```
Use session manager to log into EMR master EC2 instance. Then run
```sh
sudo su hadoop
cd ~
wget https://github.com/dmlc/xgboost/files/2161553/sparkxgb.zip
```
In sparkmagic notebook do
`


### Issue 1
```python
from sparkxgb import XGBoostEstimator

xgboost = XGBoostEstimator(
    featuresCol="features", 
    labelCol="Survival", 
    predictionCol="prediction"
)
```
gives error
```python
An error was encountered:
cannot import name 'JavaPredictionModel' from 'pyspark.ml.util' (/usr/lib/spark/python/lib/pyspark.zip/pyspark/ml/util.py)
Traceback (most recent call last):
  File "/mnt/tmp/spark-6d382661-d87e-4d35-affe-17a9b466ffb9/userFiles-6cb92273-1bd9-496f-a765-9d0aa880b71d/sparkxgb.zip/sparkxgb/__init__.py", line 20, in <module>
    from sparkxgb.xgboost import XGBoostEstimator, XGBoostClassificationModel, XGBoostRegressionModel
  File "/mnt/tmp/spark-6d382661-d87e-4d35-affe-17a9b466ffb9/userFiles-6cb92273-1bd9-496f-a765-9d0aa880b71d/sparkxgb.zip/sparkxgb/xgboost.py", line 21, in <module>
    from pyspark.ml.util import JavaMLWritable, JavaPredictionModel
ImportError: cannot import name 'JavaPredictionModel' from 'pyspark.ml.util' (/usr/lib/spark/python/lib/pyspark.zip/pyspark/ml/util.py)

```
Solution: https://stackoverflow.com/a/68243380/2643556
#### References
* [XGBoost4J-Spark Tutorial](https://xgboost.readthedocs.io/en/latest/jvm/xgboost4j_spark_tutorial.html)
* [XGBoost and Pyspark](https://towardsdatascience.com/pyspark-and-xgboost-integration-tested-on-the-kaggle-titanic-dataset-4e75a568bdb)
* [Good explanation of XGBoost classification loss functions](https://towardsdatascience.com/xgboost-mathematics-explained-58262530904a)
