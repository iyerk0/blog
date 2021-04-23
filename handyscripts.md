Listing out handy scripts we could all use.

### Get a list of roles which are tagged with a certain tag value
#### Context:
Typically resources tagged a certain way can be searched through the AWS tag editor. But surprise , surprise this is not true for tags on IAM resources such as roles 👎
Even worse there is not CLI/API such as `aws iam list-roles` which filters by tag values, what??!!
So here is a handy python script which filters for roles by tags. Enjoi
```python
# Get all role names in the account
rs = iam.list_roles()
rns = [ r['RoleName'] for r in rs['Roles']

# map out to a more workable datastructure
rtags =[{'role': rn, 'tags': iam.list_role_tags(RoleName=rn)['Tags']} for rn in rns]

# filter out roles with no tags
list(lambda tags: len(tags)>0, filter[list(list(rtag.values())[0]) for rtag in rtags])

# filter for any role which has a tag key of 'costcenter' and tag value of 'hr'
list(filter(lambda rtag: any(rt['Key']=='costcenter' and rt['Value']=='hr' for rt in rtag['tags']) , rtags))

>> [{'role': 'my-iam-role', 'tags': [{'Key': 'costcenter', 'Value': 'hr'}]}]

```
