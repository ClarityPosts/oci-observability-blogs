---
layout: post
title:  "Seamless Kubernetes Logging with Fluentd: Leveraging OCI Logging as a Back-End"
date:   2024-01-18 11:59:25 +1100
categories: OCI Observability
custom_css: style
author: Venugopal Naik
author_image: /my_assets/images/profile.png
---

In this blog, we'll explore the seamless integration of Fluentd as a DaemonSet in a Kubernetes environment, utilizing OCI (Oracle Cloud Infrastructure) logging as a backend to analyze and centralize kubernetes pod logs. Integrating Fluentd with OCI logging not only ensures a cost-effective solution but also empowers you with efficient log analysis. Let's dive into the details of how this integration enhances your Kubernetes logging experience and unlocks the full potential of your log data.

Here is a little bit about the services:

<b>OCI Logging:</b> The Logging service provides a highly scalable and fully managed single interface for all the logs in your tenancy. Use Logging to access logs from all your resources, and enable, manage, and search them.

<b>Fluentd:</b> Fluentd is an open source data collector, which lets you unify the data collection and consumption for a better use and understanding of data.

<b>Why OCI logging ?</b>
It's low cost, provides saved searches and lets us create alerting on those saved queries. It also provides archiving capabilities to OCI object storage and it also provides a fluentd output plugin which can be leveraged to receive logs from any environment (on-prem or other cloud providers environment). In this example we are looking at a kubernetes environment in general and bringing those logs back into OCI Logging for analysis. Not just alerting once the metrics from saved search are available on OCI monitoring, we can use them to build metric dashboards as well.
If your use case looks similar to this then please read on.

![Architecture](../../../../../../images/fluentd.png)

1. Illustrated in the architecture, the logs from Application pods in a Kubernetes cluster are directed to the worker nodes, stored at the path /var/log/containers.
2. Within the Kubernetes cluster, Fluentd is implemented as a daemonset, employing the tail input plugin to retrieve logs and transfer them to the OCI Logging for analysis.
3. To transmit logs for analysis to OCI logging, we employ the [OCI fluentd logging plugin](https://github.com/oracle/fluent-plugin-oci-logging)

The following steps walks through the details of the implementation:

# Pre-requisites

Before you start, Please clone [this](https://github.com/naikvenu/oci-fluentd-logging) repository.

<b>Create a Generic IAM Observability User on OCI or use existing</b>

If you are new to this, please follow instructions provided [here](https://docs.oracle.com/en-us/iaas/Content/Identity/users/create-user-accounts.htm)

OCI Identity & Management (IAM) Policy required 

&nbsp; `Allow group <group-name> to manage log-content in tenancy`

<b>Prepare your OCI Container Registry (OCIR)</b>

Please follow instructions provided [here](https://docs.oracle.com/en-us/iaas/Content/Registry/Concepts/registryprerequisites.htm)

You will have to setup a Auth token and also provide appropriate permissions to control repository access.

OCI Identity & Management(IAM) Policy required 

&nbsp; `Allow group <group-name> to manage repos in tenancy`

# Build OCI Logging Fluentd image

You can either use OCI cloud shell or any workstation where OCI CLI is already installed.

<b>Get the namespace</b>

{% highlight linux %}
$ oci os ns get 
{% endhighlight %}

<b>Build</b>

{% highlight linux %}
$ cd docker-image
$ docker login <region>.ocir.io
$ docker build -f Dockerfile -t <region>.ocir.io/<namespace>/<repo>/custom-logging-fluentd:latest  .
$ docker push <region>.ocir.io/<namespace>/<repo>/custom-logging-fluentd:latest 
{% endhighlight %}

<b>Add your OCI config as secrets</b>

{% highlight linux %}
$ kubectl create secret generic oci-creds --from-file=<path>/.oci/config
$ kubectl create secret generic oci-keys --from-file=<path>/oci_api_key.pem
{% endhighlight %}

please refer OCI docs on creating the API keys if you are not familiar with this step.

<b>The below secret is required to pull image from OCI container Registry</b>

{% highlight linux %}
$ kubectl create secret docker-registry ocirsecret --docker-server=<region>.ocir.io --docker-username=<namespace>/oracleidentitycloudservice/<username> --docker-password=<auth-token>
{% endhighlight %}

<b>The secrets would look like this</b>

{% highlight linux %}
$ kubectl get secrets

NAME         TYPE                             DATA   AGE
oci-creds    Opaque                           1      3h31m
oci-keys     Opaque                           1      10s
ocirsecret   kubernetes.io/dockerconfigjson   1      42
{% endhighlight %}

<b>Create Config map</b>

Before applying this, you first need to create a OCI logging log group and OCI logging log

`OCI console -> Observability and Management -> OCI logging -> Log group and Log`

> Note: Edit the OCID of the OCI log into the config.yaml file before applying

{% highlight linux %}
$ kubectl apply -f config.yaml

$ kubectl get configmap
NAME                              DATA   AGE
fluentd-config                    1      3h33m
{% endhighlight %}

<b>Apply the security permissions on kubernetes cluster</b>

{% highlight linux %}
$ kubectl apply -f security.yaml
{% endhighlight %}

<b> Apply fluentd as daemonset</b>

> Note: Edit the OCIR image name before creating fluentd daemonset

{% highlight linux %}
$ kubectl apply -f fluentd-daemonset.yaml
{% endhighlight %}

# Log on to OCI console and verify the logs

`OCI console -> Observability and Management -> OCI logging -> Log group and Log`


# Conclusion

It's essential to highlight that while deploying Fluentd using this method provides a robust solution for log forwarding, users are responsible for managing the fluentd.conf configuration file and handling upgrades to both Fluentd and the OCI logging plugin autonomously.

This guide has covered the crucial steps to channel logs into OCI, laying the foundation for an efficient log management system. Moving forward, users can elevate their logging strategy by defining parsers, implementing saved searches, and configuring alerts tailored to their specific needs. This additional layer of customization ensures a more tailored and effective log analysis within the Oracle Cloud Infrastructure.

We trust that the insights shared in this document have proven valuable, offering a solid foundation for users to optimize their log management processes and make the most out of OCI logging capabilities.

---
<br>
** Disclaimer: I work for Oracle and the views expressed on this documentation are my own and do not necessarily reflect the views of Oracle. ** 







