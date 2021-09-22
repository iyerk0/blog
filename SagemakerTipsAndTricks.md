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
  
  #### References
  * [Guidance to install python libs via sparkmagic kernel](https://aws.amazon.com/blogs/big-data/install-python-libraries-on-a-running-cluster-with-emr-notebooks/)
