### How to add a python package to spark session (via SparkMagic kernel when connecting from Sagemaker to spark in EMR)

In one cell do:

```
%%configure -f
{ "conf":{
          "spark.pyspark.python": "python3",
          "spark.pyspark.virtualenv.enabled": "true",
          "spark.pyspark.virtualenv.type":"native",
          "spark.pyspark.virtualenv.bin.path":"/usr/bin/virtualenv","spark.yarn.appMasterEnv.SPARK_HOME":"/usr/lib/spark/",
          "spark.yarn.appMasterEnv.PYSPARK_PYTHON":"/usr/bin/python3","spark.pyspark.virtualenv.python_version":"3.7"
          }
  }
```
This enables the virtual env which is needed to add requisite python packages
If you have do not have access to the public pypi repo but are allowed only access to a private repo, do:
In another cell do:
```
sc.install_pypi_package("boto3","<private_pypi_url>")
```
 In the next cell verify python package has been installed.
 
 `sc.list_packages()`
 
 To add a jar file from an s3 location do the following:
  ```
  %%configure -f
    {
        "conf":{
              "spark.pyspark.python": "python3",
              "spark.pyspark.virtualenv.enabled": "true",
              "spark.pyspark.virtualenv.type":"native",
              "spark.pyspark.virtualenv.bin.path":"/usr/bin/virtualenv",
              "spark.yarn.appMasterEnv.SPARK_HOME":"/usr/lib/spark/",
              "spark.yarn.appMasterEnv.PYSPARK_PYTHON":"/usr/bin/python3",
              "spark.pyspark.virtualenv.python_version":"3.7"
    },
        "jars": ["s3://<your_bucket>/<your_jar>"]
}
  ```
To add a jar file as a maven dependency do
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
the maven coordinates are specified in `groupId:artifactId:version` and multiple dependencies are specified in a comma separated format
From: https://community.cloudera.com/t5/Support-Questions/How-to-import-External-Libraries-for-Livy-Interpreter-using/td-p/171812
  #### References
  * [Guidance to install python libs via sparkmagic kernel](https://aws.amazon.com/blogs/big-data/install-python-libraries-on-a-running-cluster-with-emr-notebooks/)
