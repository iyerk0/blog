## Steps to debug s3 connectivity

Connecting to s3 and reading and writing data is some of most fundamental chores of any Cloud Engineer. 
As security has become of paramount importance, AWS has made available a plethora of tools to keep the data secure and out of prying eyes. That also means that when you add a new bucket 
or a new ec2 instance that needs access to an existing bucket, there is a high probability that things will not work in an existing network and security setup.
This can often cause frustration and swearing on the part of the budding Cloud Engineer. Why isn't the (*$#(*#&$(*!) _thing working? 
Having been through a few such episodes myself, I decided to add notes on how to debug this in a more methodical manner. Here goes:
1. Do you have a private S3 endpoint?
2. does the ec2 instance in your network have a route to the s3 endpoint?
3. Does the security group associated to the ec2 instance allows egress to the prefix list of the s3 endpoint?
4. Are there any NACLs blocking egress and ingress
5. Check the s3 endpoint policy for rules to access bucket
6. Check the role of the instance profile to see if it allows access to s3 bucket
7. Check the bucket policy of the s3 bucket if it allows access from the role to the bucket
8. Is the bucket encrypted with a KMS key? Check if the role has permission for KMS key decrypt
9. Finally check if you have Organization level service control policies which disallow certain s3 operations
