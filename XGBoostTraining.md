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
More probable cause: https://stackoverflow.com/questions/60859322/typeerror-javapackage-object-is-not-callable-for-xgboost-in-pyspark

### Stack details: 
* EMR release: 6.3.0
* Java JDK: 1.8.0_282
* Emr installed scala version 2-12.10
* Spark version:  3.1.1-amzn-0
* Python version: 3.7
* livy version: 
### Install pyspark and pycharm
` pip install pyspark==3.1.1`
To see pip dependency tree do:
```
pip install pipdeptree
$ pipdeptree -p pyspark
Ignoring invalid distribution -ffi (c:\users\k.iyer\anaconda3\lib\site-packages)

Warning!!! Possibly conflicting dependencies found:
* apache-airflow==1.10.10
 - pandas [required: >=0.17.1,<1.0.0, installed: 1.0.5]
 - jinja2 [required: >=2.10.1,<2.11.0, installed: 2.11.2]
 - werkzeug [required: <1.0.0, installed: 1.0.1]
* astroid==2.3.3
 - typed-ast [required: >=1.4.0,<1.5, installed: ?]
* msal-extensions==0.2.2
 - portalocker [required: ~=1.6, installed: 1.0.0]
* pexpect==4.8.0
 - ptyprocess [required: >=0.5, installed: ?]
* pytest-astropy==0.8.0
 - pytest-filter-subpackage [required: >=0.1, installed: ?]
 - pytest-cov [required: >=2.0, installed: ?]
* QDarkStyle==2.8
 - helpdev [required: >=0.6.2, installed: ?]
* spyder==4.0.1
 - pyqt5 [required: <5.13, installed: ?]
 - pyqtwebengine [required: <5.13, installed: ?]
------------------------------------------------------------------------
pyspark==3.1.1
  - py4j [required: ==0.10.9, installed: 0.10.9]


```
To run pyspark in pycharm

In git bash run:
```
conda init bash
# List all environments
conda env list
# Create pyspark_env
conda create -n pyspark_env
#Start pyspark: 
pyspark.cmd

```
Workbook user: https://github.com/sllynn/spark-xgboost/blob/master/examples/spark-xgboost_adultdataset.ipynb

Dataset: https://archive.ics.uci.edu/ml/machine-learning-databases/adult/ 

Download xgboost jars: 
https://mvnrepository.com/artifact/ml.dmlc/xgboost4j_2.12/1.0.0
https://mvnrepository.com/artifact/ml.dmlc/xgboost4j-spark_2.12/1.0.0
and install in C:\Users\<User.home>\Miniconda3\Lib\site-packages\pyspark\jars
In pycharm >> Settings >> ... >> Python Console , set environment variable as `SPARK_HOME=C:\Users\<User.home>\Miniconda3\Lib\site-packages\pyspark`
References
Conda cheatsheet: https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf

Pyspark installation: https://spark.apache.org/docs/latest/api/python/getting_started/install.html

Pyspark in pycharm: https://gongster.medium.com/how-to-use-pyspark-in-pycharm-ide-2fd8997b1cdd


How to link PyCharm with PySpark? https://stackoverflow.com/a/34714207/2643556
spark-xgboost PySpark wrapper for XGBoost4J-Spark: https://github.com/sllynn/spark-xgboost/blob/master/examples/spark-xgboost_adultdataset.ipynb


#### References
* [XGBoost4J-Spark Tutorial](https://xgboost.readthedocs.io/en/latest/jvm/xgboost4j_spark_tutorial.html)
* [XGBoost and Pyspark](https://towardsdatascience.com/pyspark-and-xgboost-integration-tested-on-the-kaggle-titanic-dataset-4e75a568bdb)
* [Good explanation of XGBoost classification loss functions](https://towardsdatascience.com/xgboost-mathematics-explained-58262530904a)
* [How to train xgboost with spark databricks](https://databricks.com/blog/2020/11/16/how-to-train-xgboost-with-spark.html)
* [Pyspark cheatsheet](http://datacamp-community-prod.s3.amazonaws.com/acfa4325-1d43-4542-8ce4-bea2d287db10)
* [Install Pyspark](https://www.datacamp.com/community/tutorials/apache-spark-python)
* [How to package specific libraries for pyspark using virtualenv](https://spark.apache.org/docs/latest/api/python/user_guide/python_packaging.html#using-virtualenv)
