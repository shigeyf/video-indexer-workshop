# Module #01 - Basic AI result extraction

The goal of this workshop is to upload a VOD video and/or audio files to Azure Video Indexer to do speech-to-text, face detection, and other AI analysis. We will deploy the required services, connect them together and store the result in a database.

## Prerequisites

In preparation, please make sure that you have done all steps in [Connect-to-Azure](../Connect-to-Azure/README.md).
You can continue to use your Azure Subscription that you have:

* An Azure subscription
* An Azure Active Directory (AD) domain
* A user account in your Azure AD domain with a *Work or School* type account

## Step 1 - Deploying required services

In this step, you will deploy required services for Video Indexer workflows.

Please go to the [Azure portal](https://portal.azure.com) and sign in to your Azure subscription to do the following operations:

* Create a Resource Group
* Create a Storage account
* Create a Cosmos DB account

### Step 1-1 : Create a Resource Group

All Azure services are part of the Resource Group. This is nothing more but a logical grouping of resources which will help you with organizing your services. You can create a resource group by navigating to [**Resource Groups**] in the portal and clicking [**+Add**].

The details of Resource Group are simply a name (unique to your subscription) and a location. In *Create an empty resource group* blade, please follow the steps below:

* Enter a Resource Group name for this lab in *Resource group name*
* Select a subscription in *Subscription*
* Select a location in *Resource group location*

Then, click [**Create**] button. All services that you will deploy next will be grouped into this resource group. Please open the Resource Group after creation.

### Step 1-2 : Create a Storage account

Next, you will deploy a Storage account, which will act as a storage for uploading video source files for video/audio analytics.

Go to the Resource Group, then select [**+Add**] at the top left corner. Search for "***Storage account***" and select **Storage account - blob, file, table, queue**, then click [**Create**] button.

In *Create storage account* blade, please follow the steps below:

* Enter a storage account name in *Name* that is all lower case and without special characters
* Select ***Resource Manager*** in *Deployment model*
* Select ***StorageV2*** in *Account kind*
* Select ***LRS*** in *Replication*
* Select ***Use existing*** in *Resource Group*
* Find the resource group in the drop-down list of *Resource Group*, that you created in the previous step
* Leave all other settings as default

Then, click [**Create**] button. Once the storage account is created, go to the to the storage account using the *Deployment succeeded* in Toast notification or in the list of notification by clicking the bell icon at the top, then click [**Go to resource**] button.

Next, in [**Overview**] menu of *Storage account* blade, click [**Blobs**] under services in the middle of the page.

Create a container named "***uploads***" by clicking [**+Container**] button, and leave the public access level to ***Private (no anonymous access)*** and then click [**OK**] button.

### Step 1-3 : Create a Cosmos DB account

Next, you will deploy a Cosmos DB account, which will act as a datastore for the video/audio analytics results.

Go to the Resource Group, then click [**+Add**]. Search for "***Cosmos DB***" and select **Azure Cosmos DB**, then click [**Create**] button to create a new account.

For Cosmos DB account creation parameters in *Azure Cosmos DB - New account* blade, please follow the steps below:

* Enter Cosmos DB ID which will be a DNS qualified name of Cosmos DB service
* Select SQL as the API type
* Select ***Use existing*** in *Resource Group*
* Find the resource group in the drop-down list of *Resource Group*, that you created in the previous step
* Leave all other settings as default

Then, click [**Create**] button. It will take a couple of minutes to create a new account. Once the Cosmos DB account is created, you will need to create a new collection for your analysis data to be saved in. Go to the Cosmos DB account using the *Deployment succeeded* in Toast notification or in the list of notification by clicking the bell icon at the top, then click [**Go to resource**].

Next, in [**Data Explorer**] menu of *Azure Cosmos DB account* blade, click [**New Collection**] at the top. The collection is essentially a database table. Please enter the following values:

* *Database id*: click [**Create new**] and enter a name of your choice
* *Collection id*: Enter a name of your choice
* *Storage capacity*: Change to [***Fixed 10GB***]
* *Throughput*: Please change this to ***400*** to save cost

Then, click [**OK**] button

## Step 2 – Deploying Logic App Workflows

In this step, you will deploy two Logic Apps for your custom media workflow automation, such as uploading your video/audio files to Video Indexer, executing video/audio analysis to the uploaded files, storing analysis result data to the Cosmos DB.

Two Logic App workflows are connected via callback feature in Video Indexer API. Once your video has been uploaded and analyzed in Video Indexer in 1st workflow that you will create in this step, a callback will be triggered from Video Indexer and 2nd workflow that you will also create in this step will be started.

### Step 2-1 : Create a Logic App for 1st Workflow

Go to the Resource Group, then select [**+Add**] at the top left corner. Search for "***Logic App***" and select **Logic App**, then click [**Create**] button to create a new account for 1st logic app.

For Logic App account creation parameters in *Create logic app* blade, please follow the steps below:

* Enter a Logic App name that is unique to your subscription
* Select ***Use existing*** in *Resource Group*
* Find the resource group in the drop-down list of *Resource Group*, that you created in the previous step
* Leave all other settings as default

Then, click [**Create**] button. Once the Logic App is created, go to the to the Logic App account using the *Deployment succeeded* in Toast notification or in the list of notification by clicking the bell icon at the top, then click [**Go to resource**].

Once *Logic Apps Designer* blade is opened, go down to *Templates* area and select **Blank Logic App** template.

### Step 2-2 : Create a workflow Trigger (1st workflow)

First, you will create a Trigger, which starts the Logic App workflow with every Blob creation event.

To add a Trigger to the Logic App, search for "***Event Grid***" and select the Trigger called **When a resource event occurs**.

Select your Azure AD Tenant in Azure Event Grid trigger and click [**Sign in**] button, then select your appropriate user account for sign-in if prompted.

For the parameters for Event Grid trigger, please follow the steps below:

* Select your subscription from *Subscription* drop-down list
* Select "**Microsoft.Storage.StorageAccounts**" from *Resource Type* drop-down list
* Select your Storage account name (created at Step 1) from *Resource Name* drop-down list.
* Click *Show advanced options*
  * Enter "**.mp4**" to Suffix Filter box

### Step 2-3 : Define Workflow Variables (1st workflow)

Click [**+Next step**], search for "***Variables***" and select **Initialize variable** action.

Please add two actions of **Initialize variable** to declare the following two **String** variables. The values for both variables can be captured from *Account* Settings of your Video Indexer account.

* ***VideoIndexerLocation***: A location for your Video Indexer account
  * Enter ***VideoIndexerLocation*** in *Name*
  * Select **String** in *Type*
  * Enter your Video Indexer location in *Value*
    * If you use Free Trial account, you will set **trial** string as String type variable.
    * Otherwise, you will set a region name as String type variable.
* ***VideoIndexerAccountID***: An account ID for your Video Indexer account
  * Enter ***VideoIndexerAccountID*** in *Name*
  * Select **String** in *Type*
  * Enter your Video Indexer Account ID in *Value*
    * You will set GUID string of your Account ID as String type variable.

### Step 2-4 : Create a conditional flow for ‘BlobCreated’ event (1st workflow)

Click [**+Next step**], search for "***Condition***" and select **Condition** action.

Please configure **Condition** control action with the following condition criteria:

* In the left box: select **Event Type** value of **Event Grid** trigger action from the *Dynamic content* popup
* In the middle box: set **is equal to**
* IN the right box: enter "***Microsoft.Storage.BlobCreated***"

### Step 2-5 : Create a blob path from EventGrid event information (1st workflow)

Click [**Add an action**] in *If true* box, search for "***Compose***" and select **Compose** action.

Please put the cursor at *Inputs* of this action, select *Expression* tab on the popup, enter the following value in the function (fx) field, then click [**OK**] button:

```c#
    split(triggerBody()?['subject'], '/')?[4]
```

Enter ‘/’ character after the ‘split’ function in *Inputs*.

Then, select *Expression* tab again, enter the following value in the function (fx) field, then click [**OK**] button:

```c#
    split(triggerBody()?['subject'], '/')?[6]
```

### Step 2-6 : Get Blob SAS URI for uploaded a video/audio file (1st workflow)

Click [**Add an action**] after the previous action, search for "***Blob***", and select **Create SAS URI by path** action.

At the first time of using Azure Blob Storage action, you will need to create a connection to connect with your Blob Storage account. Please follow the steps below:

* Enter any connection name (any value)
* Select Storage account which was created at the previous step
* Then, click [**Create**] button

Then, configure **Create SAS URL by path** action with:

* *Blob path*: select **Output** value of **Compose** action from the previous action of *Dynamic content* in the popup
* Click *Show advanced options*
  * *Expiry Time*: put the cursor in *Expiry Time*, select *Expression* tab on the popup, enter the following value in the function (fx) field, then click [**OK**]:

```c#
    addHours(utcNow(), 1)
```

### Step 2-7 : Get Access Token for Video Indexer account (1st workflow)

Click [**Add an action**] after the previous action, search for "***Video Indexer***", and select **Get Account Access Token** action.

At the first time of using Video Indexer (V2) action, you will need to create a connection to connect with your Video Indexer. Please follow the steps below:

* Enter any connection name (any value)
* Enter an API key value for your Video Indexer account
  * The API key can be retrieved from the [Video Indexer API Portal](https://api-portal.videoindexer.ai/).
* Then click [**Create**] button

Then, configure **Get Account Access Token** action with:

* *Location*: select *Enter custom value* from the drop-down list, then select **VideoIndexerLocation** variable from *Dynamic content* on the popup.
* *Account ID*: select *Enter custom value* from the drop-down list, then select **VideoIndexerAccountID** variable from *Dynamic content* on the popup.
* *Allow Edit*: select **true**

### Step 2-8 : Upload and Index a video/audio file to your Video Indexer account (1st workflow)

Click [**Add an action**] after the previous action, search for "***Video Indexer***", and select **Upload video and index** action.

Then, configure **Upload video and index** action with:

* *Location*: select *Enter custom value* from the drop-down list, then select **VideoIndexerLocation** variable from *Dynamic content* on the popup.
* *Account ID*: select *Enter custom value* from the drop-down list, then select **VideoIndexerAccountID** variable from *Dynamic content* on the popup.
* *Video Name*: select *Expression* tab on the popup, enter the following value in the function (fx) field, then click [**OK**]:

```c#
    split(triggerBody()?['subject'], '/')?[6]
```

* *Access Token*: select **Access Token** of **Get Account Access Token** action from *Dynamic content* on the popup

You will need to set *Callback URL* in the *advanced options* later. This is to trigger another Logic App workflow after this action has been finished. Once 2nd workflow is created, you will come back to this 1st workflow to update this action.

### Step 2-9 : Final view of 1st workflow

Click [**Save**] button to save your working workflow. The final view of 1st workflow is as below.

![Screen capture](images/overview-1st-logic-app-workflow.png?raw=true)

### Step 2-10 : Create a Logic App for 2nd Workflow

Create another logic app for a workflow after video indexing has been done in your Video Indexer account.

Go to the Resource Group, then select [**+Add**] at the top left corner. Search for “***Logic App***” and select **Logic App**, then click [**Create**] button to create a new account for 1st logic app.

For Logic App account creation parameters in *Create logic app* blade, please follow the steps below:

* Enter a Logic App name that is unique to your subscription
* Select ***Use existing*** in *Resource Group*
* Find the resource group in the drop-down list of *Resource Group*, that you created in the previous step
* Leave all other settings as default

Then, click [**Create**] button. Once the Logic App is created, go to the to the Logic App account using the *Deployment succeeded* in Toast notification or in the list of notification by clicking the bell icon at the top, then click [**Go to resource**].

Once *Logic Apps Designer* blade is opened, go down to *Templates* area and select **Blank Logic App** template.

### Step 2-11 : Create a workflow trigger (2nd workflow)

First, you will create a Trigger, which starts the Logic Apps workflow with every HTTP request.

To add a Trigger to the Logic Apps, search for "***HTTP request***" and select the Trigger called **When a HTTP request is received**.

### Step 2-12 : Define Workflow Variables (2nd workflow)

Click [**+Next step**], search for "***Variables***" and select **Initialize variable** action.

Please add two actions of **Initialize variable** to declare the following two **String** variables. The values for both variables can be captured from *Account* Settings of your Video Indexer account.

* ***VideoIndexerLocation***: A location for your Video Indexer account
  * Enter ***VideoIndexerLocation*** in *Name*
  * Select **String** in *Type*
  * Enter your Video Indexer location in *Value*
    * If you use Free Trial account, you will set **trial** string as String type variable.
    * Otherwise, you will set a region name as String type variable.
* ***VideoIndexerAccountID***: An account ID for your Video Indexer account
  * Enter ***VideoIndexerAccountID*** in *Name*
  * Select **String** in *Type*
  * Enter your Video Indexer Account ID in *Value*
    * You will set GUID string of your Account ID as String type variable.

### Step 2-13 : Get Access Token for Video Indexer account (2nd workflow)

Click [**+Next step**] after the previous action, search for "***Video Indexer***", and select **Get Account Access Token** action.

Then, configure **Get Account Access Token** action with:

* *Location*: select *Enter custom value* from the drop-down list, then select **VideoIndexerLocation** variable from *Dynamic content* on the popup.
* *Account ID*: select *Enter custom value* from the drop-down list, then select **VideoIndexerAccountID** variable from *Dynamic content* on the popup.
* *Allow Edit*: select **true**

### Step 2-14 : Get Video Index for uploaded video/audio file from Video Indexer account (2nd workflow)

Click [**+Next step**] after the previous action, search for "***Video Indexer***", and select **Get Video Index** action.

Then, configure “Get Video Index” action with:

* *Location*: select *Enter custom value* from the drop-down list, then select **VideoIndexerLocation** variable from *Dynamic content* on the popup.
* *Account ID*: select *Enter custom value* from the drop-down list, then select **VideoIndexerAccountID** variable from *Dynamic content* on the popup.
* *Video ID*: select *Expression* tab on the popup, enter the following value in the function (fx) field, then click [**OK**]:

```c#
    encodeURIComponent(triggerOutputs()['queries']['id'])
```

* *Access Token*: select **Access Token** of **Get Account Access Token** action from *Dynamic content* on the popup

### Step 2-15 : Upload Video/Audio Analysis data to Cosmos DB (2nd workflow)

Finally, you will add an action to save the result to Cosmos DB.

Click [**+Next step**] after the previous action, search for "***Cosmos DB***", and select **Create or update document** action.

At the first time of using Cosmos DB action, you will need to create a connection to connect with your Cosmos DB account. Please follow the steps below:

* Enter any connection name (any value)
* Select Cosmos DB account which was created at the previous step
* Then, click [**Create**] button

Then, configure **Create or update document** action with:

* *Database ID*: select the correct values from the drop-down list
* *Collection ID*: select the correct values from the drop-down list
* *Document*: select **Body** of **Get Video Index** action from *Dynamic content* on the popup

### Step 2-16 : Final view of 2nd workflow

Click “Save” button to save your working workflow. The final view of 2nd workflow is as below:

![Screen capture](images/overview-2nd-logic-app-workflow.png?raw=true)

### Step 2-17 : Configure callback in the 1st workflow

To get the callback URL, please follow the steps below:

* Go to the Resource Group, open 2nd logic app workflow
* Click [**Edit**] button
* Open **When a HTTP request is received** action
* Copy **HTTP POST URL**
* Close 2nd logic app workflow

To configure the callback URL, please follow the steps below:

* Go to the Resource Group, open 1st logic app workflow
* Click [**Edit**] button.
* Open **Upload video and index** action
* Paste **HTTP POST URL** to *Callback URL*
* Click [**Save**] button to save 1st logic app workflow

## Step 3 – Execute your workflow

Now to get end-to-end results, you can upload video/audio files to the ‘uploads’ blob container of your Storage account.

### Prepare scalable resources

Before uploading video/audio files and processing with Video Indexer, two steps must be done for scalable video/audio analysis processing.

1. Sign-in to the Azure Portal and go to your Azure Media Services account which was created in your preparation step. Open “Media Reserved Units”, select ***S3*** pricing tier with 1 Media Reserved Unit, then click [**Save**] button. ![Screen capture](images/set-ams-mru.png?raw=true)

2. Sign-in to your Video Indexer account, click your account icon at the right top corner, click **Settings** in the drop-down list. Click **Account** tab in Settings page and set **Autoscale** to [**On**]. ![Screen capture](images/set-vi-autoscale.png?raw=true)

### Getting Results

Go to the Storage account and click on Blobs then ‘uploads’. Then, you can now upload mp4 files using the [**Upload**] button at the top bar.

Once a video uploading is finished, it may take a few moments to detect ***BlobCreated*** event in EventGrid, then your 1st Logic App is triggered. Go to your 1st Logic App workflow to see the Runs history as below:
![Screen capture](images/runs-history-1st-logic-app-workflow.png?raw=true)

You can click one of the history entries to open the Logic App run details of your 1st workflow like below:
![Screen capture](images/logic-app-run-1st-logic-app-workflow.png?raw=true)

Next, go to your 2nd Logic App workflow to see the Runs history as below:
![Screen capture](images/runs-history-2nd-logic-app-workflow.png?raw=true)

Once Video Indexer Analysis is done, you can click one of the history entries to open the Logic App run details of 2nd workflow like below:
![Screen capture](images/logic-app-run-2nd-logic-app-workflow.png?raw=true)

You can also see the uploaded Video in Video Indexer. Also, in Cosmos DB a document will be saved with the full JSON result from this indexing process. This can be seen in *your Cosmos DB account* > *Data Explorer* > *Documents*:
![Screen capture](images/cosmos-db-data-document.png?raw=true)
