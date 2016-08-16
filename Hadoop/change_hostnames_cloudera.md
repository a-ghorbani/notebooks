
# Change Hostnames 
If hostname of a machine runs cloudera manager server/agent was changed (new instance on AWS from an snapshot), 
follow the instruction below:

* Update the target hosts:  add ip and host name to `/etc/hosts`
* in `/etc/cloudera-scm-agent/config.ini` set `server_host=new.private.dns`
* Update /etc/cloudera-scm-server/db.properties
* Update /etc/cloudera-scm-server/db.mgmt.properties
* Update database hosts entry:
Note: For scm you can find the password in `/etc/cloudera-scm-server/db.properties`
```
$> psql -U cloudera-scm -p 7432 -h localhost -d scm

scm> update hosts set name = 'new.private.dns' where public_ip_address = 'old.public.ip';
scm> update hosts set public_name = 'new.public.dns' where public_ip_address = 'old.public.ip';
scm> update hosts set ip_address = 'new.private.ip' where public_ip_address = 'old.public.ip';
scm> update hosts set public_ip_address = 'new.punlic.ip' where public_ip_address = 'old.public.ip'; 
``` 
* Then restart Cloudera Manager Server and Agent:
```
$> sudo service cloudera-scm-server stop
$> sudo service cloudera-scm-agent stop
$> sudo service cloudera-scm-server start
$> sudo service cloudera-scm-agent start
```

resources: 

http://www.cloudera.com/documentation/enterprise/5-5-x/topics/cm_ag_change_hostnames.html

http://www.cloudera.com/documentation/enterprise/5-5-x/topics/cm_ig_embed_pstgrs.html
