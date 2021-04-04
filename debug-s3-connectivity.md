## Steps to debug s3 connectivity

Connecting to s3 and reading and writing data is some of most fundamental chores of any Cloud Engineer. 
As security has become of paramount importance, AWS has made available a plethora of tools to keep the data secure and out of prying eyes. That also means that when you add a new bucket 
or a new ec2 instance that needs access to an existing bucket, there is a high probability that things will not work in an existing network and security setup.
This can often cause frustration and swearing on the part of the budding Cloud Engineer. Why isn't the damn thing working? 
Having been through a few such episodes myself, I decided to add notes on how to debug this in a more methodical manner. Here goes.

#### Ensure the Role access exists for ability to read/write data
One can specify the permissions to read/write data in either the IAM role policy and/or the S3 Bucket policy. Thus there are multiple ways to specify policy and often both bucket policy and IAM policy are specified. AWS uses a combination of both policies to determine if the role has ability to perform read/write on the bucket. Here a useful diagram: 

![IAM Auth flow](https://dmhnzl5mp9mj6.cloudfront.net/security_awsblog/images/AuthZDiagram.png)
1. Do you have a private S3 endpoint? If yes:
  2.  ensure the VPC endpoint policy allows access to tbe bucket
  3.  ensure that the VPC endpoint and target bucket are in the same region. VPC endpoints do not allow cross region access
3. does the ec2 instance in your network have a route to the s3 endpoint?
4. Does the security group associated to the ec2 instance allows egress to the prefix list of the s3 endpoint?
5. Are there any NACLs blocking egress and ingress
6. Check the s3 endpoint policy for rules to access bucket
7. Check the role of the instance profile to see if it allows access to s3 bucket
8. Check the bucket policy of the s3 bucket if it allows access from the role to the bucket
9. Is the bucket encrypted with a KMS key? Check if the role has permission for KMS key decrypt
10. Finally check if you have Organization level service control policies which disallow certain s3 operations

### References
* [Iam and bucket policy](https://aws.amazon.com/blogs/security/iam-policies-and-bucket-policies-and-acls-oh-my-controlling-access-to-s3-resources/)
