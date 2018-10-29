# Set up Video Indexer - Connect to Azure

When creating a Video Indexer account, you can choose:

* a free trial account - where you get a certain number of free indexing minutes
* a paid option - where you are not limited by the quota

With free trial, Video Indexer provides up to 10 hours of free indexing to website users and up to 40 hours of free indexing to API users. With paid option, you create a Video Indexer account that is connected to your Azure subscription and an Azure Media Services account. You pay for minutes indexed as well as the Media Account related charges.

 Paid option has the following characteristics compared to free trial account:

* Manage video/audio files directly in your Azure Storage account in your Azure Subscription
* Meet with many security & compliances in line with what Microsoft Azure cloud platform provides
* Provide the scalability of computing resources for Video/audio analysis processing

## Prerequisites

Before starting this step, please make sure that you have:

* An Azure subscription
* An Azure Active Directory (AD) domain
* A user account in your Azure AD domain with a *Work or School* type account
  * which can create a new user account of *Work or School account* (a user account must have either *Global administrator* role or *User administrator* role in your Azure AD)
  * or which is *Work or School account* and has either an *Owner* role, or both *Contributor* and *User Access Administrator* roles.

You will need to use the user account in your Azure AD domain, when connecting your Video Indexer account to Azure. The user account in you Azure AD domain must be:

* an Azure AD user with a work or school account, not a personal account, such as outlook.com, live.com, or hotmail.com, etc.
* a member in your Azure subscription with either an *Owner* role, or both *Contributor* and *User Access Administrator* roles.

## Step 1 - Preparation before Sign-up in Video Indexer

You will do the following steps for the preparation before Video Indexer Sign-up.

1) Create a *Work or School account* in your Azure AD
2) Register EventGrid resource provider using the Azure portal
3) Login Azure portal with new *Work or School account*

### Step 1-1 : Create a *Work or School account*

Please go to the [Azure portal](https://portal.azure.com) and sign in to your Azure subscription with your account in order to create a new *Work or School account*. Please see more details in the [document](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/add-users-azure-active-directory#add-a-new-user) for Azure AD.

If you have a user account for your Azure Subscription which is *Work or School account* and has either an *Owner* role, or both *Contributor* and *User Access Administrator* roles, you can skip this Step 1-1.

The detail steps are here:

* In the Azure portal, at the left menu (or All services), go to *Active Directory* > *User*, then click *+ New user* to create a new *Work or School account*.
* In the [User] blade, you will need to the following configurations, then click *Create* button:
  * Enter Name for this new user
  * Enter User name (this must be an email address with your Azure AD domain, such as @foobar.onmicrosoft.com)
  * Click *Profile* and enter First name and Last name for this user
  * Click *Directory role* and select *User* in the [Directory role] blade
  * Check *Show Password* checkbox to confirm an initial password for this account
* Next, you will need to add this new user account to your Azure Subscription. In the Azure Portal, at the left menu (or All services), go to *Subscriptions* > [your Azure Subscription name] > *Access control (IAM)* to add two permissions to a new user you created in the previous step. You will need to repeat two times of following configurations for both *Contributor* and *User Access Administrator* roles:
1. Click *+ Add* button
2. Select a target role in *Role* pull-down menu to be added in [Add permissions] blade
3. Select a user *Select* area
4. Then click “Save” button

### Step 1-2 : Register EventGrid resource provider using the Azure portal

Please go to the [Azure portal](https://portal.azure.com) and sign in to your Azure subscription with your account.
In the Azure portal, go to *Subscriptions* > [your Azure Subscription name] > *Resource providers*.
Enter "*EventGrid*" into search box, then the status of “Microsoft.EventGrid” resource provider will be shown. If not in the "Registered" state, click Register. It takes a couple of minutes to register.
Click *Refresh* button, then you will see it as “Registered”.

### Step 1-3 : Login Azure portal with new *Work or School account*

Please sign out from the Azure portal, and select *+ Use another account*, then login with new account.
At the first login, you will be prompted with “Update your password”. Enter a current temporal password and new password to update your account password.

## Step 2 - Sign up in Video Indexer with *Work or School account*

Please go to the [Video Indexer web site](http://video.ai) and select "Sign up for free" to sign up with your *Work or School account* which was created in the previous Step 1. You will automatically get a new account for Video Indexer in this Sign-up action.
If you are asked by "Sign-In" screen, please select “Sign in with a corporate account” option in the "Sign-In" screen, and enter your new *Work or School account* (with email address and password) which is created in Azure AD.

After log in to the Video Indexer portal, your account detail is displayed in the icon of top-right corner. This is a free trial account of Video Indexer.

## Step 3 - Connect your Video Indexer account to Azure

In the [Video Indexer portal](https://www.videoindexer.ai/), click *Connect to Azure* button to create a new Azure Media Services account in your Azure Subscription and connect your Video Indexer account with it.

Please follow steps below:

1. Select a right Azure subscription to create a new Azure Media Services account
2. Select an appropriate region for your Video Indexer usage
    * As of September 2018, Video Indexer account in Paid option is located only in 3 regions (East Asia, North Europe, West US2)
3. Select *Create new resource group* and input new resource group name
4. Click *Connect* button

This operation will create a new resource group with one Azure Storage account and one Azure Media Services account in your Azure Subscription.

### Optional notes in Step 3

You can connect your Video Indexer account with your existing Azure Media Services account. There are two cases to be considered.

* If you have an Azure Media Services account in the same region with your new Paid Video Indexer account
* If you have an Azure Media Services account in a region which is different from a region of your new Paid Video Indexer account

#### Case 1 : Same region between AMS and VI account

If you will create a Video Indexer account in a region which is same with your existing Azure Media Services account, you can follow steps below when operating *Connect to Azure* in Video Indexer portal:

1. Select a right Azure subscription to create a new Azure Media Services account
2. Select an appropriate region for your Video Indexer usage
3. Select *Use existing resource group* and select Azure Media Services account in the pull down menu
4. Click *Connect* button

#### Case 2 : Different region between AMS and VI account

If you will create a Video Indexer account in a region which is different from your existing Azure Media Services account, you can still connect your Video Indexer account with your existing Azure Media Services with the following steps below when operating *Connect to Azure* in Video Indexer portal.

This step requires Service Principal authentication configuration in your Azure Media Services account. Please see details in the [Azure Media Services document](https://docs.microsoft.com/en-us/azure/media-services/previous/media-services-portal-get-started-with-aad#service-principal-authentication).

1. Click *Switch to manual configuration* at the bottom of *Connect Video Indexer to an Azure subscription* dialog.
    * Select an appropriate region for your Video Indexer usage in *Video Indexer account region*
    * Input your Azure Active Directory tenant domain name in *Azure Active Directory (Azure AD) tenant*
    * Input your Azure Subscription ID in *Azure subscription ID*
    * Input your existing Resource Group name (which has your existing Azure Media Services account) in *Azure Media Services resource group name*
    * Input your existing Azure Media Services account name in *Media service resource name*
    * Input your service principal account ID and password (which is associated with your existing Azure Media Services account) in *Application ID* and *Application key*
2. Click *Connect* button
