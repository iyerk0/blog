### How to view logs live in EMR cluster
1. Setup Session Manager for EMR. Add the `AmazonSSMManagedInstanceCore` policy to the EMR Instance Role.
2. Create EMR cluster using above role
3.Create a session manager session from your terminal to login to EMR master instance `winpty aws ssm start-session --target i-xxxxxxx`. Here, i-xxxxxxx is the EMR master node instance id. Use winpty if this is a Git BASH terminal
4.Once you are in the EMR master instance, logs can be found as listed [here](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-manage-view-web-log-files.html). 
5. Trigger a pyspark session from Sagemaker
6. Tail livy logs in the EMR master instance using the SSM session in your bash shell: tail -f /mnt/var/log/livy/livy-livy-server.out

### How to increase spark application timeout in livy
https://stackoverflow.com/questions/57666983/increasing-spark-application-timeout-in-jupyter-livy
* In Sagemaker notebook, open a terminal and do: 
  * `vim ~/.sparkmagic/config.json`
  * update the value for variable `"livy_session_startup_timeout_seconds": 900`
* In EMR master node: `vim /etc/livy/conf/livy.conf`
* Update/set values: 
```
# Enabled to check whether timeout Livy sessions should be stopped.
livy.server.session.timeout-check true

# Time in milliseconds on how long Livy will wait before timing out an idle session.
livy.server.session.timeout 1h

livy.rsc.server.connect.timeout  900s

# If Livy can't find the yarn app within this time, consider it lost.
livy.server.yarn.app-lookup-timeout 300s

```
* restart livy : 
```
sudo systemctl restart livy-server
sudo systemctl status livy-server
```

### How to view livy logs from sparkmagic kernel
curl  http://ip-xxx-xxx-xxx-xxx.us-east-1.compute-internal:8998/sessions/5/log?from=0 | jq
where ip-xxx-xxx-xxx-xxx.us-east-1.compute-internal is the internal DNS name for the EMR master node
 ### How to view all the logs in yarn
 Use session manager to login to master instance. Do: 
 
 ```
sudo su hadoop
yarn logs -applicationId application_xxxxx_yyyy	
 ```
### Logging via Sparkmagic kernel in Sagemaker
There are two variations to be able to log via spark session. Option 1 is to log via the Spark Context in the JVM. Option 2 is to be log in native python when using pyspark
#### Logging in pyspark via JVM
```python
log4jLogger = sc._jvm.org.apache.log4j
logger = log4jLogger.LogManager.getLogger("org.myorg.myapp")
logger.setLevel(log4jLogger.Level.DEBUG)
logger.debug("This is a debug level log")
```
You should now be able to see logs in: EMR master node: `tail -100f /var/log/livy/livy-livy-server.out`

#### Logging in pyspark via python
```python
import logging
logger = logging.getLogger("org.myorg.myapp")
logger.setLevel(logging.DEBUG)

# reset the handlers otherwise re-running the kernel will keep appending the handlers
logger.handlers=[]

# create formatter
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

file_handler = logging.FileHandler("/var/log/livy/myapp.log")
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(formatter)
logger.addHandler(file_handler)

#############################################
# There is an issue with adding streamhandler in pyspark via livy. See https://issues.apache.org/jira/browse/LIVY-774 . The following does not work
# add formatter to ch
ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)
ch.setFormatter(formatter)

# add ch to logger
logger.addHandler(ch)
#############################################

logger.debug("This is a debug level log")


```
### How to debug EMR logs not showing up in s3 logging bucket
* Ensure you have not capitalized the name of the bucket in EMR log URI field when creating a cluster via an [API](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-plan-debugging.html). Capitalization of s3 bucket name is invalid, but in certain cases EMR will accept a capitalized s3 name and then fail silently to post the logs to the s3 bucket
##### Troubleshooting missing logs files in S3
* Use Sessions manager to login into the EMR master instance
* Check status of logpusher services: `systemctl --type=service | grep -i log`
* View logpusher logs here: `vim /emr/logpusher/log/logpusher.log`
* Here you might see messages which show access denied to post to your S3 bucket. 
* Verify ec2 instance has appropriate access to write to the s3 logging bucket
* Restart logpusher: `sudo systemctl restart logpusher`

#### How to add jars to pyspark via a private repository
Add a ivysettings.xml to the master instance in EMR in `/home/hadoop/.ivy2/ivysettings.xml`
```xml
<ivysettings>
 <settings defaultResolver="artifactory" />
<resolvers>
    <ibiblio name="artifactory" m2compatible="true" root="https://artifactory.<your-org>.com/artifactory/libs-release"/>
</resolvers>
</ivysettings>
```
Then invoke the spark-shell as:
```
spark-shell --packages joda-time:joda-time:2.10.1 --conf spark.jars.ivySettings=/home/hadoop/.ivy2/ivysettings.xml
```
#### How to pre-install pandas 0.30.1 for all livy users in EMR
This solves the following [Stackoverflow question](https://stackoverflow.com/questions/68724073/install-pandas-on-emr-cluster) and [AWS Support Question](https://forums.aws.amazon.com/thread.jspa?messageID=989210&tstart=0)
* Use session manager to get a terminal into the master instance
* Run the following script file:
```sh
#!/bin/bash
sudo yum install python3-devel -y
sudo pip3 install boto3==1.18.46 -v
sudo pip3 install Cython==0.29.24 -v
sudo pip3 install numpy==1.20.3 -v
sudo pip3 install pandas==1.3.3  -v
#Pandas installation can take upto 10 minutes

#Install matplotlib
sudo pip3 install --upgrade --force-reinstall setuptools -v
udo pip3 install certifi -v
sudo pip3 install cppy -v
sudo yum install -y libjpeg-devel
sudo pip3 install matplotlib==3.4.3 -v
#Successfully installed matplotlib-3.4.3 pillow-8.3.2 pyparsing-2.4.7
sudo pip3 freeze
```
### References
* [Install python developer package](https://stackoverflow.com/a/21530768/2643556)
* [livy client template](https://github.com/cloudera/livy/blob/master/conf/livy-client.conf.template)
* [increase session timeout in livy](https://aws.amazon.com/premiumsupport/knowledge-center/emr-session-not-found-http-request-error/)
* [Spark context logging](https://stackoverflow.com/questions/25407550/how-do-i-log-from-my-python-spark-script)
* [Python logging](https://stackoverflow.com/a/13733863/2643556)
* Excellent general info on how pyspark interacts with the JVM based Spark context:
  * https://stackoverflow.com/a/13733863/2643556
  * https://dev.to/steadbytes/python-spark-and-the-jvm-an-overview-of-the-pyspark-runtime-architecture-21gg
