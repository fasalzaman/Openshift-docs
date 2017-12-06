# AEC Open Shift | Lab Guide

<!-- TOC -->

1. [Workshop Architecture and Objective](#workshop-architecture-and-objective)
2. [Lab 1: Introduction to Azure Portal](#lab-1-introduction-to-azure-portal)
3. [Lab 2: Deploying Open Shift cluster using ARM templates](#lab-2-deploying-open-shift-cluster-using-arm-templates)
4. [Lab 3: Deploying workload on Openshift](#Lab-3-deploying-workload-on-openshift)
5. [Lab 4: Integration of ACR with OpenShift](#Lab-4-integration-of-acr-with-openShift)

<!-- /TOC -->

## Workshop Architecture and Objective
 As part of Hand on Lab, you will get following details via email. Make a note of these details as these shall be leveraged throughout the lab exercise
- Azure Access: Azure Username and Password (Default Password, Change at first login)

### Labs Objective
During this lab, you will deploy Open Shift cluster on Azure and integrate Azure AD Authentication and Azure Container Registry into Open Shift. Detailed steps to achieve this is as follows.
- Get Familiar with Azure Portal and Ansible Tower UI
-	Create an Azure AD Application for Authentication
-	Create a key vault to store SSH Key
-	Deploy Open Shift using ARM Template
-	Configure Azure AD Authentication
-	Deploy 2 Tier App on Open Shift
-	Integration of Azure Container Registry with Open Shift

### Workshop Architecture after deploying ARM Template
Following illustrates the architecture in your Azure deployment after completion of exercise’s part of workshop.
<img src="images/1workshop_arch.jpg"/>
## Lab 1: Introduction to Azure Portal
### Lab Overview
This lab will take you through Azure login and portal experience.

### Prerequisites
-	Windows or a Mac machine with HTML5 supported browser such as Microsoft Edge, Internet Explorer, Chrome or Firefox
-	You should have registered in the training portal https://azuretraining.spektrasystems.com and received the confirmation message with the credentials to login to the [Azure portal](http://portal.azure.com).
-	Red Hat Customer Portal login credentials so that the Azure instances can be registered with Red Hat Subscription Manager properly, and you must have enough OpenShift Container Platform entitlements to cover the chosen configuration.
<img src="images/2registration_notification.jpg"/>

### Time Estimate

10 minutes

### Exercise 1: Log into your Azure Portal

In this exercise, you will log into the Azure Portal using your Azure credentials.
1.	**Launch** a browser and **Navigate** to https://portal.azure.com. Provide the credentials that you received via email. Click on **Sign In**.
<img src="images/3azure_login.jpg"/>

```
Note : At the first login, you may have to change the password, if asked for.
```

2.	**Enter** a new **password**. Then select **Update password and sign in**.
<img src="images/4update_password.jpg"/>

3.	Now, you will be directed to the Azure Dashboard
<img src="images/5azure_dashboard.jpg"/>

### Exercise 2: Verify access to the Subscription
In this exercise, you will verify the type of role you are assigned in this Subscription.

1.	**Launch** a browser and **Navigate** to https://portal.azure.com. **Login** with the Microsoft Azure credentials you received via email.
<img src="images/6azure_dashboard.jpg"/>

2. **Click** on **Microsoft Azure**  at the top left corner of the screen, to view the Dashboard.
<img src="images/7microsoftazure.jpg"/>

3.	To toggle **show/hide** the Portal menu options with icon, **Click** on the **Show Menu** button. 
<img src="images/8azure_menu.jpg"/>

4.	**Click** on the **Resource groups** button in the **Menu navigation** bar to view the **Resource groups** blade.
<img src="images/9resourcegroup.jpg"/>

5.	You will see a Resource Group which you have access to, **click** on it.

<img src="images/10select_rg.jpg"/>

```
Note:
The Resource Group shown here is for demo purpose only. Actual name of the Resouce Group that you see may differ.
```

6.	From the Resource Group blade that come up, **Select** the Access Control ( IAM ) which is on the left side of the blade.
<img src="images/11access_control.jpg"/>

7.	In the new blade that come up, you can see the **role** that is assigned to you.
<img src="images/12role.jpg"/>

## Lab 2: Deploying Open Shift cluster using ARM templates
### Lab Overview
In this lab, you will learn how to deploy the Open Shift Cluster on Azure using ARM templates.
### Prerequisites
•	Lab 1 must be completed

### Time Estimate
120 minutes

### Exercise 1: Create an Azure AD Application
In this exercise, you will create an Azure AD App and retrieve the Client ID and Client secret values.
1.	**Launch** a browser and **Navigate** to https://portal.azure.com. **Login** with the Microsoft Azure credentials you received via email.
<img src="images/13azuredashboard.jpg"/>

2.	**Click** on the **Azure Active Directory** button in the **Menu navigation** bar to view the **Azure Active Directory** blade.
<img src="images/14selectazure_ad.jpg"/>

3.	You will be directed to the Azure Active Directory blade, **click** on **App registrations**.
<img src="images/15app_reg.jpg"/>

4.	In the next blade, **click** on **New Application Registration** on top of the blade.
<img src="images/16new_appreg.jpg"/>

5.	In the **Create** blade, **configure** as follows:

-	Name: **(Provide a unique value)**
-	Application type: **Web app/API**
-	Sign-on URL: https://contoso.com

```
Note: We will change this value later during the lab.
```

And then **click** on **Create**.

<img src="images/17createapp.jpg"/>

6.	You will be redirected to the **App registrations** blade. You can check the app has been created by typing the App Name in the search field.
<img src="images/18check_app.jpg"/>

If the app has been created, you can see it in the results as shown above.

7.	Click on the app you created and you will be directed to the App blade.

8.	Copy the Application Id and save it in a notepad or any text editor for later use.
<img src="images/19app_id.jpg"/>

9.	Now, **Click** on **Keys** in the settings blade.
<img src="images/20app_key.jpg"/>

10.	In the **Keys** blade, **configure** as follows:

- Description: **key1**
- Expires: **Never expires**

And **Click** on **Save.**

<img src="images/21save_key.jpg"/>

11.	After you click on save, the key value will be displayed which is the Client Secret.
Copy the value into the text editor where you saved the value of Application Id for later use.
<img src="images/22copy_key.jpg"/>

### Exercise 2: Create a Keyvault 
In this exercise, you will configure Azure Bash Cloud Shell and create a Key vault in the existing resource group and store the SSH key inside the vault. 
1.	**Launch** a browser and **Navigate** to https://portal.azure.com. **Login** with the Microsoft Azure credentials you received via email.
<img src="images/23azure_dashboard.jpg"/>

2.	**Click** on **Cloud Shell**  at the top right corner of the screen, to open the cloud shell.
<img src="images/24cloudshell.jpg"/>

3.	Then **Click** on **Bash ( Linux )**, and in the next page, **click** on **Show advanced settings**
<img src="images/25selectbash.jpg"/>
<img src="images/26advanced_settings.jpg"/>

4.	In the new blade, select the existing resource group, provide unique names under Create new(Storage account and File share) and **click** on **Create Storage**.
<img src="images/27create_storage.jpg"/>

5.	In a few minutes, the bash shell will come up.
<img src="images/28bashshell.jpg"/>

6.	Now execute the following command in the cloud shell to create a key vault in the existing resource group.
```
az keyvault create -n <uniquename> -g <ResourceGroup> -l <LocationOfResourceGroup> 
--enabled-for-template-deployment true
```
```
Note:
Provide the existing Resource Group name, it’s location and a unique name for key vault in the above command when executing
```
<img src="images/29create_keyvault_bash.jpg"/>

7.	Now execute the following command in the cloud shell to generate ssh key.
```
ssh-keygen
```
```
Note: Keep on clicking enter button until the key has been created.
```
<img src="images/30generate _ssh.jpg"/>

8.	Now execute the following command in the cloud shell to display the public ssh key. Copy the key into a text editor for later use.
```
cat .ssh/id_rsa.pub
```
```
Note: 
The copied SSH Key should be made into a single line. You will need this key for later use.
```
<img src="images/31display_publickey.jpg"/>

9.	Now execute the following command to store the generated key in the key vault.
```
az keyvault secret set --vault-name <keyvaultname> -n osdemovaultsecret --file ~/.ssh/id_rsa
```
```
Note:
Substitute for key vault name in the above command with the name of the keyvault created earlier when executing.
```
<img src="images/32store_key.jpg"/>

## Exercise 3: Deploy Openshift Cluster using ARM Template  
In this exercise, you will deploy the Openshift cluster on Azure using ARM Template . 
1.	**Launch** a browser and **Navigate** to https://github.com/SpektraSystems/openshift-container-platform

2.	Now **click** on **Deploy to Azure** button and you will be redirected to the azure portal. If prompted **login** with the Microsoft Azure credentials you received via email.
<img src="images/33deploytemplate.jpg"/>

3.	Now you will be directed to the Custom Deployment blade.
<img src="images/34custom_deployment.jpg"/>

4.	In the **Custom Deployment** blade, **configure** the settings as follows:
-	Resource Group : Choose Use **existing** and scroll down to see the Resource Group which is already there)
-	Openshift Password  :  **Provide a unique password**
-	Ssh Public Key :  **Provide the copied SSH key**
-	Rhsm Username or Org Id: **Provide the username of Redhat credentials**
-	Rhsm Password or Org Id: **Provide the password of Redhat credentials**
-	Rhsm Pool Id: **Provide the pool Id of Redhat OpenShift Subscription**
-	Key Vault Name : **Provide the Key Vault name you provided**
-	Key Vault Secret : **osdemovaultsecret**
-	Aad Auth App Name : **Provide the name of the AD App you created** 
-	Aad Auth App Id : **Provide the Client ID of the AD App you created** 
-	Aad Auth Client Secret : **Provide the secret key of the AD App**
**And** accept the terms of conditions.
<img src="images/35customdeployment_configure.jpg"/>
<img src="images/36custeomdeployment_purchase.jpg"/>

5.	And then **click** on **Purchase**.

6.	Once the deployment starts, you can see the progress in the notification bar at the top of the Azure portal.
<img src="images/37deployment_progress.jpg"/>

7.	Once the deployment is complete, you can see it in the notifications tab as Deployment succeeded. Now, **click** on **Go to resource group** from the notifications tab.
<img src="images/38dep_succeed.jpg"/>

8.	In the resource group blade that come up, you can see the deployments as Succeeded, click on that.
<img src="images/39resource_group.jpg"/>

9.	Select **Microsoft Template** from the new blade that come up.
<img src="images/40dep_status.jpg"/>

10.	From the new blade that come up, you can see the outputs of the deployment.
<img src="images/41dep_output.jpg"/>

11.	Copy the Openshift Console URL, Bastion DNS FQDN and OpenShift Master SSH by clicking on Copy to a text editor

12.	To verify that the deployment is working, **Open** a new tab in the browser and paste the copied URL.
```
Note: Skip the certificate warning
```

13.	Now you will be directed to the Openshift Console Login page.
<img src="images/42openshift_console.jpg"/>

```
Note: If the above page comes up, then the deployment is working.
```

## Exercise 4: Configure Azure AD Authentication
In this exercise, you will configure the AD App you created for Authentication into the Open Shift console.
1.	**Launch** a browser and **Navigate** to https://portal.azure.com. **Login** with the Microsoft Azure credentials you received via email.
<img src="images/43az_dashboard.jpg"/>

2.	**Click** on the **Azure Active Directory** button in the **Menu navigation** bar to view the **Azure Active Directory** blade.
<img src="images/44az_ad.jpg"/>

3.	You will be directed to the Azure Active Directory blade, **click** on **App registrations**.
<img src="images/45app_reg.jpg"/>

4.	You will be redirected to the **App registrations** blade. You can search the App by typing the name of the App you created earlier, in the search field.
<img src="images/46select_app.jpg"/>

5.	Click on the app you created and you will be directed to the App blade.
<img src="images/47app_blade.jpg"/>

6.	Now Click on Properties under Settings blade.
<img src="images/48app_properties.jpg"/>

7.	In the **Properties** blade, **edit** as follows:
-	App ID URI: (Provide the Open Shift Console URI)
-	Home Page URL type: (Provide the Open Shift Console URI)
And then **click** on **Save**.
<img src="images/49save_properties.jpg"/>

8.	Once you save the properties, close the properties blade.
<img src="images/50close_properties.jpg"/>

9.	Then you will be redirected to the Settings Blade of AD App. Click on the Reply URLs
<img src="images/51reply_url.jpg"/>

10.	Now modify the openshift console url by removing the ‘console’ from the end and appending ‘oauth2callback/AzureAD’ to the url and provide it in the Reply URL blade that come up and then Click on Save. 
<img src="images/52replyurl_save.jpg"/>

11.	Now go back to the setting blade of the App and Click on Required permissions.
<img src="images/53req_permission.jpg"/>

12.	Click on Grant Permissions in the blade that come up and then **Click** on **Yes**
<img src="images/54grant_permission.jpg"/>
<img src="images/55grantpermission_yes.jpg"/>

13.	Now go back to the **Active Directory** blade by clicking on Azure Active Directory button in the **Menu navigation** bar.
<img src="images/56az_ad.jpg"/>

14.	Click on Enterprise Applications from the menu on the left side.
<img src="images/57enterprise_app.jpg"/>

15.	In the new blade that come up, click on **All applications**
<img src="images/58all_app.jpg	"/>

16.	In the new blade that come up, edit the filter as follows: 
-	Show: Select All Applications
-	Application status: Any
-	Application visibility: Any
And then **click** on **Apply**.
<img src="images/59all_application.jpg"/>

17.	You can search the App by typing the name of the App you created earlier, in the search field.
Select the App from the results.
<img src="images/60searchapp.jpg	"/>

18.	You will be redirected to the App Blade. Click on Properties under Manage section on the left side of the properties blade.
<img src="images/61properties.jpg"/>

19.	In the new blade that come up, select User Assignment Required and Click on Save.
<img src="images/62save_properties.jpg"/>

20.	Now, click on **Users and groups** under Manage section on the left side of the App Blade.
<img src="images/63user_groups.jpg"/>

21.	In the blade that come up, click on **+Add user** to assign a user to the app.
```
Note: If user is already added then skip next three steps.
``` 
<img src="images/64adduser.jpg"/>

22.	In the Add Assignment blade that come up, click on Users and then select the id with which you logged in to Azure portal and click on **Select**.
<img src="images/65add_assignment.jpg"/>

23.	Now you will be redirected to the Add Assignment blade. Click on Assign to assign the user to the app.
<img src="images/66assign.jpg"/>

24.	Now to verify that the user is able to authenticate to Openshift console via Azure AD, **Open** a new tab in the browser and paste the Openshift Console URL which you copied earlier.
```
Note: Skip the certificate warning
```
<img src="images/67openshift_console.jpg"/>

25.	Now click on AzureAD, you will be redirected to the Login Page. Provide the Azure credentials you received via email over there and click on Sign in.
<img src="images/68sign_in.jpg"/>

26.	Once the login is successful, you will be redirected to the Openshift console.
<img src="images/69openshift_cp.jpg"/>
 





