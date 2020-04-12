# PVMAX REST API CLI

A bash script that provides CLI to interact with RESTAPI on VMAX All Flash arrays and POWERMAX arrays. 

With this you can perform provisioning of storage to standalone hosts, clusters and perform snapvx snapshot operations.

Total setup time probably takes 5 mins. 

You may leverage this script in infrastructure automation and complete storage provisioning in 4 simple steps.
# Supported Platforms
*  Dell EMC VMAX ALL FLASH & POWERMAX ARRAYS 
>  - Code level 5978.221.221 & above
>  - Unisphere 9.0 & above
*  Any Unix platform with bash shell 
*  Windows can be used with cygwin or similar emulators

# Prerequisite
*  Install cURL -- https://www.tecmint.com/install-curl-in-linux/

# List of available operations
```
pvmax key -v {4 digit symmid}
-- this is used to set authentication key for user specified in config file for a particualr array

pvmax igcreate -v {4 digit symmid} -n {name of initiator group} -w {comma seperated WWN without colons or child IG's to form parent IG}
-- this will create initiator group

pvmax pgcreate -v {4 digit symmid} -n {name of port group} -d {comma seperated dirport as 1d:4,2d:11}
-- this will create port group

pvmax sgcreate -v {4 digit symmid} -n {name of storage group} -p {SRP name} -l {SLO name} -s {comma seperated sizes of LUN in GB} -c {comma seperated count of LUNs}
-- this will create SG with specified SLO,SRP and creates requested TDEV's and add them to SG

pvmax mvcreate -v {4 digit symmid} -n {name of masking view} -i {init group} -p {port group} -s {storage group}
-- this will create masking view

pvmax snap_list -v {4 digit symmid} -s {SG name}
-- this will list the names of snapshots associated with specified SG

pvmax snap_get -v {4 digit symmid} -s {SG name} -n {snapshot name} -g {generation number}
-- this will show details of a specific snapshot

pvmax snap_establish -v {4 digit symmid} -s {source SG name} -n {snapshot name} -t {no of days to live}
-- this will create a new snapshot with specified name on given SG

pvmax snap_link -v {4 digit symmid} -s {source SG name} -n {snapshot name} -l {link SG name} -g {generation number}
-- this will link a specific snap of source SG to target SG

pvmax snap_unlink -v {4 digit symmid} -s {source SG name} -n {snapshot name} -l {unlink SG name} -g {generation number}
-- this will unlink a specific snap of source SG from target SG

pvmax snap_relink -v {4 digit symmid} -s {source SG name} -n {snapshot name} -l {link SG name} -g {generation number}
-- this will link a specific snap of source SG to target SG
```


  *SRDF operations, all delete operations will be added in next version*

# Configuration
*Note that the host where you install this doesn't need to have any gatekeepers. 
All communication happens over https calls directly to Unisphere server.*

1. On your management host create the following directory /usr/pvapi/
2. Download content of "PVAPI" directory from this repo and place them in /usr/pvapi/
3. Create a file named pvmax.config as shown below and save it under /usr/pvapi/

```
cat /usr/pvapi/pvmax.config

1234  https://10.190.10.1:8443/univmax/restapi/  000197801234   admin
1122  https://10.190.20.1:8443/univmax/restapi/  000197801122   admin 
```

4. Each line in config file contains 4 columns. 
>             - last 4 digits of symm id
>             - Unisphere IP:port ( you may have to adjust the port number if you are not using default 8443)
>             - Full symm ID
>             - Unishpere user that will be running this script

5. Add /usr/pvapi to $PATH
```
   export PATH=$PATH:/usr/pvapi
   You may have to save it permanently in /root/.bash_profile
```

6. Type pvmax and verify if you are able to see the list of allowed commands. That's it, you are ready to perform operations using REST API. 
   

# Usage & Examples
Most of the commands are self explainatory. The one time thing you need to do is generating auth token for each user/SID combination.

1. Generate authentication token, needed to be run only once. Remains active unless you change the password for the user.
```
   pvmax key -v 1234
   Token saved for user successfully
```

2. For cluster provisioning :  Create child IG's and then create a parent IG with child IG's. 
3. Creating SG with multiple sizes of TDEV's. Below command will create an SG and add 2x1000GB,1x500GB devices to the SG. SLO and SRP must be specified and compression will be automatically set to on.
```
   pvmax sgcreate -v 1234 -n dummy_sg -p SRP_1 -l diamond -s 1000,500 -c 2,1
```
4. Establishing new snapshot requires "time to live" to be specified. If no expiry needed, use -t 0.


   
