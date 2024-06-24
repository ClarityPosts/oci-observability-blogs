---
title: "Setting up Identity & management Policies"
date: 2024-06-24 10:00:00 +1100
categories: EBS OCI O&M, EBS OCI IAM POLICIES
authors: 
  - name: Venu
    image: /images/default_profile.png
  - name: Tam
    image: /images/default_profile.png
tutorial_series: Monitoring E-Business Suite using OCI O&M
tutorial_number: 3
---
<br>
# Overview

In this tutorial we will go through the list of policies that would be needed by each OCI Observability & Management services.

Identiy Access management provides Security control access for cloud resources in your tenancy. Services, Groups and Users can be configured to access specific resources as per requirement. We need to ensure this is configured before performing activities to enabling the Observability & management (O&M) for EBS.

# Compartments

We either create a new Identity & Management (IAM) compartment or use existing compartment. Do note that the resource name naming conventions will be differ from one organization to another. Please substitute the values accordingly.

`OCI CONSOLE -> IDENTITY & SECURITY -> COMPARTMENTS` 

| IAM COMPARTMENT NAME | DESCRIPTION | 
|----------------------|-------------|
| non-prod-ebs | Non Production EBS Compartment  | 

> Note: non-prod-ebs is just used as an example.

# Identity & Management(IAM) User Groups

Create the below user group and add the users as shown.

| IAM GROUP NAME | IAM USER NAMES | DESCRIPTION | 
|----------------|-------------|-----------------|
| om-admins | om-user1@abc-corp.com, om-user2@abc-corp.com | OCI Administrators performing O&M Lifecycle Activities  | 

> Note: There can be more than one group, for simplicity we have used just one user group named om-admins

# Identity & Management(IAM) Dynamic Groups

- Create a Dynamic Group named 'om-dyn-group' and Substitute the values accordingly

| IAM DYNAMIC GROUP NAME | DESCRIPTION |
|-------------------|-------------|
| om-dyn-group | Non Production EBS Dynamic Group  |

> Note: om-dyn-group is used as an example and it can be different in your environment.

Dynamic group rules:

```
ANY {instance.compartment.id = '<ENTER_YOUR_TARGET_COMPARTMENT_OCID>'}
ALL {resource.type='managementagent', resource.compartment.id=<ENTER_YOUR_TARGET_COMPARTMENT_OCID>'}
ALL {resource.type='loganalyticsscheduledtask'}
ANY { ALL {resource.type = 'certificateauthority', resource.compartment.id = '<comaprtment_id>'} }
```

# Management Gateway Policies

> Note: This will only be applicable if you have installed a management gateway in your environment.

### With Identity Domain

Substitute the values <Identity_Domain_Name>, dynamic group name <om-dyn-group> and compartment name <non-prod-ebs>.

```
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to USE certificate-authority-delegates in compartment non-prod-ebs
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to USE vaults in compartment non-prod-ebs
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to USE keys in compartment non-prod-ebs
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to READ certificate-authority-bundle in compartment non-prod-ebs
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to READ leaf-certificate-bundle in compartment non-prod-ebs
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to manage certificate-authorities in compartment non-prod-ebs where any{request.permission='CERTIFICATE_AUTHORITY_CREATE', request.permission='CERTIFICATE_AUTHORITY_INSPECT', request.permission='CERTIFICATE_AUTHORITY_READ'}
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to manage leaf-certificates in compartment non-prod-ebs where  any{request.permission='CERTIFICATE_CREATE', request.permission='CERTIFICATE_INSPECT', request.permission ='CERTIFICATE_UPDATE', request.permission='CERTIFICATE_READ'}
ALLOW dynamic-group <Identity_Domain_Name>/'om-dyn-group' TO manage leaf-certificates in compartment non-prod-ebs where all{request.permission='CERTIFICATE_DELETE', target.leaf-certificate.name=request.principal.id}
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to manage vaults in compartment non-prod-ebs where any{request.permission='VAULT_CREATE', request.permission='VAULT_INSPECT', request.permission='VAULT_READ', request.permission='VAULT_CREATE_KEY', request.permission='VAULT_IMPORT_KEY', request.permission='VAULT_CREATE_SECRET'}
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to manage keys in compartment non-prod-ebs where any{request.permission='KEY_CREATE', request.permission='KEY_INSPECT', request.permission='KEY_READ'}
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to USE certificate-authority-delegates in compartment non-prod-ebs
Allow dynamic-group <Identity_Domain_Name>/'om-dyn-group' to USE key-delegate in compartment non-prod-ebs
```

### Without Identity Domain (Default Domain)

Substitute the values dynamic group name <om-dyn-group> and compartment name <non-prod-ebs>.

```
Allow dynamic-group om-dyn-group to USE certificate-authority-delegates in compartment non-prod-ebs
Allow dynamic-group om-dyn-group to USE vaults in compartment non-prod-ebs
Allow dynamic-group om-dyn-group to USE keys in compartment non-prod-ebs
Allow dynamic-group om-dyn-group to READ certificate-authority-bundle in compartment non-prod-ebs
Allow dynamic-group om-dyn-group to READ leaf-certificate-bundle in compartment non-prod-ebs
Allow dynamic-group om-dyn-group to manage certificate-authorities in compartment non-prod-ebs where any{request.permission='CERTIFICATE_AUTHORITY_CREATE', request.permission='CERTIFICATE_AUTHORITY_INSPECT', request.permission='CERTIFICATE_AUTHORITY_READ'}
Allow dynamic-group om-dyn-group to manage leaf-certificates in compartment non-prod-ebs where  any{request.permission='CERTIFICATE_CREATE', request.permission='CERTIFICATE_INSPECT', request.permission ='CERTIFICATE_UPDATE', request.permission='CERTIFICATE_READ'}
ALLOW dynamic-group om-dyn-group tO manage leaf-certificates in compartment non-prod-ebs where all{request.permission='CERTIFICATE_DELETE', target.leaf-certificate.name=request.principal.id}
Allow dynamic-group om-dyn-group to manage vaults in compartment non-prod-ebs where any{request.permission='VAULT_CREATE', request.permission='VAULT_INSPECT', request.permission='VAULT_READ', request.permission='VAULT_CREATE_KEY', request.permission='VAULT_IMPORT_KEY', request.permission='VAULT_CREATE_SECRET'}
Allow dynamic-group om-dyn-group to manage keys in compartment non-prod-ebs where any{request.permission='KEY_CREATE', request.permission='KEY_INSPECT', request.permission='KEY_READ'}
Allow dynamic-group om-dyn-group to USE certificate-authority-delegates in compartment non-prod-ebs
Allow dynamic-group om-dyn-group to USE key-delegate in compartment non-prod-ebs
```

# Management Agent Policies

### Without Identity Domain (Default Domain)

- Dynamic group Policies

Substitute the values dynamic group name <om-dyn-group> and compartment name <non-prod-ebs>.

```
Allow dynamic-group om-dyn-group to manage management-agents in compartment non-prod-ebs
Allow dynamic-group om-dyn-group to USE METRICS in compartment non-prod-ebs
Allow dynamic-group om-dyn-group to {LOG_ANALYTICS_LOG_GROUP_UPLOAD_LOGS} in compartment non-prod-ebs
Allow dynamic-group om-dyn-group to USE loganalytics-collection-warning in compartment non-prod-ebs
Allow dynamic-group om-dyn-group TO {STACK_MONITORING_DISCOVERY_JOB_RESULT_SUBMIT} IN TENANCY
```
- User Group Policies

Substitute the values user group name <om-admins> and compartment name <non-prod-ebs>.

```
ALLOW GROUP om-admins to manage management-agents in compartment non-prod-ebs
ALLOW GROUP om-admins to manage management-agent-install-keys in compartment non-prod-ebs
ALLOW GROUP om-admins to manage instance-agent-plugins in compartment non-prod-ebs
ALLOW GROUP om-admins to READ metrics in tenancy
ALLOW GROUP om-admins to manage alarms in compartment non-prod-ebs
ALLOW GROUP om-admins to manage ons-topics in compartment non-prod-ebs
```

### With Identity Domain

- Dynamic group Policies

Substitute the values <Identity_Domain_Name>, dynamic group name <om-dyn-group> and compartment name <non-prod-ebs>.
```
Allow dynamic-group '<Identity_Domain_Name>'/'om-dyn-group' to manage management-agents in compartment non-prod-ebs
Allow dynamic-group '<Identity_Domain_Name>'/'om-dyn-group' to USE METRICS in compartment non-prod-ebs
Allow dynamic-group '<Identity_Domain_Name>'/'om-dyn-group' to {LOG_ANALYTICS_LOG_GROUP_UPLOAD_LOGS} in compartment non-prod-ebs
Allow dynamic-group '<Identity_Domain_Name>'/'om-dyn-group' to USE loganalytics-collection-warning in compartment non-prod-ebs
Allow dynamic-group '<Identity_Domain_Name>'/'om-dyn-group' TO {STACK_MONITORING_DISCOVERY_JOB_RESULT_SUBMIT} IN TENANCY
Allow dynamic-group '<Identity_Domain_Name>'/'om-dyn-group' TO USE METRICS IN TENANCY
```
- User Group Policies

Substitute the values <Identity_Domain_Name>, dynamic group name <om-dyn-group> and compartment name <non-prod-ebs>.

```
ALLOW GROUP '<Identity_Domain_Name>'/'om-admins' to manage management-agents in compartment non-prod-ebs
ALLOW GROUP '<Identity_Domain_Name>'/'om-admins' to manage management-agent-install-keys in compartment non-prod-ebs
ALLOW GROUP '<Identity_Domain_Name>'/'om-admins' to manage instance-agent-plugins in compartment non-prod-ebs
ALLOW GROUP '<Identity_Domain_Name>'/'om-admins' to READ metrics in tenancy
ALLOW GROUP '<Identity_Domain_Name>'/'om-admins' to manage alarms in compartment non-prod-ebs
ALLOW GROUP '<Identity_Domain_Name>'/'om-admins' to manage ons-topics in compartment non-prod-ebs
```

> Note: If there is a multi-level compartment hierarcy then use <root_compartment>:<sub-compartment>:non-prod-ebs and so on.

# DB management Policies

- Only Default domain examples are provided for simplicity.

Substitute the values user group name <om-admins> and compartment name <non-prod-ebs>.

```
ALLOW GROUP om-admins TO manage dbmgmt-family IN compartment non-prod-ebs
ALLOW GROUP om-admins TO READ database-family IN compartment non-prod-ebs
ALLOW GROUP om-admins TO manage virtual-network-family IN compartment non-prod-ebs
ALLOW SERVICE dpd TO READ secret-family IN TENANCY
ALLOW SERVICE dpd TO manage objects IN TENANCY
ALLOW GROUP om-admins TO manage secret-family IN compartment non-prod-ebs
ALLOW GROUP om-admins TO manage buckets IN compartment non-prod-ebs
ALLOW GROUP om-admins TO manage autonomous-database-family IN compartment non-prod-ebs
ALLOW GROUP om-admins TO USE external-database-family IN compartment non-prod-ebs
```

# Stack Monitoring

- User group policies

Substitute the values user group name <om-admins> and compartment name <non-prod-ebs>.

```
ALLOW GROUP om-admins TO manage stack-monitoring-family IN compartment non-prod-ebs
ALLOW GROUP om-admins to USE ons-topics IN compartment non-prod-ebs
ALLOW GROUP om-admins to manage alarms IN compartment non-prod-ebs
ALLOW GROUP om-admins TO USE metrics IN compartment non-prod-ebs
ALLOW GROUP om-admins TO {MGMT_AGENT_DEPLOY_PLUGIN_CREATE, MGMT_AGENT_INSPECT, MGMT_AGENT_READ} IN compartment non-prod-ebs
ALLOW GROUP om-admins TO READ instances IN TENANCY
ALLOW GROUP om-admins TO manage external-database-family IN compartment non-prod-ebs
```
- Dynamic group policies

Substitute the values dynamic group name <om-dyn-group> and compartment name <non-prod-ebs>.

```
ALLOW dynamic-group om-dyn-group TO {STACK_MONITORING_DISCOVERY_JOB_RESULT_SUBMIT} IN compartment non-prod-ebs
ALLOW dynamic-group om-dyn-group TO USE METRICS IN TENANCY
```
# Logging Analytics (LA) Policies

O&M Logging Analytics policies for enabling service features and administration

- User group policies

Substitute the values user group name <om-admins> and compartment name <non-prod-ebs>.

```
ALLOW GROUP om-admins TO manage loganalytics-resources-family IN compartment non-prod-ebs
ALLOW GROUP om-admins TO READ compartments IN TENANCY
ALLOW GROUP om-admins TO READ metrics IN TENANCY
ALLOW GROUP om-admins TO manage management-dashboard-family IN TENANCY
ALLOW SERVICE loganalytics TO READ buckets IN TENANCY
ALLOW SERVICE loganalytics TO READ objects IN TENANCY
```
- Dynamic group policies

Substitute the values dynamic group name <om-dyn-group> and compartment name <non-prod-ebs>.
```
ALLOW dynamic-group om-dyn-group TO {LOG_ANALYTICS_LOG_GROUP_UPLOAD_LOGS} IN compartment non-prod-ebs
ALLOW dynamic-group om-dyn-group TO USE loganalytics-collection-warning IN compartment non-prod-ebs
```

# Application Performance Monitoring (APM) Policies

For Default Identity Domain :

Substitute the values user group name <om-admins> and compartment name <non-prod-ebs>.

```
ALLOW GROUP om-admins TO manage apm-domains IN compartment non-prod-ebs
ALLOW GROUP om-admins TO manage alarms IN compartment non-prod-ebs
ALLOW GROUP om-admins TO READ metrics IN compartment non-prod-ebs
ALLOW GROUP om-admins TO USE ons-topics IN compartment non-prod-ebs
```