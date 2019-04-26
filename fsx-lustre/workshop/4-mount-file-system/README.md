![](https://s3.amazonaws.com/aws-us-east-1/tutorial/AWS_logo_PMS_300x180.png)

![](https://s3.amazonaws.com/aws-us-east-1/tutorial/100x100_benefit_available.png)![](https://s3.amazonaws.com/aws-us-east-1/tutorial/100x100_benefit_ingergration.png)![](https://s3.amazonaws.com/aws-us-east-1/tutorial/100x100_benefit_ecryption-lock.png)![](https://s3.amazonaws.com/aws-us-east-1/tutorial/100x100_benefit_fully-managed.png)![](https://s3.amazonaws.com/aws-us-east-1/tutorial/100x100_benefit_lowcost-affordable.png)![](https://s3.amazonaws.com/aws-us-east-1/tutorial/100x100_benefit_performance.png)![](https://s3.amazonaws.com/aws-us-east-1/tutorial/100x100_benefit_scalable.png)![](https://s3.amazonaws.com/aws-us-east-1/tutorial/100x100_benefit_storage.png)

# **Amazon FSx for Lustre**

## Mount the file system

### Version 2018.11

fsx.l.wrkshp.2018.11

---

© 2018 Amazon Web Services, Inc. and its affiliates. All rights reserved. This work may not be  reproduced or redistributed, in whole or in part, without prior written permission from Amazon Web Services, Inc. Commercial copying, lending, or selling is prohibited.

Errors or corrections? Email us at [darrylo@amazon.com](mailto:darrylo@amazon.com).

---
### Prerequisites

* An AWS account with administrative level access
* An Amazon FSx for Lustre file system
* An Amazon EC2 instance
* An Amazon EC2 key pair

If a key pair has not been previously created in your account, please refer to [Creating a Key Pair Using Amazon EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair) from the AWS EC2 User's Guide.  

Verify that the key pair is created in the same AWS region you will use for the tutorial.

WARNING!! This workshop environment will exceed your free-usage tier. You will incur charges as a result of building this environment and executing the scripts included in this workshop. Delete all AWS resources created during this workshop so you don’t continue to incur additional compute and storage charges.

---

### Mount the file system

You must first complete [**Prerequisites**](../0-prerequisites) and the previous step [**Launch client**](../3-launch_client).

You must wait for the file system you created in section 1 **Create file system** to have a **Lifecycle** status of **AVAILABLE**. You can verify the file system's lifecycle status by viewing the status using FSx Console or the lifecycle using the CLI command below, substituting the appropriate region parameter based on your environment.

```sh
aws fsx describe-file-systems --output json --region ${region}
```

### Step 4.1: Log on to the Linux EC2 instance

- From the Amazon EC2 Console, select the **Lustre client - FSx Workshop** instance
- Click **Connect**
- Use the connection information to SSH to the instance using your laptop's terminal application

### Step 4.2: Install linux applications

> Complete the following steps in your SSH session connected to the **Lustre client - FSx Workshop** instance

- Run the following script

```sh
# update
sudo yum update -y

# install utilities
sudo yum install -y tree git wget
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm -P /tmp
sudo yum install -y /tmp/epel-release-latest-7.noarch.rpm
sudo yum install -y parallel nload ioping

# install smallfile
git clone https://github.com/bengland2/smallfile.git

# install fio
sudo yum groupinstall -y "Development Tools"
sudo yum install -y libaio-devel
git clone git://git.kernel.dk/fio.git
cd fio
./configure
make
sudo make install
cd

# install ior
sudo yum groupinstall -y "Development Tools"
sudo yum install -y openmpi openmpi-devel
git clone https://github.com/hpc/ior.git
export PATH=$PATH:/usr/lib64/openmpi/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib64/openmpi/lib/
cd ior
./bootstrap
./configure
make
sudo cp src/ior /usr/local/bin
cd

```

### Step 4.3: Verify applications were installed correctly

> Complete the following steps in your SSH session connected to the **Lustre client - FSx Workshop** instance

#### Verify parallel

- Run the following command

```sh
parallel --will-cite --version
```

- The output should be similar to this:

```sh
GNU parallel 20160222
Copyright (C) 2007,2008,2009,2010,2011,2012,2013,2014,2015,2016
Ole Tange and Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
GNU parallel comes with no warranty.

Web site: http://www.gnu.org/software/parallel

When using programs that use GNU Parallel to process data for publication
please cite as described in 'parallel --bibtex'.
```

#### Verify smallfile

- Run the following command

```sh
python ~/smallfile/smallfile_cli.py
```

- The output should be similar to this:


```sh
                                 version : 3.1
                           hosts in test : None
                   top test directory(s) : ['/var/tmp/smf']
                               operation : cleanup
                            files/thread : 200
                                 threads : 2
           record size (KB, 0 = maximum) : 0
                          file size (KB) : 64
                  file size distribution : fixed
                           files per dir : 100
                            dirs per dir : 10
              threads share directories? : N
                         filename prefix :
                         filename suffix :
             hash file number into dir.? : N
                     fsync after modify? : N
          pause between files (microsec) : 0
             minimum directories per sec : 50
                    finish all requests? : Y
                              stonewall? : Y
                 measure response times? : N
                            verify read? : Y
                                verbose? : False
                          log to stderr? : False
                           ext.attr.size : 0
                          ext.attr.count : 0
host = ip-172-31-21-162,thr = 00,elapsed = 0.604281,files = 200,records = 0,status = ok
host = ip-172-31-21-162,thr = 01,elapsed = 0.604252,files = 200,records = 0,status = ok
total threads = 2
total files = 400
total IOPS = 0
100.00% of requested files processed, minimum is  70.00
elapsed time =     0.604
files/sec = 661.943755
```

#### Verify fio

- Run the following command

```sh
fio --version
```
- The output should be similar to this:

```sh
fio-3.13-37-g0513
```


#### Verify ior

- Run the following command

```sh
ior
```

- The output should be similar to this:

```sh
IOR-3.2.0: MPI Coordinated Test of Parallel I/O
Began               : Mon Nov 19 02:02:51 2018
Command line        : ior
Machine             : Linux ip-172-31-21-162
TestID              : 0
StartTime           : Mon Nov 19 02:02:51 2018
Path                : /home/ec2-user
FS                  : 7.7 GiB   Used FS: 23.8%   Inodes: 0.5 Mi   Used Inodes: 13.6%

Options:
api                 : POSIX
apiVersion          :
test filename       : testFile
access              : single-shared-file
type                : independent
segments            : 1
ordering in a file  : sequential
ordering inter file : no tasks offsets
tasks               : 1
clients per node    : 1
repetitions         : 1
xfersize            : 262144 bytes
blocksize           : 1 MiB
aggregate filesize  : 1 MiB

Results:

access    bw(MiB/s)  block(KiB) xfer(KiB)  open(s)    wr/rd(s)   close(s)   total(s)   iter
------    ---------  ---------- ---------  --------   --------   --------   --------   ----
write     2004.51    1024.00    256.00     0.000014   0.000483   0.000002   0.000499   0
read      13588      1024.00    256.00     0.000002   0.000070   0.000001   0.000074   0
remove    -          -          -          -          -          -          0.000136   0
Max Write: 2004.51 MiB/sec (2101.88 MB/sec)
Max Read:  13587.69 MiB/sec (14247.73 MB/sec)

Summary of all tests:
Operation   Max(MiB)   Min(MiB)  Mean(MiB)     StdDev   Max(OPs)   Min(OPs)  Mean(OPs)     StdDev    Mean(s) Test# #Tasks tPN reps fPP reord reordoff reordrand seed segcnt   blksiz    xsize aggs(MiB)   API RefNum
write        2004.51    2004.51    2004.51       0.00    8018.02    8018.02    8018.02       0.00    0.00050     0      1   1    1   0     0        1         0    0      1  1048576   262144       1.0 POSIX      0
read        13587.69   13587.69   13587.69       0.00   54350.78   54350.78   54350.78       0.00    0.00007     0      1   1    1   0     0        1         0    0      1  1048576   262144       1.0 POSIX      0
Finished            : Mon Nov 19 02:02:51 2018
```



### Step 4.4: Install Lustre client

> Complete the following steps in your SSH session connected to the **Lustre client - FSx Workshop** instance

- Run the following commands to install Lustre client 2.10.6

```sh
sudo yum -y install https://downloads.whamcloud.com/public/lustre/lustre-2.10.6/el7/client/RPMS/x86_64/kmod-lustre-client-2.10.6-1.el7.x86_64.rpm
sudo yum -y install https://downloads.whamcloud.com/public/lustre/lustre-2.10.6/el7/client/RPMS/x86_64/lustre-client-2.10.6-1.el7.x86_64.rpm

```

- Reboot your instance


```sh
sudo reboot

```

- Log back into the **Lustre client - FSx Workshop** instance 

### Step 4.5: Mount the file system

- Find the DNS name of the file system. You can find the file system's DNS name by viewing it on the **Network & Security** tab of the FSx Console or running the CLI command below on your local laptop.

> Run this command on your local laptop using your terminal application

```sh
aws fsx describe-file-systems --output json --region ${region}

```

> Complete the following steps in your SSH session connected to the **Lustre client - FSx Workshop** instance

- Copy the script below into your favorite text editor

```sh
dnsname=<file-system-dns-name>

sudo mkdir -p /mnt/fsx
sudo chmod 777 /mnt/fsx
sudo mount -t lustre ${dnsname}@tcp:/fsx /mnt/fsx

```

- Replace <file-system-dns-name> with the **DNS Name** of your file system and run the script on the **Lustre client** command line.

- Run **df** to verify mount

```sh
lfs df

```

- The output should be similar to this:

```sh
UUID                   1K-blocks        Used   Available Use% Mounted on
fsx-MDT0000_UUID       107838464     4578048   103258368   4% /mnt/fsx[MDT:0]
fsx-OST0000_UUID      1182566272        4608  1182559616   0% /mnt/fsx[OST:0]
fsx-OST0001_UUID      1182566272        4608  1182559616   0% /mnt/fsx[OST:1]
fsx-OST0002_UUID      1182566272        4608  1182559616   0% /mnt/fsx[OST:2]

filesystem_summary:   3547698816       13824  3547678848   0% /mnt/fsx
```

---
## Next section
### Click on the link below to go to the next section

| [**Examine the file system**](../5-examine-file-system) |
| :---
---

For feedback, suggestions, or corrections, please email me at [darrylo@amazon.com](mailto:darrylo@amazon.com).

## License Summary

This sample code is made available under a modified MIT license. See the LICENSE file.

