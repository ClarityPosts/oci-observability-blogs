---
title: "Selecting the Right Solution Architecture"
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

Before designing OCI O&M Solution consider whether in your architecture you would consider having a management gateway, or just a management agent installation on the hosts would suffice the requirement. We have explained under each of the below sections about these.

Observability and management services are usually exposed and accessed over the internet, but customers with the workloads running on-premises would connect to OCI either using Secure IPSec VPN or fastConnect securely and by leveraging end to end encryption.
Note, that you would require to whitelist *.oraclecloud.com or IP address ranges for OCI O&M services on your corporate firewall.

[Reference](https://docs.oracle.com/en-us/iaas/management-agents/doc/perform-prerequisites-deploying-management-agents.html#GUID-BC5862F0-3E68-4096-B18E-C4462BC76271__NETWORKPREREQUISITES-C3DFB461)

There is also a gateway to gateway peering architecture for use case when customers would just want to whitelist one IP.

# Management Gateway

> Note: Oracle recommends to configure the Management Gateway first and then the Management Agent on the other hosts.

The Management Gateway provides a single point of communication between the Management Agents and the Oracle Cloud Infrastructure.

Using the Management Gateway as the single point for traffic to and from the Oracle Cloud Infrastructure means that the enterprise firewall only needs to allow HTTPS communication from the host where the Management Gateway resides. This allows installing Management Agent on the remaining hosts which do not need to have direct access to the internet. 

![Gateway_Architecture]({{ site.baseurl }}/images/EBS-TUTORIAL/ebs_sol_arch.png)

In the above diagram, we see that management agents are deployed on the App servers and Database servers. The agents talk to Management gateway on port 4480. Management gateway inturn talks to OCI O&M services via corporate firewall on port 443 securely either via Site to Site VPN or fastConnect.

Oracle's Official documentation on Management gateway can be found [here](https://docs.oracle.com/en-us/iaas/management-agents/doc/management-gateway.html)

# Whitelisting

Do note that we would still need to whitelist *.oraclecloud.com or IP address ranges for management gateway to talk to OCI O&M services.

# Management Gateway Peering

Refer [this](https://docs.oracle.com/en/solutions/secure-data-upload/index.html#GUID-237C029D-F2A2-4128-85FF-01C6B47542EB) documentation which talks about different management gateway architectures and Peering.

# Management Gateway High availability

Refer [this](https://docs.oracle.com/en-us/iaas/management-agents/doc/install-management-gateway-ha-main-steps.html) documentation if the requirement is to deploy management gateways in a high availability fashion. 

# Management Agents

If the requirement is just to use management agents then refer [this](https://docs.oracle.com/en-us/iaas/management-agents/doc/you-begin.html#GUID-0FDD7E1D-E228-4D34-8EAD-21508405C67A) documentation and proceed to install agents on host and database servers. Firewall rules still be applicable in this case as well.

---
<br>
** Disclaimer: I work for Oracle and the views expressed on this documentation are my own and do not necessarily reflect the views of Oracle. ** 
