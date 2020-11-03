### High Level Objectives:


1.  Preparing Environment

2.  Creating Databricks Cluster

3.  Importing Notebooks

4.  Databricks Extensibility

5.  Uploading Manual Data

6.  Let's dig into the notebook.


## 1. Preparing Environment – Securing Sensitive Information


### Why Bother?

For a given workspace, when an Azure key-Vault Secret Scope is created, it links
Azure Key Vault, a secure place to store password sensitive information, and
then makes it available to a workspace. So, really, we need to have 1 key-vault
per workspace, put the secrets there, and then leverage them. But…

-   Suppose you have a notebook and would like to use a 3rd Party API Key –
    example FaceBook Analytics, then you would need a place to store sensitive
    information. During this hack – you have been granted access to go into the
    Azure Key Vault Resource, and See the Secrets. Perhaps your team will need
    to store secrets, that might be a good scenario for knowing how to Add a
    Secret Scope to a workspace.

-   Suppose that have very sensitive information. Currently the mounts in a
    workspace are created by a service principal. That means they are available
    for the entire workspace. You may find that based on your use cases,
    additional Azure Databricks workspaces are needed, in which case, this also
    makes sense to know how to do. Super easy, and secures your information.


### Create an Azure Key-Vault Secret Scope using the UI.


<https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes#create-an-azure-key-vault-backed-secret-scope-using-the-ui>

Once your Azure Databricks workspace is open in a separate window, append
\#secrets/createScope to the URL. The URL should have the following
format: **https://\<\\location\>.azuredatabricks.net/?o=\<\\orgID\>\#secrets/createScope**

Enter a scope name, and enter the Azure Key Vault DNS name and Resource ID you
saved earlier. Save the scope name in a text editor for use later in this
tutorial. Then, select Create.

Creating the Secret Scope 
As an FYI - we could do this using:

- Databricks ClI
- Databricks APIs
- Through the Portal

Often this will be part of a an IAC deployment, but consider the use case of 3rd party API keys.

Steps to Create a Secret Scope:

1.  Go to the url for your databaricks environment and add
    **\#secrets/createScope**

![](media/cf9e3f1f5ed0dfc337f4d6b53b77a0e8.png)

2.  Head over to the Azure portal to view our resources.

    Portal.azure.com - open the key vault and head to properties and grab the
    **Vault URI** and the **Resource ID** to place into our secret scope

![](media/12e34099ccef19d1031d9e9bb30b6907.png)

5.  Note for the name, it needs to be unique.  A recommendation would be to user your initials to make it unique to yourself.


4.  Here we can provide into the databricks secret scope the Vault URI into the
    DNS name, as well as the Resource ID into the ResourceID. For Manage
    Principal -

![](media/4dd2abcbb68c5124894fd6b5be64c156.png)

**Keep the name of your secret scope handy**

## 2. Create a Databricks Cluster

Let's create a databricks cluster by clicking on the cluster tab.

Let's leverage the 7.3 Runtime

<https://docs.databricks.com/release-notes/runtime/7.3ml.html>

![](media/11_CreateCluster.PNG)

Then click on the `Create Cluster` button

![](media/12_CreateCluster.PNG)



**Provide a cluster name** – each cluster must have a unique name.

Typically if I am working in a shared envirment, I will use my initials if that works for the team.

![](media/13_CreateCluster.PNG)


![](media/1316441e18b24bba72d098e3b977b797.png)

Select a Cluster Mode of **Standard**

![](media/e1ab58bc98a50dd701efe6de8b145e24.png)



| Cluster Mode| Cluster Mode Description|
| :---------| :----------------------------------------------|
| Standard | A standard cluster is meant to be used by 1 user|
|High Concurrency| Shared for multiple users |
| Single Node| A Single Node cluster is a cluster consisting of a Spark driver and no Spark workers (in public preview) |

![](media/cb254ed5e6b9bb725c18e51b600e6888.png)

 

### Whats a pool ? Pools
Speeds up cluster creation using existing available Azure vms


<https://docs.microsoft.com/en-us/azure/databricks/clusters/instance-pools/>

![](media/8ab523d1124aa55fd3c00db7a24d8366.png)

| Options|Description|
| :---------| :---------------------------------------------------------------|
| Enable Auto-scalling | Good if uncertain how many clusters you wil need, or great varability in your job|
| Auto shutdown | Makes sense to use|
| Worker Type | Select based on the work load. |
|Drive Type| Select based on the work load, if bringing a lot of data back to the head node, then give the driver more resources|

### FYI - Advanced Options

<https://docs.microsoft.com/en-us/azure/databricks/clusters/configure#spark-config>

Setting Environment Variables

![](media/a220099bbc374b7c11e0016fbbc4b3ec.png)

### Install Libraries:

![](media/099fee9d5b21f6d6b977214f2e163063.png)

Install ML Flow

![](media/ee738473652cbe28b140acdafb77dfb4.png)



## 3. Importing Notebooks 


When you export a notebook as HTML, IPython notebook, or archive (DBC), and you have not cleared the results, the results of running the notebook are included.

These notebooks are located in a git repo.

Leverage git to clone down this repo:

<https://github.com/memasanz/AMLHackADBMLFlow.git>

Or Download the Zip File.

![](media/ed4a55ad263c9e40f7af00fa52b49cff.png)


We will have **1** person import the HelperFuncions notebook.  This notebook is intented to how leveraging one notebook inside another.  We will spend a little time customizing this notebook to demonstrate leveraging a python class inside the databricks notebook expereince.

Everyone will import the FullExpermient, BaseExperiment and BaseExperimentWithAzureMLIntegration notebooks under this user.

![](media/06_ImportNotebooks.PNG)

![](media/07_Import1.PNG)

![](media/08_Import2.PNG)

![](media/09_ImportForUser.PNG)

![](media/10_ImportForUser.PNG)

Note that we will have to manually load notebooks 1 at a time if they are `*.pynb` extension

Import the following:

- FullExperiment.ipynb
- BaseExperimentWithAzureMLIntegration.ipynb
- BaseExperiment.ipynb

The 2 files: `BaseExperiment.ipynb` `BaseExperimentWithAzureMLIntegration.ipynb` are really just for reference, so you can move them into a folder to keep your workspace clean if you would like.

Go ahead and create a folder - samples and place them in there.



## 4. Databricks Extensibility (Optional Content):

Databricks Rest APIS <https://docs.databricks.com/dev-tools/api/index.html>

In order to make an API Call, we need to generate a token.  You could supply user/password informaiton, but it is safer and cleaner to provide token.  To generate a token head over to the left

![](media/03_GETToken.PNG)


![](media/04_GetToken.PNG)

Click on the **generate new token** button

![](media/05_RESTAPI.PNG)

### Sample Curl to get list of clusters  (Optional Content):
```
curl -X GET -H 'Authorization: Bearer dapi09a1af0770a1c777e4b962e8a54607e1' https://adb-1797455930767468.0.databricks.azure.us//api/2.0/clusters/list
```

We can also examine the list of secrets in a secret scope.

![](media/01_RESTAPI.PNG)

Just don't forget to provide the token

![](media/02_RESTAPI.PNG)




## 5. Uploading Data Manually


Uploading Datasets:

![](media/a8fc39b1bf631d7b639ee11c538f79d8.png)



## 6.  Let's dig into the notebook

In the notebook we will cover:

-	Protecting Sensitive Data: setting up an Azure KeyVault Back Secret Scope
-	Reviewing the existing Databricks environment (cluster creation, library installs etc)
-	Databricks REST APIs intro
-	Leveraging an external notebook with our secret scopes (updating a class to reflect what currently is marked as secret scopes)
-	Importing data into databricks (wine dataset)
-	Reading csv into Spark Dataframe & Pandas Dataframe
-	Adding columns to Spark Dataframe
-	Adding columns to Pandas Dataframe
-	Creating Tables
-	Creating plots for analysis
-	Create Model
-	Registering a model
-	Improve Model
-	Archiving Model and moving previous model to archive
-	Registering Model in Azure ML and Deploying to AKS Cluster


