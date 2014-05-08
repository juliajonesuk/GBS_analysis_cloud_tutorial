Starting a spot instance
========================

GUI: Internet browser
---------------------

In this example we will request, through your internet browser, a spot instance with our public `AMI <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html>`_ already loaded with all the Linux update and GBS software necessary to start analyzing your GBS data!

**Here is a detailed video that shows you the steps outlined below:**

.. raw:: html

    <div style="margin-top:10px;">
      <iframe width="640" height="360" src="http://www.youtube.com/embed/pJH8nnKe51w?feature=player_detailpage" frameborder="0" allowfullscreen></iframe>
    </div>

.. Note::

 **Launch a Spot Instance in 10 Easy Steps!**

 1. In your browser, navigate to `Amazon console <https://console.aws.amazon.com/ec2/>`_ and select **Spot Requests** on the left panel.

 2. Use **Pricing History** to help you make a price decision: select instance 'r3.8xlarge' and you will see that over the last month, the price of the instance didn't reach 0,30 $! Not bad! We pay less than 10% of the actual price of the instance ($2.80/h)

 3. Request a spot instance

 4. Go to community AMIs and search for GBS (**ami-f422c59c**)

 5. Choose an Instance Type: `memory optimized r3.8xlarge <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/r3-instances.html>`_

 6. Configure the instance with a **maximum price of $0.30 ** or higher if you really want to make sure that your spot request is accepted)

 7. Add Storage: Size of the new image: 16 GB and add New Volume: select instance store 0 with device /dev/sdb (320 GB EBS drive 1). Add New Volume:select instance store 1 with device /dev/sdb (320 GB EBS drive 1). If you think you need more space than the instance provides, add it here.

 8. Configure `security group <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html>`_ : create a new security group named **GBS** with description **GBS analysis**. Type: SSH, Protocol: TCP, Port range 22, Source: My IP

 9. Review Spot Instance Request
 10. Will prompt you to create a new key pair (name: GBS_keypair)

.. Warning::

 If you plan on using this spot instance from your office and home or with your computer and other devices (iPad/iPhone), leave the IP address set to "Anywhere" in *Source* (step 8).

While waiting for the approval (~2-5 min) **go to your Terminal** and put restrictions on your keypair.

.. code-block:: bash

 mv /Users/thierry/Downloads/GBS_keypair.pem.txt /Users/thierry/Downloads/GBS_keypair.pem # you need to change the keypair name to finish with .pem
 sudo chmod 600 /Users/thierry/Downloads/GBS_keypair.pem
 
**You are ready to launch**, see :ref:`Launch command`.


 

CLI: Terminal
-------------

**Pricing History**

Missing the Terminal... Get the **Pricing History** of the instance over the last month to help you make a price decision. Copy and paste (in one block) these commands in your Terminal:

.. code-block:: bash

 s="-s 2014-03-26T12:00:00.000Z" # Start-time
 e="-e 2014-04-25T12:00:00.000Z" # End-time
 t="-t r3.8xlarge"               # Instance-type
 a="-a us-east-1b"               # Availability-zone
 d="-d Linux/UNIX"               # product-description

 ec2-describe-spot-price-history $s $e $t $a $d

**Request a Spot Instance**

Request a spot instance using our public `AMI <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html>`_ already loaded with the GBS analysis tools. Before requesting a spot instance, you need to create a `key pair <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html>`_, necessary to authenticate communications between Amazon and your computer. It's similar to pairing a Bluetooth speaker with your iPhone. For security reasons, you also need to create a `security group <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html>`_ and enable that group to control who's talking to your instance and computer through the port 22 reserved for Secure Shell (`SSH <http://en.wikipedia.org/wiki/Secure_Shell>`_). If you're only using one computer for launching commands, use your IP address with /32 prefix after the IP address. You need your external IP address:

1. `get your IP address here <http://whatismyipaddress.com>`_
2. Google search ``ip address`` (the first result displayed will be your external, or public, IP address).
3. use the Terminal with this command: ``curl -s checkip.dyndns.org|sed -e 's/.*Current IP Address: //' -e 's/<.*$//'``


.. code-block:: bash

 ec2-create-keypair GBS_keypair > ~/Downloads/GBS_keypair.pem   # create keypair GBS_keypair
 sudo chmod 600 /Users/thierry/Downloads/GBS_keypair.pem          # put restriction on the keypair
 ec2-create-group GBS -d "for GBS analysis test" # create a security group GBS
 ec2-authorize GBS -p 22 -s 65.92.226.105/32     # enable SSH from your IP address.

 ami_id="ami-f422c59c"               # ID of AMI with GBS tools
 t="r3.8xlarge"                      # instance type
 z="us-east-1a"                      # availability zone
 k="GBS_keypair"                     # name of YOUR keypair
 p="0.30"                            # maximum price 
 n="1"                               # number of spot instances
 r="one-time"                        # request type 'one-time|persistent'
 g="GBS"                             # ID of the security group
 b1="/dev/xvda=snap-2c1d8af2:16"     # block device mapping for AMI with 16GB
 b2="/dev/sdb=ephemeral0"            # block device mapping for EBS_1 (provided with instance)
 b3="/dev/sdc=ephemeral1"            # mapping of block device EBS_2 (provided with instance)
 #b4="-b /dev/sdd=:1000"              # optional EBS drive of 1TB (not provided with instance: $$, uncomment if you want)
 v="-v"                              # display verbose output
 

 ec2-request-spot-instances $ami_id -t $t -z $z -k $k -p $p -n $n -r $r -g $g -b $b1 -b $b2 -b $b3 $b4 -v $v

.. Warning::

 If you plan on leaving your office and having a look at your running instance from home or with an iPad/iPhone, don't use the field ``-s your-IP-address/32``.

To get the description of the Spot Instance Request use this command:

.. code-block:: bash

 ec2-describe-spot-instance-requests
 
If you need to cancel the request, use the request ID from the description into this command:

.. code-block:: bash

 request_id="your-spot-instance-request-id"
 ec2-cancel-spot-instance-requests $request_id

.. _Launch command:

Launch command
--------------

**Get a description of your instance**

After a few minutes, you can look in the Amazon console for approval of your spot instance request and get a description of your instance with this command:

.. code-block:: bash

 ec2-describe-instances

**Edit the next 2 commands to reflect your public DNS and Path to keypair**

.. code-block:: bash

 instance="ec2-50-16-153-255.compute-1.amazonaws.com"      # public DNS
 keypair_path="/Users/thierry/Downloads/GBS_keypair.pem"   # path to your key pair (might have to delete *.txt* at the end of the keypair file)


Start SSH connection with the instance
--------------------------------------

.. code-block:: bash

 ssh -i $keypair_path ec2-user@$instance


Mount your EBS volume
---------------------

The r3.8xlarge instance comes with 2x320 GB of SSD storage (EBS volume). Use the ``lsblk`` command to view your available disk devices and their mount points, to help you determine the correct device name to use. The output of lsblk removes the /dev/ prefix from full device paths.

.. code-block:: bash

 lsblk
 

To format the 2 drives use:

.. code-block:: bash

 sudo mkfs -t ext4 /dev/xvdb
 sudo mkfs -t ext4 /dev/xvdc

Use the following command to mount the EBS volume:

.. code-block:: bash

 sudo mount /dev/xvdb /media/ebs_1
 sudo mount /dev/xvdc /media/ebs_2

.. Note::

 Use the ``lsblk`` command to view your block device mapping -> their mount points, to help you determine the correct device name to use. The output of ``lsblk`` removes the /dev/ prefix from full device paths.


Sometimes 2 x 320 GB drive may not be enough space and if you need to create another `EBS drive <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-volume.html>`_ with more space, say 1 TB (`see pricing <http://aws.amazon.com/ebs/pricing/>`_) and you forgot when starting the instance, you can use these commands **from your computer**:

.. code-block:: bash

 SIZE=1000
 TYPE=standard
 ZONE=us-east-1a

 ec2-create-volume -s $SIZE -t $TYPE -z $ZONE

To describe volumes you have use this command

.. code-block:: bash

 ec2-describe-volumes

To attach the 1TB EBS drive to the instance with these commands:

.. code-block:: bash

 I="-i i-611a8642"                       # ID of the instance
 EBS="vol-e859faa4"                      # ID of Amazon EBS volume
 D="-d /dev/xvdf"                        # the device name

 ec2-attach-volume $I $EBS $D
 
To detach a volume from an instance use these commands:

.. code-block:: bash

 I="-i i-2a2bf74b"                       # ID of the instance
 EBS="vol-f46cc2ae"                      # ID of Amazon EBS volume
 D="-d /dev/sdf"                         # the device name
 ec2-detach-volume $I $EBS $D



After the 1 TB EBS volume is attached, you'll have to **format**, **make a mounting point** and **mount the EBS volume**.

.. code-block:: bash

 sudo mkfs -t ext4 /dev/xvdc          # format
 sudo mkdir /media/ebs_3              # create mounting point
 sudo mount /dev/xvdd /media/ebs_3    # mount


.. Note:: **The difference between S3 and EBS**

 1. EBS can only be used with EC2 instances while S3 can be used outside EC2
 2. EBS appears as a mountable volume while the S3 requires software to read and write data
 3. EBS can accommodate a smaller amount of data than S3
 4. EBS can only be used by one EC2 instance at a time while S3 can be used by multiple instances
 5. S3 typically experiences write delays while EBS does not

Mount your s3 bucket
--------------------

You need to **edit s3fs**, the software use to mount your s3 bucket to your Amazon instance.

.. code-block:: bash

 sudo nano /etc/passwd-s3fs                # edit the personal file used by s3fs
 s3-bucket-name:AccessKey:SecretKey        # add the information keeping " : "
 crtl-o                                    # to write the change to the file
 crtl-x                                    # to exit nano editor
 chmod 640 /etc/passwd-s3fs                # set permissions (might need sudo)
 chown ec2-user:ec2-user /etc/passwd-s3fs  # change ownership
 s3fs -o allow_other gbs_data /media/s3/   # mount s3 bucket 'gbs_data'
 
 # test your mounting installation:
 sudo touch /media/s3/testing # will save the file 'testing' to your bucket
 ls -l /media/s3 # will show the content of your bucket and your testing file

.. Note::

 See `s3fs documentation <https://github.com/s3fs-fuse/s3fs-fuse/wiki/Fuse-Over-Amazon>`_ for more information

Start Stacks
------------

Now, I guess you can't wait to start `Stacks <http://creskolab.uoregon.edu/stacks/>`_ version 1.19? To see if everything is working properly, just type ``populations``...

.. code-block:: bash

 populations # to test your installation!


**You are ready to start analyzing your GBS data!**

1. Output your analysis in one of the 2 EBS volumes or your extra 1 TB EBS volume
2. When your analyses are completed, transfer the compressed .tar.gz files to your s3 bucket to have access to your data when the instance shuts down.


**To terminate an instance, use this command from your computer:**

.. code-block:: bash

 instance_id="i-91f2d9c2"                 # modify to match your instance id
 ec2-terminate-instances $instance_id     # don't touch this one! 



.. Note::

 **Useful commands**

 - Name of your keypairs:             ``ec2-describe-keypairs``
 - Describe Secutity Group:           ``ec2-describe-group``
 - Delete a security Group:           ``ec2-delete-group name_of_security_group``
 - Add a `tag <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html>`_ for bookkeeping
 .. code-block:: bash

  resource="ami-55fa8e3c"               # the AWS-assigned ID to tag
  tag="--tag GBS=1"                     # key=value of the tag
  ec2-create-tags $resource $tag


 - Shows processing units available:  ``nproc``
 - For human readable CPU architecture information    ``lscpu``

 **Further reading:** `Introduction to spot instances <http://ec2-downloads.s3.amazonaws.com/Intro-to-Spot-Instances.pdf>`_

 
 
