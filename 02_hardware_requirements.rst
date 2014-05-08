Hardware requirements
=====================

Desktop computer
----------------

Used for fine tuning the workflow, running preliminary analysis, small data sets and making ready-to-publish figures. Here are some specs that will help make the GBS pipeline, discussed in TREE, run smoothly:

- Recommended OS architecture: `UNIX <http://en.wikipedia.org/wiki/UNIX>`_, (e.g. Apple Computers of PCs running Linux)
- `CPU <http://en.wikipedia.org/wiki/Central_processing_unit>`_: > 8 cores architecture 
- `Memory <http://en.wikipedia.org/wiki/Memory_(computers)>`_: 8 GB RAM (minimum)
- Disk: 500-750 GB `SSD drive <http://en.wikipedia.org/wiki/Solid-state_drive>`_
- External `HDD drive <http://en.wikipedia.org/wiki/Hard_disk_drive>`_ for storing analyses (e.g. `Drobo <http://www.drobo.com>`_) and backup.

.. Note::

 **Hardware example**: High-end retina MacBook Pro or if you can afford it, go for a Mac Pro with 12 cores/64 GB RAM. Having an iPad or an iPhone can be handy, although not essential, not only to read this tutorial, but also to monitor the progress of your analysis on Amazon. We have made the process so easy that you can even run the entire pipeline on an iPad! 
 

Cloud computer
--------------

Recommend for best price/performance ratio -> `Amazon EC2 instance r3.8xlarge <http://aws.amazon.com/ec2/instance-types/>`_

- CPU: 104 `ECU <http://aws.amazon.com/ec2/faqs/#What_is_an_EC2_Compute_Unit_and_why_did_you_introduce_it>`_ (32 vCPU) cores architecture
- Memory: 244 GB RAM
- Disk: 2x320 GB of SSD (I also recommend using an extra 600-1000GB `EBS volume <http://aws.amazon.com/ebs/pricing/>`_)
- Storage: `Amazon S3 <http://aws.amazon.com/s3/>`_ for short term backup and to transfer next-gen data to instance. `Amazon Glacier <https://aws.amazon.com/glacier/>`_ for off-site and long term storage.
- **On demand instance:** 2,80 $US per hour
- **Spot instance:** 0,25 $US per hour!


.. Note:: **Understanding Amazon Elastic Cloud Compute (EC2) Instances**

 Amazon EC2 instances comes in different `CPU <http://en.wikipedia.org/wiki/Central_processing_unit>`_/`memory <http://en.wikipedia.org/wiki/RAM_memory>`_ configurations (`instances types <http://aws.amazon.com/ec2/instance-types/>`_ and `prices <http://aws.amazon.com/ec2/pricing/>`_). For evolutionary biologists 3 purchasing options will be interesting: spot, on-demand and reserved instances.

 `Spot Instances <http://aws.amazon.com/ec2/purchasing-options/spot-instances/>`_

 Spot Instances allow you to name your own price for Amazon EC2 computing capacity. Yes, you read correctly, no need to see the web site or look in the documentation. You simply bid on unused Amazon EC2 instances and run your instances for as long as your bid remains higher than the current Spot Price. You specify the maximum hourly price that you are willing to pay to run a particular instance type. The Spot Price fluctuates based on supply and demand for instances, but customers will never pay more than the maximum price they have specified. If the Spot Price moves higher than a customer’s maximum price, the customer’s instance will be shut down by Amazon EC2 (`how to manage interruption <http://youtu.be/wcPNnUo60pc>`_). Other than those differences, **Spot Instances perform exactly the same as On-Demand or Reserved Instances.** This pricing model provides the most cost-effective option for obtaining compute capacity with interruption-tolerant tasks.


 `On-Demand Instances <https://aws.amazon.com/ec2/purchasing-options/>`_

 On-Demand Instances let you pay the specified hourly rate for the instances you use with no long-term commitments or upfront payments. This type of purchasing option is recommended for applications that cannot be interrupted.


 `Reserved Instances <http://aws.amazon.com/ec2/purchasing-options/reserved-instances/>`_

 Reserved Instances let you make a low, one-time, upfront payment for an instance, reserve it for a one or three year term, and pay a significantly lower hourly rate for that instance. For applications that have steady state needs, Reserved Instances can provide savings of up to 71% compared to using On-Demand Instances. This is probably the least interesting option for evolutionary biologists.

Do you think Google Cloud might be a better solution for you? `See their compute engine here. <https://cloud.google.com/products/compute-engine/>`_ 
