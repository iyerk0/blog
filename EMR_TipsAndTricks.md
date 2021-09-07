### How to view logs live in EMR cluster
1. Setup Session Manager for EMR. Add the `AmazonSSMManagedInstanceCore` policy to the EMR Instance Role.
2. Create EMR cluster using above role
3. Use SSM session from your terminal to login to EMR master instance `winpty aws ssm start-session --target i-xxxxxxx`. Here, i-xxxxxxx is the EMR master node instance id. Use winpty if this is a Git BASH terminal
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

### References
* [livy client template](https://github.com/cloudera/livy/blob/master/conf/livy-client.conf.template)
* [increase session timeout in livy](https://aws.amazon.com/premiumsupport/knowledge-center/emr-session-not-found-http-request-error/)
* [Spark context logging](https://stackoverflow.com/questions/25407550/how-do-i-log-from-my-python-spark-script)
* [Python logging](https://stackoverflow.com/a/13733863/2643556)
