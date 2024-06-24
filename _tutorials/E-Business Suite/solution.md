---
title: "Choosing the Right Solution Architecture"
date: 2024-06-06 10:00:00 +1100
categories: EBS O&M Solution Architectures, EBS OCI O&M
authors: 
  - name: Venugopal Naik
    image: /images/profile.png
  - name: Tam Nguyen
tutorial_series: Monitoring E-Business Suite using OCI O&M
tutorial_number: 2
---

# Overview

Before designing an OCI O&M Solution, consider whether your architecture requires a management gateway or if simply installing a management agent on the hosts will suffice. Each option is detailed in the sections below.

Typically, observability and management services are accessed over the internet. However, customers with on-premises workloads can securely connect to OCI using Secure IPSec VPN or FastConnect, leveraging end-to-end encryption. Note that it is necessary to whitelist *.oraclecloud.com or the IP address ranges for OCI O&M services on your corporate firewall.

Additionally, for cases where customers prefer to whitelist a single IP address, a gateway-to-gateway peering architecture is available.

# Management Gateway

The Management Gateway provides a single point of communication between the Management Agents and the Oracle Cloud Infrastructure.

Using the Management Gateway as the single point for traffic to and from the Oracle Cloud Infrastructure means that the enterprise firewall only needs to allow HTTPS communication from the host where the Management Gateway resides. This allows installing Management Agent on the remaining hosts which do not need to have direct access to the internet. 

![Gateway_Architecture]({{ site.baseurl }}/images/EBS-TUTORIAL/ebs_sol_arch.png)

In the above diagram, we see that management agents are deployed on the App servers and Database servers. The agents talk to Management gateway on port 4480. Management gateway inturn talks to OCI O&M services via corporate firewall on port 443 securely either via Site to Site VPN or fastConnect.

Oracle's Official documentation on Management gateway can be found [here](https://docs.oracle.com/en-us/iaas/management-agents/doc/management-gateway.html)

> Note: Oracle recommends to configure the Management Gateway first and then the Management Agent on the other hosts.

# Whitelisting

Do note that we would still need to whitelist *.oraclecloud.com or IP address ranges for management gateway to talk to OCI O&M services.

# Management Gateway Peering

Refer [this](https://docs.oracle.com/en/solutions/secure-data-upload/index.html#GUID-237C029D-F2A2-4128-85FF-01C6B47542EB) documentation which talks about different management gateway architectures and Peering.

# Management Gateway High availability

Refer [this](https://docs.oracle.com/en-us/iaas/management-agents/doc/install-management-gateway-ha-main-steps.html) documentation if the requirement is to deploy management gateways in a high availability fashion. 

# Management Agents

If the requirement is just to use management agents then refer [this](https://docs.oracle.com/en-us/iaas/management-agents/doc/you-begin.html#GUID-0FDD7E1D-E228-4D34-8EAD-21508405C67A) documentation and proceed to install agents on host and database servers. Firewall rules still be applicable in this case as well.

<br>
---
<br>
** Disclaimer: I work for Oracle and the views expressed on this documentation are my own and do not necessarily reflect the views of Oracle. ** 
