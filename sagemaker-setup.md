# How to setup sagemaker with minimal external connectivity
By default sagemaker will allow connecting to the internet to allow the data scientist to install python lib to jupyter at free will. 
However for enterprise level setup, uncontrolled access to the internet is frowned due to security concerns. Security wise the ideal setup is to have a no outbound internet access.
However installing the libs offline to the notebook is quite an involved task. As an interim waypoint to that final solution is to restrict access just to the pypi repository.
This would go a long way in reducing the blast radius of an errant program egressing data to wild. 
However even getting to this point is an involved setup. Below I describe the steps needed to make it happen


### Setup your VPC
1. In this example we set up the VPC cidr range to `10.0.0.0/16` and has the name `Acme VPC`
2. Create a public subnet associated with `Acme VPC` with a cidr which is a subrange of the VPC cidr range. Here I have chosen the cidr subrange to be `10.0.0.0/26`
3. Create an internet gateway and attach it to your VPC.
4. Update the route table for acme public subnet, add a route for `0.0.0.0/0` to the Internet Gateway
5. Create a Nat gateway which belongs to the acme public subnet
6. Create a private subnet as part of the acme VPC. In this case I used a distinct cidr range of `10.0.1.0/26`
7. By default this private subnet will have the same routing table as that of the public subnet, which gives internet access. We dont want that. So instead create a separate route table for the private subnet. Add the route `0.0.0.0/0` to the nat gateway
8. Change the association in the private subnet to newly created private subnet above.
9. For Acme VPC AWS will create a default security group. By default it will allow all inbound access. Remove this rule to disallow any inbound access
10. What needs to be done at this point is to create a sage maker notebook which is connected to this VPC and disallows internet access through the sagemaker VPC. Make sure to click on the radio button for "Disable Access the internet through a VPC". We will give the notebook controlled access through the Acme VPC.
11. After a few minutes sagemaker notebook will start up
12. Open jupyterlab. In the newly launched jupyterlab page, in the launcher start a new shell
13. Verify your settings for the VPC, subnet and Security group have taken by ensuring the internet is accesible. `ping google.com` should succeed
14. Unfortunately Security group rules don't allow whitelist only certain hostnames. Rather we have to explicitly specify the IP addresses that are allowed/disallowed
15. For pip to work we need access to the following hosts
    1. `pypi.org`
    2. `files.pythonhosted.org`
18. Note down the ip address by running the command `nslookup pypi.org`. Do the same for the second hostname. These are the ipaddress that we need to add to default security group to scope it down
19. The Security group changes take fairly immediately. Check that you are not longer able to access any random host such as google.com. However installing packages using pip should work without issue since our Security Group updates have whitelisted those IPs
20. Happy Sagemaker deployment!
