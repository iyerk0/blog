## Steps to debug s3 connectivity

Connecting to s3 and reading and writing data is some of most fundamental chores of any Cloud Engineer. 
As security has become of paramount importance, AWS has made available a plethora of tools to keep the data secure and out of prying eyes. That also means that when you add a new bucket 
or a new ec2 instance that needs access to an existing bucket, there is a high probability that things will not work in an existing network and security setup.
This can often cause frustration and swearing on the part of the budding Cloud Engineer. Why isn't the (*$#(*#&$(*!) _thing working? 
Having been through a few such episodes myself, I decided to add notes on how to debug this in a more methodical manner. Here goes:
