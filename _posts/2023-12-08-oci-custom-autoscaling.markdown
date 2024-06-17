---
layout: post
title:  "Oracle Cloud Infrastructure (OCI) Custom Metric-based Autoscaling using Serverless Functions"
date:   2023-12-08 11:59:25 +1100
custom_css: style
categories: OCI Autoscaling
author: Venugopal Naik

---

# Overview
This document provides a detailed guide for implementing Custom Metric-based Autoscaling using Serverless Functions. 

While OCI Autoscaling inherently allows for automatic adjustments of compute instances in an instance pool based on default metrics like CPU and Memory utilization, there might be scenarios where customizations are necessary. In cases where scaling decisions are based on application-specific metrics (custom metrics), this documentation offers a specialized approach that deviates from the out-of-the-box features. 

This approach ensures adaptability to unique requirements, enabling efficient scaling decisions tailored to specific application needs, complementing the default autoscaling capabilities provided by OCI.

We will use Apache metrics in this demonstration.


![Architecture]({{ site.baseurl }}/images/arch.png)


1. As shown in the architecture, the instance pool is built using Oracle Linux 8 image. We will install Apache and all the required dependencies using a cloud-init script.
2. The cloud init script performs management agent installation, installation of required exporters (Apache and Node exporters), configures mgmt agent to scrape prometheus metrics exposed by exporters.
3. After the previous step, the custom metrics start showing up on OCI monitoring console with various metrics. One of the metric that we would use is apache_accesses_total. We will monitor the rate of change of this metric per minute.
4. We shall next create a Serverless Functions app using the OCI console and deploy 2 functions: scale-out and scale-in. Also, configure the required parameters.
5. Two notification topics are required: One for scale out and the other for scale in. scale-out Fn subscribes to scale-out topic and scale-in Fn subscribes to scale-in topic.
6. We will then define an Alarm rule to be triggered based on the custom metric threshold.

<b>High level Flow:</b> When the custom metric 'apache_accesses_total' rate per minute (in short, the requests per minute) crosses the threshold mark a trigger would be sent to notifications and the function is invoked. The function does required checks before scaling.

The following <b>steps</b> walks through the details of the implementation:

# PRE-REQUISITES

Before you start, Please clone [this](https://github.com/naikvenu/autoscaling/blob/main/apache_cloud_init.sh) repository.

# INSTANCE CONFIGURATION

Firstly, create an instance configuration using [these](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstanceconfig.htm) instructions.

> Note: Select Oracle Linux 8 as your image, add your ssh keys and click advanced options to add apache_cloud_init.sh script.

# PREPARE YOUR OBJECT STORAGE BUCKET

The apache cloud init script refers to an object storage bucket named ‘artifacts’ to download all the packages.
Create a new bucket named ‘artifacts’ and add the following packages:

1. Management Agent RPM (Package). It can be found on OCI Console -> Management Agent -> Download Agent RPM , Rename it to oracle.mgmt_agent.rpm
2. Create a new Management Agent Data Key and download key to file and name it as input.rsp and add it to artifacts bucket.
3. Download [Apache exporter](https://github.com/Lusitaniae/apache_exporter/releases/download/v1.0.3/apache_exporter-1.0.3.linux-386.tar.gz) , extract the executable and add it to the bucket.
4. Download [Node exporter](https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-386.tar.gz) , extract the executable and add it to the bucket.

The artifacts bucket after uploading all the required packages would look like this:

![ObjectStorage]({{ site.baseurl }}/images/obj.png)

# INSTANCE POOL

Create an instance pool using the instance configuration created in the previous step. Refer [this](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/creatinginstancepool.htm) document if required. 
Select a size of 2 as this is minimum required by the scaling function.

# ATTACH OCI LOAD BALANCER

Follow the steps provided [here](https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/updatinginstancepool_topic-To_attach_a_load_balancer_to_an_instance_pool.htm) to attach a load balancer to your instance pool.

# ACCESS THE APPLICATION

On your browser access the application using load balancer or instance public IP

http://Load-Balancer-IP

You should see an output like this:

>![Output]({{ site.baseurl }}/images/output.png)

# DEPLOY OCI SERVERLESS FUNCTION<

Create Serverless Functions Application:

In the `OCI Console -> open the navigation menu and click Developer Services -> Under Functions, click Applications and Create Application`

Follow the instructions on the screen. Use <b>Generic_X86</b> as the shape of the Fn.

Reference [doc](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscreatingapps.htm)

Click on getting started tab and follow `'Setup fn CLI on Cloud Shell'`. Open Cloud shell and perform the steps as per the instructions.

{% highlight linux %}
$ cd scale-out-Fn
$ fn list apps
$ fn -v deploy --app <app-name>    
$ cd scale-in-Fn
$ fn list apps
$ fn -v deploy --app <app-name>
{% endhighlight %}
    
Configure Functions Application:

{% highlight linux %}
INSTANCE_POOL_INCREMENT_SIZE 2
INSTANCE_POOL_MIN_SIZE	1	
INSTANCE_POOL_MAX_SIZE	10
INSTANCE_POOL_ID  <pool-id>
{% endhighlight %}

Configure Functions Logs:
    
`Under Functions -> Click on logs and enable logging` Select or create a new log group and log name as per instructions.


# CREATE NOTIFICATIONS TOPIC

<b>Scale Out Topic</b>
    
`OCI console -> OCI Notifications -> Create scale-out topic.`
Add Subscription as scale-out Function created from previous step.

<b>Scale In Topic</b>
    
`OCI console -> OCI Notifications -> Create scale-in topic.`
Add Subscription as scale-in Function created from previous step.

# ALARM DEFINITIONS 

At this point you shall see the metrics flowing through to OCI monitoring. Check the following:

`OCI console -> OCI monitoring -> Metrics Explorer -> apache_mod_stats (namespace) -> apache_accesses_total (metric)` and check if you see the data points.

If yes, then proceed to creating Alarm definitions.

<b>Scale-out Alarm</b>

    a. OCI console -> OCI monitoring -> Create scale-out Alarm. 
    b. Choose apache_mod_stats (namespace) -> apache_accesses_total (metric) -> Statistics as rate of change.
    c. Choose 5 minute intervals.
    c. Choose a threshold value of 500 for the testing.
    d. Select target as scale-out notification topic.

<b>Scale-in Alarm</b>

    a. OCI console -> OCI monitoring -> Create scale-in Alarm. 
    b. Choose apache_mod_stats (namespace) -> apache_accesses_total (metric) -> Statistics as rate of change.
    c. Choose 5 minute intervals.
    d. Choose a threshold value of 100 for the testing.
    e. Select target as scale-in notification topic.

# TEST SCALING

For load testing, create another instance and call it load generator

    $ sudo dnf install -y httpd

We can use apache benchmark tool to introduce some load:

    $ ab -n 5000 -c 50 http://Load-Balancer-IP:80/index.php

    -n indicates the number of requests
    -c is the concurrency

Observe the instance pool scaling operations. Optionally, we can also check Functions logs and verify the details of the scaling operation.

> Note: It is advised to please test this solution fully using a load testing   
        tool before using it in production.

---
<br>
** Disclaimer: I work for Oracle and the views expressed on this documentation are my own and do not necessarily reflect the views of Oracle. ** 








