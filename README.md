![Ripjar Logo](/imgs/ripjar-logo-horizontal-orange.svg)

# Ripjar Labyrinth Threat Investigations (LTI) - AWS Security Lake Integration Guide

## Table of Contents

- [Overview](#overview)
  - [What is Labyrinth Threat Investigations (LTI)](#what-is-labyrinth-threat-investigations-lti)
  - [What is Amazon Security Lake](#what-is-amazon-security-lake)
  - [Integration overview](#integration-overview)
  - [Prerequisites](#prerequisites)
- [Integration guide](#integration-guide)
  - [1. Setup Cross-Account Searching](#1-setup-cross-account-searching)
    - [1.1 Configure IAM Roles](#11-deploy-iam-roles)
    - [1.2 Example Policies](#12-example-policies)
  - [2. Setup Labyrinth Threat Investigations (LTI)](#1-setup-labyrinth-threat-investigations-lti)
    - [2.1 Search App Configuration](#21-search-app-configuration)
    - [2.2 Adding New Security Lake Sources](#22-adding-new-security-lake-sources)
- [Support / Troubleshooting](#support)

## Overview

### What is Labyrinth Threat Investigations (LTI)

Labyrinth for Threat Investigations (LTI) is a Ripjar managed SaaS solution that provides a comprehensive enterprise-wide approach to threat exploration at scale.  It is based on data fusion, with fine-grained security, extensible workflows and sophisticated reporting. Augment your analysts with LTI’s Security Lake integration including native Open Cybersecurity Schema Framework (OCSF) support. With LTI, analysts can assess data from various sources, investigate and manage risk across your environments, enriching your investigations with external data sources using Ripjar's Robotic Process Automation (RPA) workflows and AI-based analytics.

**Potential use cases:**

- Threat Investigations - Empower analysts to leverage automation to quickly expose the impact of malicious activity by building connections across local and remote data sources and derived knowledge stores.
- Threat Hunting - Proactively Explore and Discover early indications of threat actors preparing to compromise network or corporate security, sharing intelligence with colleagues and partners.
- Incident Response - Respond to cyber and physical threats in real-time utilising a single view of risk and intelligence to understand potential impact.
-	Knowledge Management - Annotate findings from investigations directly on top of fused data sources with full lineage back to underlying data. Analysts collate a functional knowledge layer to support smarter decisions and investigations.

**Key features of LTI:**
-	Investigations UI: An analytical canvas for curating, fusing and interrogating unstructured and structured data.  This enables entity-centric analysis through network, geospatial, timeline and other visualisations and filters.
-	RPA Workflows: Configurable and extensible RPA workflows where users can low-code a sequence of processing for a range of use cases including data integration, rendering custom interfaces and running APIs and natural language processing (NLP) tasks.
-	Glue Data On-Boarding Wizard: Built using an RPA workflow, the wizard enables rapid on-boarding of new data sources. Analysts input the Glue table target, edit schema transformations if required and the data-source is automatically added to the Search App.
-	Knowledge: A graph-type store of versioned entities and relationships that can be published to and pulled from. This enables analysts to enrich their investigations with data curated by others across the Enterprise.
-	Insights: Analysts can write versioned intelligence reports with live data and visualisations. Insights can be securely shared with document, section / tear lines, and data object level security to restrict content to appropriate readers.
- Explorer:  A UI which provides analysts with the ability to understand, filter and aggregate data-sets through several views.
-	Entity Importer: Entities can be extracted by analysts manually, or automatically through NLP, and imported into the Investigation as nodes. This allows for more rapid searching and navigating through specific data points relevant to the domain.


### What is Amazon Security Lake

Amazon Security Lake is a fully managed security data lake service. You can use Security Lake to automatically centralize security data from AWS and third-party sources into a data lake that's stored in your AWS account. Security Lake helps you analyze security data, so you can get a more complete understanding of your security posture across the entire organization. With Security Lake, you can also improve the protection of your workloads, applications, and data.

The data lake is backed by Amazon Simple Storage Service (Amazon S3) buckets, and you retain ownership over your data.

Security Lake automates the collection of security-related log and event data from integrated AWS services and third-party services. It also helps you manage the lifecycle of data with customizable retention and replication settings. Security Lake converts ingested data into Apache Parquet format and a standard open-source schema called the Open Cybersecurity Schema Framework (OCSF).

Ripjar's Labyrinth Threat Investigations can subscribe to query the data that's stored in Security Lake for incident response and security data analytics.

### Integration overview

This integration guide provides instructions for setting up LTI and Amazon Security Lake so that the Security Lake data can be queried within LTI SaaS. This integration will:

1. Configure cross-account polices and IAM role to share data between the host/customer AWS account, and Ripjar LTI SaaS account.
2. Add a new Security Lake source to be available for Athena and LTI.
3. Configure the LTI Search Application to query the new Security Lake Source.
4. Verify the new source is available to be searched in the LTI application.

### Prerequisites

- You must be a customer of Ripjar Labyrinth Threat Investigations (LTI) SaaS. This is run on Ripjar managed AWS.
- Your source AWS account with the required Amazon Security Lake sources deployed and connected. (e.g. CloudTrail, VPC Flow, Route53 etc.)
- Ensure AWS [Athena](https://docs.aws.amazon.com/athena/latest/ug/getting-started.html) has access to query Security Lake data. Please follow [this setup guide](https://docs.aws.amazon.com/security-lake/latest/userguide/getting-started.html#explore-data-lake) for further details if you are unfamiliar with Athena.
- Note the names of the tables you are querying.
- When you can successfully query Security Lake data from Athena in the originating account, note down the names of the tables/sources that you are interested in querying from Labyrinth Threat Investigations (see the Example config object in section 2.2 below for details). These will be needed when configuring the Labyrinth Threat Investigations Search App.
- The tables names will start with `amazon_security_lake_glue_table_` and will contain the region name and the log type, for example: `amazon_security_lake_table_us_east_1_sh_findings`.


## Integration guide
### 1. Set Up Cross Account Searching
#### 1.1 Configure IAM Roles

The two AWS accounts must have the appropriate roles and policies to enabled cross account searching of data in a secure and trusted manner.
Cross Account IAM role assumptions are used to enable the LTI SaaS application (Ripjar managed AWS) to search Security Lake data originating from the source (customer) AWS account.

[Create an IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_common-scenarios_third-party.html) that trusts the Labyrinth Threat investigations AWS account.
The role you create should have access to read your AWS glue tables, query from AWS Athena, and read data from S3. This should be done on a least privileged basis, giving access only to the buckets and tables you wish to be able to query from Labyrinth Threat Investigations. It is recommended to use an External ID for increased security.

Once this role has been created you will need to supply Ripjar with the ARN of the role and the External ID. Labyrinth Threat Investigations will then assume that role to query your Security Lake data. This can be found on the Role details page and will appear with this syntax:
`arn:aws:iam::<ACCOUNT_ID>:role/<ROLE_NAME>`

#### 1.2 Example Polices
There are two policies in this repository which describe the permissions the IAM role will assume when performing queries on the target Security Lake across accounts.
The trust policy describes who can assume the role, and the permission policy describes what that role can do.

1. Please use or copy the policy files in this repository, and update the parameters in the angled brackets, for example `<TABLE NAME>`.
   1. *the two JSON policy files that need to be updated are:*
      1. [cross_account_trust_policy.json](/cross_account_trust_policy.json)
      1. [cross_account_permissionPolicy.json](/cross_account_permissionPolicy.json)
  
1. In the AWS Console, go to 'IAM console', 'Roles', `<CROSS ACCOUNT ROLE NAME>`, 'Permissions policies' and upload/edit the `cross_account_permissionPolicy.json` with your updated version.
1. In the AWS Console, go to 'IAM console', 'Roles', `<CROSS ACCOUNT ROLE NAME>`, 'Trust relationships' and upload/edit the `cross_account_trust_policy.json` with your updated version.
  
### 2. Set Up Labyrinth Threat Investigations (LTI)

#### 2.1 Search App Configuration
The search app configuration is where you can configure which sources are available for searching and investigation within the platform. You do not have to make all sources available for search. Labyrinth Threat Investigations will come with a set of default source options which need some parameters to be updated in order to connect to your Security Lake data. 
  1. Assert that the tables described in the search app configuration match the tables in the Glue Catalog.
  ![Screenshot of search app config](/imgs/screenshot-search-app-config.png)
  1. Assert that the service account can access all necessary components for making an AWS Athena Query through a test query in the Athena Console. Ripjar will also test that the query can be performed from the Ripjar account, once you have provided the table name, role name and account ID.
  
#### 2.2 Adding New Security Lake Sources
The following steps need to be taken in order to onboard a new data source to Labyrinth Threat Investigations:
  1. Ensure that the new source is available as an AWS Glue Table and can be queried from the source account through Athena console with a test query.  An example screenshot is below.
  ![Screenshot of Athena Query](/imgs/screenshot-athena-query.png)
  1. Ensure that the new source can be queried from the service account (Ripjar LTI) by providing the account ID, role name, table name and an example query to Ripjar who will test this.
  1. Create a Labyrinth dataset to store the results from Athena Queries, using Labyrinth Threat Investigations OCSF dataset Schema. For further guidance on how to do this, please see the LTI user guide within the application, or access your Ripjar provided Confluence.
  1. Add an additional config object to the Labyrinth Threat Investigations Search App.  This is done by editing the respective Search RPA Workflow in the workflow tab in LTI.
  
An example config object is shown below. Please note you will need to populate the Athena Table Name `athenaTableName:` and the Labyrinth RPA Workflow ID `preAsyncHook` as noted by the angled brackets.
  
Example config object:
      ```{
        id: "athena__cloudtrail", // Id and datasetId must match
        label: "CloudTrail", // Friendly UI label
        jobProcessor: "athena_request_processor", // Job Processor used to manage the searching of Athena
        datasetId: "athena__cloudtrail", // ID and datasetId must match
        options: {
          checked: true, // Should this source be one of the defaults
          estimatedRunTime: 120,
          defaultJobInput: {
            athena: true,
            athenaTableName: "amazon_security_lake_glue_db_<SOURCE>",
            region: "eu-west-1",
            athenaDatabaseName: "security-lake",
          },
          preAsyncHook: <WORKFLOW ID>
        },
      }```
  
  This can be tested by running a known query or wildcard search in the LTI application's search.
  ![Screenshot LTI Search Example](/imgs/screenshot-lti-search.png)
  
## Troubleshooting
### Queries failing
  - If queries across all sources of AWS Security Lake are failing, ensure that the IAM role used to connect to Athena has the ability to access not only Athena and Glue but also the relevant S3 buckets.
### Querying multiple tables.
  - Labyrinth Threat Investigations allows users to query across multiple data sources at once.
  - If you are attempting to use a fielded query across multiple sources, you must ensure that the field is available in both sources, if a field does not exist in a source which is queried against then the query will fail.
  - The failing of a query against a single source will have no impact on the remaining sources.

**For further support, please contact your Ripjar account manager or follow your agreed support system.**


