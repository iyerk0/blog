### How to view logs live in EMR cluster
1. Setup Session Manager for EMR
2. Use SSM session from your terminal to login to EMR master instance `winpty aws ssm start-session --target i-xxxxxxx` . i-xxxxxxx is the EMR master node instance id. Use winpty if this is a Git BASH terminal
3. EMR logs location are:
4. Trigger a pyspark session from Sagemaker
5. Tail livy logs in the EMR master instance: tail -f /mnt/var/log/livy/livy-livy-server.out

### How to increase spark application timeout in livy
https://stackoverflow.com/questions/57666983/increasing-spark-application-timeout-in-jupyter-livy

restart livy : 
sudo systemctl restart livy-server
sudo systemctl status livy-server

### How to view livy logs from sparkmagic kernel
curl  http://ip-xxx-xxx-xxx-xxx.us-east-1.compute-internal:8998/sessions/5/log?from=0 | jq
where ip-xxx-xxx-xxx-xxx.us-east-1.compute-internal is the internal DNS name for the EMR master node
