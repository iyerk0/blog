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
* Update/set value: `livy.rsc.server.connect.timeout = 900s`
* restart livy : 
```
sudo systemctl restart livy-server
sudo systemctl status livy-server
```

### How to view livy logs from sparkmagic kernel
curl  http://ip-xxx-xxx-xxx-xxx.us-east-1.compute-internal:8998/sessions/5/log?from=0 | jq
where ip-xxx-xxx-xxx-xxx.us-east-1.compute-internal is the internal DNS name for the EMR master node


### References
* [livy client template](https://github.com/cloudera/livy/blob/master/conf/livy-client.conf.template)
* [increase session timeout in livy](https://aws.amazon.com/premiumsupport/knowledge-center/emr-session-not-found-http-request-error/)
