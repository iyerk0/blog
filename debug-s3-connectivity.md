## Steps to debug s3 connectivity

Connecting to s3 and reading and writing data is some of most fundamental chores of any Cloud Engineer. 
As security has become of paramount importance, AWS has made available a plethora of tools to keep the data secure and out of prying eyes. That also means that when you add a new bucket 
or a new ec2 instance that needs access to an existing bucket, there is a high probability that things will not work in an existing network and security setup.
This can often cause frustration and swearing on the part of the budding Cloud Engineer. Why isn't the damn thing working? 
Having been through a few such episodes myself, I decided to add notes on how to debug this in a more methodical manner. Here goes.

#### Ensure the Role access exists for ability to read/write data
One can specify the permissions to read/write data in either the IAM role policy and/or the S3 Bucket policy. Thus there are multiple ways to specify policy and often both bucket policy and IAM policy are specified. AWS uses a combination of both policies to determine if the role has ability to perform read/write on the bucket. Here a useful diagram: 

![IAM Auth flow](https://user-images.githubusercontent.com/5314200/113523658-f59aeb80-955d-11eb-9075-35116624412c.png)
The rules of thumb are:
1. If there is no allow rule specified for action on a resource, then it is deny by default
2. An explicit deny will always override an explicit allow

Where keeping this in context becomes essential is the default deny policies in the bucket can block access to the bucket even if an IAM policy allows it.
Here is an example of a deny all s3 bucket policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::MyExampleBucket",
        "arn:aws:s3:::MyExampleBucket/*"
      ],
      "Condition": {
        "StringNotLike": {
          "aws:userId": [
            "AROAEXAMPLEID:*",
            "AIDAEXAMPLEID",
            "111111111111"
          ]
        }
      }
    }
  ]
}
```
Now even if you had an IAM Role `ReaderRole` (whose unique ID is *not* in the above `aws:userId` list), which gave explicit read access to the bucket as shown below
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::111111111111:role/ROLENAME"
            },
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": "arn:aws:s3:::MyExampleBucket"
        }
    ]
 }
```
The user who assumes this role will *not* have read access and will get *AccessDenied* error.
In order to get this working, you have to add the IAM role's uniqueId to the exception list of the bucket policy in the `aws:userId` key value.
You can retrieve a role's unique id by the aws cli command `aws iam get-role --role-name <role-name>`
Typically you can assume the role and test for read access or create an ec2 instance with the given role as an instance profile for that instance.
Then use [SSM](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-sessions-start.html#start-ec2-console) to shell into the instance and try to do a get object from there: `aws s3 cp s3://example_bucket/example_object`

If you still are unable to access the object troubleshoot using the [IAM policy simulator](https://policysim.aws.amazon.com/) to test if the role has access to bucket.
In the policy simulator 
If Policy SIM shows that the role has access to the bucket, but you are unable to still access objects from the s3 bucket, continue reading below :-) 

Ensure that policy sim shows *allowed* by tweaking both the bucket and IAM role policy

Try downloading from the SSM shell again. 
You may still get the error:
`An error occurred (AccessDenied) when calling the GetObject operation: Access Denied`


If you have setup a user with access key and secret. Ensure the creds are provisioned in your local desktop. Then edit the trust relationship of the role to allow this user to assume the role. 
On your command line in your desktop, do: `aws sts assume-role --role-arn <role_arn> --role-session-name <some-role-session-name>`
This will output a new access key and secret. Note them and use is in the next command
Then do `AWS_ACCESS_KEY_ID=<access-key> AWS_SECRET_ACCESS_KEY=<secret-key> AWS_DEFAULT_REGION=<aws-region> AWS_ROLE_SESSION_NAME=<some-role-session-name> AWS_SESSION_TOKEN=<session-token> aws s3 cp s3://<bucket>/<object_path> .`
You may get the error: `fatal error: An error occurred (403) when calling the HeadObject operation: Forbidden`
Is the S3 bucket having encrypted with KMS key. If yes, then your role will need apporpriate decrypt privileges

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
* [S3 bucket deny all policy](https://aws.amazon.com/blogs/security/how-to-create-a-policy-that-whitelists-access-to-sensitive-amazon-s3-buckets/)
* [Role unique identifiers](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html#identifiers-unique-ids)
