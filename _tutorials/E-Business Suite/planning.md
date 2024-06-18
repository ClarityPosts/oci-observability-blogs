---
title: "Planning E-Business Suite Observability and Monitoring"
date: 2024-06-12 10:00:00 +1100
categories: EBS OCI O&M
author: Venugopal Naik
author_image: /images/profile.png
tutorial_name: EBS OCI Monitoring
tutorial_series: E-Business Suite
tutorial_number: 1
---
# Overview

This tutorial guides users through the planning and discovery phase for enabling observability and management (O&M) for Oracle E-Business Suite (EBS) on Oracle Cloud Infrastructure (OCI). Specifically, it focuses on EBS workloads that are currently on-premises. We'll cover how to perform an environment discovery and guide you in selecting the most suitable solution architecture from the available options. Additionally, we will explain the importance of each detail collected during this process to ensure a smooth transition and integration.


# General Environment Details

Understanding the EBS version being used and collecting the EBS URL is essential, as this information may be needed to enable synthetic monitoring later and also ensure that we are using
a supported version of EBS.
  	 	 
| EBS Version | EBS URL | 
|---------------|-------------|
| 12.2.11 | https://your-ebs-url.com  | 


# Database Details	 

Collecting EBS database details is essential to enable Observability & Management for EBS Databases. We would use these details with OCI O&M Stack monitoring service which provides simplified discovery for Oracle Databases. 

Service that need these details:
- Database Management Service
- Stack Monitoring Service

| DB System | DB Type | DB Name | DB User Name | CDB/PDB Details | Port | Service Name | DNS hostname or SCAN name | 
|------------|---------|---------|---------------|----------|------|---------------|------------|-----------|
| Exadata (on-prem) | 19C | EBSDEV | dbsnmp (user role: Normal or SYSDBA ?) | EBSCDB/EBSPDB | 1521 | ebs_dev |  |       

| DB Host names | System IP's | Platform |
|--------------|------------|-------------|
| exadata001.domain.com | 10.1.x.x | Linux |
| exadata002.domain.com | 10.2.x.x | Linux |
| exadata003.domain.com | 10.3.x.x | Linux |

# Application Server Details

Host details are needed to install OCI management agents. Set Permissions on certain log locations
to collect logs for OCI log analytics service.
We also provision APM agents to setup Application Performance monitoring distributed tracing
for applications.

Service that need these details:
- Application Performance Monitoring
- Stack Monitoring Service
- Management Agent Installation
- OCI logging analytics Service

| Application HostName |	IP Address	| OS Version |	Java Version |
|------------------------|---------------|------------|---------------|
 | Host001 |	10.1.x.x	| Oracle Solaris 11.4 SPARC |	1.8.0_282 |
 | Host002 |	10.2.x.x	| Oracle Solaris 11.4 SPARC	| 1.8.0_282 |
 | Host003 |	10.3.x.x	| Oracle Solaris 11.4 SPARC	| 1.8.0_282 |
 | Host004 |	10.4.x.x	| Oracle Solaris 11.4 SPARC	| 1.8.0_282 |
 | Host005 |	10.5.x.x	| Oracle Solaris 11.4 SPARC	| 1.8.0_282 |

# Weblogic Details

For APM we need to deploy APM jar file on the weblogic console and add arguments for
OAFM and OACORE managed servers.

Service that need these details:
- Application Performance Monitoring
- Stack Monitoring 

| Weblogic Host | Weblogic Port | Weblogic Protocol | Weblogic username |
|---------------|-------------|---------------|-------------|
| 12.2.11 | 7001               | t3/t3s           | weblogic |

# Network Details

When deploying the management gateway or management agent in an on-premises environment with a proxy or firewall, it is necessary to configure the proxy settings appropriately. This ensures that the components can communicate with Oracle Cloud Infrastructure (OCI) through the designated proxy or firewall. The proxy details need to be added to the input configuration files during the installation process. The required information includes the proxy host and port number. 


| Proxy Details | OCI Connection type |
|---------------|---------------------|
| Host: company.proxy.com port: 80 | Fastconnect/IPSec VPN/Direct Internet |

# OCI Naming conventions

If you already subscribed to OCI tenancy, then please gather the tenancy name.

| OCI Tenancy Name | OCI Resources Naming | Observability Admins | OCI Observability Compartment | OCI IAM Domain |
|---------------|---------------------|---------------|---------------------|---------------------|
| your-tenancy-name | EBS_DEV_(your_resource_name) ex: EBS_DEV_DB_LOG, EBS_DEV_APM_POLICIES| obsAdmin1@abc-corp.com | EBS-OBS-DEV | DEFAULT (Provide URL)

# Tutorial Summary

In this tutorial, we explored the process of discovering the on-premises Oracle E-Business Suite (EBS) environment. This step is crucial and should be undertaken in the early stages before enabling the management of the EBS environment from Oracle Cloud Infrastructure (OCI).

The discovery process provides valuable insights into the current state of the on-premises EBS environment, which can be beneficial for the current implementation as well as future reference. 

As you progress through the tutorial series, the next step will be to delve deeper into the enablement process. 

Thank you !
