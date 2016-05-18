
# Download

`$ wget http://archive.cloudera.com/cm5/installer/latest/cloudera-manager-installer.bin`

`$ chmod u+x cloudera-manager-installer.bin`

# Install

`$ sudo ./cloudera-manager-installer.bin`

Add the following line to 

Follow the instructions.

Note: during installation for SSH login credential, provide pkey, the user (not root).

If you got error:  "Installation failed. Failed to receive heartbeat from agent"
follow the instruction [here](http://scotthuan.blogspot.de/2015/06/cloudera-installation-error.html) 
and also for the hostname use `echo $HOSTNAME`


# Notes

To run commands under another user, use for example for oozie as:

```bash
sudo -H -u oozie bash -c 'echo "I am $USER, with uid $UID"' 
```

