Creating your own AMI
=====================

GUI internet browser
--------------------

This part of the tutorial covers the creation of your own AMI, from scratch! (~ 45 min)

**To help you through the GUI steps with your browser, watch this short video tutorial (3 min)**

**Here is a detailed video that show you the step outline below:**

.. raw:: html

    <div style="margin-top:10px;">
      <iframe width="640" height="360" src="http://www.youtube.com/embed/1BHKvJQAwDk?feature=player_detailpage" frameborder="0" allowfullscreen></iframe>
    </div>


1. Choose AMI: `Amazon Linux AMI (HVM) <https://aws.amazon.com/marketplace/pp/B00CIYTAUW?sr=0-28>`_
2. Choose an Instance Type: `memory optimized r3.8xlarge <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/r3-instances.html>`_
3. Configure Instance Details: 1 instance with Maximum price :0.30
4. Add Storage: Size of the new image: 16 GB
5. Add New Volume: select instance store 0 with device /dev/sdb (320 GB EBS drive 1)
6. Add New Volume:select instance store 1 with device /dev/sdb (320 GB EBS drive 1)
7. If you think you need more space than the instance provide, add it here.
8. Configure Security Group: create a new security group named **GBS** with description **GBS analysis**. Type: SSH, Protocol: TCP, Port range 22, Source: My IP
9. Review Spot Instance Request
10. Will prompt to create a new key pair (name: GBS_keypair)	

.. Warning::

 If you plan on using this spot instance from your office and home or with your computer and other devices (iPad/iPhone), leave the IP address set to "Anywhere" in *Source* (step 8).



While waiting for the approval (~2-5 min) **go to your Terminal** and put restriction on your keypair.

.. code-block:: bash

 mv /Users/thierry/Downloads/GBS_keypair.pem.txt /Users/thierry/Downloads/GBS_keypair.pem # you need to change the keypair name to finish with .pem
 sudo chmod 600 /Users/thierry/Downloads/GBS_keypair.pem
 
Start `SSH <http://en.wikipedia.org/wiki/Secure_Shell>`_ connection
-------------------------------------------------------------------

After a few minutes, you can look in Amazon console for approval of your spot instance request and **get a description of your instance** with this command:

.. code-block:: bash

 ec2-describe-instances

**Edit the next 2 lines with your public DNS and Path to keypair**

.. code-block:: bash

 instance="ec2-54-82-149-164.compute-1.amazonaws.com"      # public DNS
 keypair_path="/Users/thierry/Downloads/GBS_keypair.pem"   # path to your key pair (might have to delete *.txt* at the end of the keypair file)


**To start SSH connection:**

.. code-block:: bash

 ssh -i $keypair_path ec2-user@$instance

**Become root and keep your sudo privilege active throughout the session:**

.. code-block:: bash

 sudo -i


Updates..
---------

For librairies, software/dependencies and system updates, run the following commands and answer "y" or "yes" to all the questions:

.. code-block:: bash

 yum update  # will output-> Is this ok [y/d/N]: y


.. code-block:: bash

 yum install automake build-essential curl-devel expat-devel fuse fuse-devel fuse-utils gcc gcc-c++ gcc-gfortran gettext gettext-devel git java-1.7.0-openjdk-devel libcurl3 libcurl4-gnutls-dev libexpat1-dev libpng-devel libssl-dev libstdc++-devel libX11-devel libxml2-devel libXt-devel libz-dev mailcap mysql mysql-devel mysql-server openssh-* openssl-devel pkg-config python-dev python-magic readline-devel texinfo-tex trickle unixODBC-devel unzip zlib-devel

.. Note::

 **Fedora/Linux** distros uses ``yum`` while **Debian/Ubuntu** distros uses ``apt-get`` natively.


Install `pip <http://www.pip-installer.org/en/latest/index.html>`_
------------------------------------------------------------------

.. code-block:: bash

 cd /home/ec2-user
 wget https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py -O - | sudo python
 #retry the last command if you get an error
 sudo yum install python-pip # to install pip, Is this ok [y/d/N]: type "y"
 sudo rm -R /home/ec2-user/setuptools-3.4.4.zip

Install `s3cmd Tools <https://github.com/s3tools/s3cmd>`_
---------------------------------------------------------

This software will help get your data from s3 to your instance! You can install it on your computer too...

.. code-block:: bash

 sudo -i
 cd /etc/yum.repos.d
 wget http://s3tools.org/repo/RHEL_6/s3tools.repo
 yum install s3cmd # will output a question you say "y" twice!

You will be asked to accept a new GPG key – answer yes (perhaps twice).
That’s it. Next time you run ``yum upgrade`` you’ll automatically get the very latest s3cmd for your system. Now you need to configure s3cmd:

.. Note::

 **Configuring s3cmd** (output and **answers** you'll want to type)

 .. code-block:: bash

  s3cmd --configure
  
 - Enter new values or accept defaults in brackets with Enter.
 - Refer to user `manual <http://s3tools.org/kb/>`_ for detailed description of all options.
 - Access key and Secret key are your identifiers for Amazon S3
  - Access Key: **enter-your-Access-Key**
  - Secret Key: **enter-your-Secret-Key**
 - Encryption password is used to protect your files from reading by unauthorized persons while in transfer to S3
  - Encryption password: **your-password**
 - Path to GPG program [/usr/bin/gpg]: **hit-the-return-key**
 - When using secure HTTPS protocol all communication with Amazon S3 servers is protected from 3rd party eavesdropping. This method is slower than plain HTTP and can't be used if you're behind a proxy
  - Use HTTPS protocol [No]: **hit-the-return-key**
 - On some networks all internet access must go through a HTTP proxy. Try setting it here if you can't conect to S3 directly
  - HTTP Proxy server name: **hit-the-return-key**

 - New settings:
  - Access Key: your-Access-Key
  - Secret Key: your-Secret-Key
  - Encryption password: your-password
  - Path to GPG program: /usr/bin/gpg
  - Use HTTPS protocol: False
  - HTTP Proxy server name: 
  - HTTP Proxy server port: 0


 - Test access with supplied credentials? [Y/n] **y**
 - Please wait...
 - Success. Your access key and secret key worked fine :-)
 - Now verifying that encryption works...
 - Success. Encryption and decryption worked fine :-)
 - Save settings? [y/N] **y**
 - Configuration saved to '/root/.s3cfg'

**To modify s3cmd configurations (e.g. you want to remove your Amazon keys and password**

.. code-block:: bash

 cd /root
 ls -al  # to view all the files
 sudo nano .s3cfg 
 ctrl-o   # to write the change to the file
 crtl-x   # to exit nano editor

.. Note::

 **Use s3cmd to see your bucket content**

 .. code-block:: bash

  s3cmd ls s3://gbs_data/

 **Use s3cmd to GET content from your s3 into your instance:**

 .. code-block:: bash

  s3cmd get -r s3://gbs_data/ /home/ec2-user


 **Use s3cmd to fill with content your s3:**

 .. code-block:: bash

  s3cmd -P --recursive --acl-public put folder_to_transfer s3://gbs_data/


Install `s3fs tools <https://github.com/s3fs-fuse/s3fs-fuse>`_
--------------------------------------------------------------

.. code-block:: bash

 cd /home/ec2-user/
 git clone git://github.com/s3fs-fuse/s3fs-fuse.git
 cd s3fs-fuse
 ./autogen.sh
 ./configure
 make
 sudo make install
 cd ..
 sudo rm -R s3fs-fuse
 
**Modify your start up script**

With this Linux flavour it's *.bash_profile*

.. code-block:: bash

 cd /home/ec2-user/
 sudo nano .bash_profile # or use TextWrangler via Cyberduck or Transmit.
 
add ``:/usr/local/bin`` to the path so it becomes: ``PATH=$PATH:$HOME/bin:/usr/local/bin``. Save the file ``ctrl-o`` and ``crtl-x`` to exit nano editor.

**Don't forget to reload your start up script**

.. code-block:: bash

 source ~/.bash_profile

**Edit s3fs**

You need to add s3-bucket-name:AccessKey:SecretKey

.. code-block:: bash

 sudo nano /etc/passwd-s3fs
 s3-bucket-name:AccessKey:SecretKey
 crtl-o  # to write the change to the file
 crtl-x  # to exit nano editor
 chmod 640 /etc/passwd-s3fs
 chown ec2-user:ec2-user /etc/passwd-s3fs
 
**Edit fuse.conf**

.. code-block:: bash

 sudo sed -i'' 's/^# *user_allow_other/user_allow_other/' /etc/fuse.conf # uncomment 'user_allow_other'
 sudo chmod 640 /etc/fuse.conf
 chown ec2-user:ec2-user /etc/fuse.conf

Instance storage and s3 bucket
------------------------------

**First, create directories for your s3 bucket and your 2 x 320 GB EBS drives**

.. code-block:: bash

 sudo mkdir -p /media/s3
 sudo mkdir /media/ebs_1
 sudo mkdir /media/ebs_2

 sudo chown -R ec2-user:root /media/s3
 sudo chown -R ec2-user:root /media/ebs_1
 sudo chown -R ec2-user:root /media/ebs_2

**Format the 2 EBS drives:**

Use the ``lsblk`` command to view your available disk devices and their mount points, to help you determine the correct device name to use. The output of lsblk removes the /dev/ prefix from full device paths.

.. code-block:: bash

 sudo mkfs -t ext4 /dev/xvdb
 sudo mkfs -t ext4 /dev/xvdc

**Mount your s3 bucket and your 2 EBS drive**

.. code-block:: bash

 s3fs -o allow_other gbs_data /media/s3/
 sudo touch /media/s3/testing # will save the file testing to your bucket
 ls -l /media/s3 # will show the content of your bucket and your testing file
 sudo mount /dev/xvdb /media/ebs_1
 sudo mount /dev/xvdc /media/ebs_2

.. Note:: **Unmount drive with this command:**

 .. code-block:: bash
 
  sudo umount /media/s3/   # unmount s3 drive


**To see the if your 3 drives are mounted use this command:**

.. code-block:: bash

 df -h

**Congratulations! You now have a mounted s3 bucket and 2 EBS volumes mounted on your instance.**

Install GBS software
--------------------

Now, I guess you can wait to install `Stacks <http://creskolab.uoregon.edu/stacks/>`_ version 1.19 ?

.. code-block:: bash

 cd /home/ec2-user
 wget http://creskolab.uoregon.edu/stacks/source/stacks-1.19.tar.gz # you have to use version 1.14 instead? Just change 1.19 to 1.14.
 tar -xvf stacks-1.19.tar.gz
 cd stacks-1.19
 ./configure
 make
 sudo make install # this will install the binaries in /usr/local/bin
 cd ..
 sudo rm -R stacks-1.19 stacks-1.19.tar.gz # to remove the folder and gz file
 populations # to test your installation!

Also vcftools maybe?

.. code-block:: bash

 cd /home/ec2-user
 wget http://sourceforge.net/projects/vcftools/files/vcftools_0.1.12a.tar.gz
 tar -xvf vcftools_0.1.12a.tar.gz
 cd vcftools_0.1.12a
 make
 cd /home/ec2-user/vcftools_0.1.12a/bin
 sudo rm -R man1
 mv * /usr/local/bin/
 cd ..
 sudo rm -R /home/ec2-user/vcftools_0.1.12a vcftools_0.1.12a.tar.gz
 vcftools # to test your installation!

Now that you have customized your instance, you want to keep it for the next time you will use Amazon EC2...

`Save your AMI <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html>`_ 
-------------------------------------------------------------------------------

.. Warning::

 But first, **remove sensitive data on AMI that you share**. Delete the shell history before creating your AMI. The shell history may contain your secret access key or other private info that are not intended to be shared with users of your AMI. As an example, the following command can help locate the root user and other users' shell history files on disk and delete them, when run as root:

 .. code-block:: bash

  find /root/.*history /home/*/.*history -exec rm -f {} \;

 Remove your s3cfg configuration file and s3fs password information with this command:

 .. code-block:: bash

  sudo rm /root/.s3cfg /etc/passwd-s3fs


1. **From your instance:** unmount your EBS drives and s3 bucket

.. code-block:: bash

 sudo umount /media/s3/
 sudo umount /media/ebs_1/
 sudo umount /media/ebs_2/

2. `Create an Amazon EBS-backed AMI <http://docs.aws.amazon.com/AWSEC2/latest/CommandLineReference/ApiReference-cmd-CreateImage.html>`_ from your stopped instance will create a private AMI.  **From your computer:** 

.. code-block:: bash

 volume_id="i-e5b45bb6"                 # ID of the EBS root volume
 name="GBS_analysis"                    # name of your new AMI
 D="for GBS analysis image"             # description of your snapshot
 
 
 ec2-create-image $volume_id -n $name -d $D 
 
 
3. Go back in Amazon console, with your browser, and look for the progress of AMI creation, this may take up to 10 min. **To see the description of your image, from time to time, run this command:**

.. code-block:: bash

 ec2-describe-images 

**Congratulation you now have a private AMI associated to your Amazon AWS account!**
 
You want to share your AMI and make it public ? See this `Amazon tutorial <https://aws.amazon.com/articles/530>`_.


.. Note::

 **Handy commands (not available for spot instances)**

 Stop your instance:

 .. code-block:: bash

  instance_id="i-ddda368e"              # id of your instance
  ec2-stop-instances $instance_id

 Re-start your instance:
  
 .. code-block:: bash

  instance_id="i-ddda368e"              # id of your instance
  ec2-start-instances $instance_id

 Create a `snapshots <http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-creating-snapshot.html>`_ of your stopped instance's root volume

 .. code-block:: bash

  volume_id="i-ddda368e"                           # ID of the EBS root volume
  D="for GBS analysis image"             # description of your snapshot
  ec2-create-snapshot $volume_id -d $D

