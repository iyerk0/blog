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

Updated, the reference for `JavaPredictionModel` in `xgboost.py` from `pyspark.ml.util` package to `pyspark.ml.wrapper` 

### Issue 2
```python
An error was encountered:
Cannot create a consistent method resolution
order (MRO) for bases object, JavaModel, JavaPredictionModel, JavaMLWritable, XGBoostReadable
Traceback (most recent call last):
  File "<frozen importlib._bootstrap>", line 983, in _find_and_load
  File "<frozen importlib._bootstrap>", line 967, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 668, in _load_unlocked
  File "<frozen importlib._bootstrap>", line 638, in _load_backward_compatible
  File "/mnt/tmp/spark-97e40e40-2f87-43f5-b48f-7c3128e60167/userFiles-18f60080-1b96-4832-b197-51a85580835e/sparkxgb-updated.zip/sparkxgb/__init__.py", line 20, in <module>
    from sparkxgb.xgboost import XGBoostEstimator, XGBoostClassificationModel, XGBoostRegressionModel
  File "<frozen importlib._bootstrap>", line 983, in _find_and_load
  File "<frozen importlib._bootstrap>", line 967, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 668, in _load_unlocked
  File "<frozen importlib._bootstrap>", line 638, in _load_backward_compatible
  File "/mnt/tmp/spark-97e40e40-2f87-43f5-b48f-7c3128e60167/userFiles-18f60080-1b96-4832-b197-51a85580835e/sparkxgb-updated.zip/sparkxgb/xgboost.py", line 184, in <module>
    class XGBoostClassificationModel(JavaParamsOverrides, JavaModel, JavaPredictionModel, JavaMLWritable, XGBoostReadable):
  File "/tmp/1633021828870-0/lib64/python3.7/abc.py", line 126, in __new__
    cls = super().__new__(mcls, name, bases, namespace, **kwargs)
TypeError: Cannot create a consistent method resolution
order (MRO) for bases object, JavaModel, JavaPredictionModel, JavaMLWritable, XGBoostReadable
```

POtential solution: https://stackoverflow.com/questions/29214888/typeerror-cannot-create-a-consistent-method-resolution-order-mro

### Issue 3
`NoClassDefFoundError: scala/Product$class`
Solution: https://stackoverflow.com/questions/44387404/noclassdeffounderror-scala-productclass move to scala 2.12 version of the xgboost4j libs

#### References
* [XGBoost4J-Spark Tutorial](https://xgboost.readthedocs.io/en/latest/jvm/xgboost4j_spark_tutorial.html)
* [XGBoost and Pyspark](https://towardsdatascience.com/pyspark-and-xgboost-integration-tested-on-the-kaggle-titanic-dataset-4e75a568bdb)
* [Good explanation of XGBoost classification loss functions](https://towardsdatascience.com/xgboost-mathematics-explained-58262530904a)
* [How to train xgboost with spark databricks](https://databricks.com/blog/2020/11/16/how-to-train-xgboost-with-spark.html)
