# Lab: Extract and Load with ADF

## Table of Contents

Summary<br/>
Pre-requisites<br/>
Scenario<br/>
Part 1 – Create an Azure Data Factory (v2)<br/>
Part 2 – Connect ADF to a code repository to begin using the ADF GUI (Optional)<br/>
Part 3 – Setting up the Connections in the ADF GUI (Azure SQL Database -> Blob)<br/>
Part 4 – Setting up the Copy Activity in the ADF GUI (Azure SQL Database -> Blob)<br/>
Part 5 – Using Parameters and Triggers (scheduling) in ADF GUI<br/>

## Summary
This tutorial walks through creating a pipeline copy activity to copy a file from a S3 storage location to an Azure blob storage container, so we can prepare the file to be processed later for transformation.<br/>

In this lab we will:<br/>
•	Show the graphical user interface for creating a pipeline<br/>
•	Copy CSV file via a Copy Activity<br/>
•	Copy Structed data from SQL Azure via a Copy Activity<br/>
•	Use parameters to make the pipeline easy to change and more reusable<br/>
 
## Prerequisites
•	Azure Subscription with rights to use/deploy Azure services, and X of Azure credit<br/>
•	An ADLS Gen2 storage account with a filesystem (container)<br/>
•	Azure Data Factory<br/>
•	Visual Studio Team Services Git project (optional)<br/>

## Scenario
### Part 1 – Create an Azure Data Factory (v2)
We are going to use the portal to create the Azure Data Factory.
1.	Go to Azure Portal- https://portal.azure.com.<br/>
2.	Select **New** on the left menu, select **Analytics**, and then select **Data Factory**.<br/>
    <img src="images/ex01.jpg"/><br/>
3.	On the New data factory page, enter **ad-flab-adf** for Name.
4.	Make sure you select **Version** as **V2 (Pre-view)**<br/>
   <img src="images/ex03.jpg"/><br/>

``
Note: The name of the Azure data factory must be globally unique. Please modify the name if the Name validation fails. 
``

5.	After the creation is complete, you see the **Data Factory** page. Select the **Author & Monitor** tile to start the **Azure Data Factory** user interface (UI) application on a separate tab.<br/>
   <img src="images/ex02.jpg"/><br/>
### Part 2 – Connect ADF to a code repository to begin using the ADF GUI (Optional)
One option to be able to sync our code is to connect ADF to a code repository. This section walks through the steps to connect ADF to a Visual Studio Team Services Git project, so we can save our code for later re-use. Note that this is not required but a recommended best practice.

1.	Navigate to the **Azure portal** within your web browser and navigate to https://portal.azure.com
2.	Open the Azure Data Factory blade **adflab-adf**.
3.	In the Overview Blade, Click on **Author and Monitor**<br/>
   <img src="images/ex03.jpg"/><br/>
4.	Click the **Set up Code Repository** button to begin connecting this Azure data factory to a code repository.<br/>
   <img src="images/ex3.jpg"/><br/>
5.	You can either create a new VSTS account for this lab or use an existing one. You can create a new one - http://app.vsaex.visualstudio.com/project/profile/account and configure it in ADF GUI under repository settings. You need to create/ select a Project under this Account.<br/>
   <img src="images/ex04.jpg"/><br/>
6.	The Repository Settings pane will appear on the right.<br/>
   <img src="images/ex05.jpg"/><br/>
7.	Select **Use Exiisting** in select working branch page and click the **Save** button when you have verified your settings.


### Part 3 – Setting up the Connections in the ADF GUI (Azure SQL Database -> Blob)
We now want to use the GUI to create another copy activity in the same pipeline to copy the Data from Azure SQL DB to Azure blob storage to be ready for transformation along with the earlier CSV file. Our first step is setting up the connections and linked services need for the source and destination.

1.	In the Left Menu click the **Connections** menu item.
2.	Click the **+New** button under **Linked** Services.
3.	In the right pane you should now see the list of possible **Linked** Services. 
4.	Click on the** Azure SQL Database** tile.
5.	Click **Continue**.
6.	In the right pane you should see the properties to configure the **Azure SQL Database** account.We will name this linked service AzureSqlData-base-Source and using the Default runtime. Use the following  
 
  * Account Selection Method -> **Manual**
  * Fully qualified domain name ->**adlabserver.database.windows.net**
  * Database name -> **adflab**
  * User name -> **lab_user**
  * Password -> **P@ssw0rd**

7.	Click the **Test Connection** to verify settings are entered correctly.
8.	Click **Save**.

### Part 4 – Setting up the Copy Activity in the ADF GUI (Azure SQL Database -> Blob)
We now want to use the GUI to create a Copy Activity in the pipeline to move the files from the Azure SQL Database as source to our Azure storage destination.

1.	Click the **CopyPipeline** in the left menu to return to the pipeline GUI.
2.	In the **Pipeline GUI**, drag the Copy activity (under DataFlow) to the empty pane above General.
3.	Rename the activity to **AzureSQL-toAzureBlob**.
4.	Click **Save**.
5.	Click the **Source** Tab in the **Copy Activity GUI**.
6.	Click the **+New** button next to Source **Dataset**.
7.	You should now see the list of source **dataset** connectors. 
8.	Choose the **Azure SQL dataset** and click **Finish**.
9.	You should now add the connection property information.
10.	Name this **datasetAzureSqlTable** 
11.	Select Table -> **SalesLT.Customer**
12.	Click **Preview Data** to preview the first several data rows.
13.	Click on **CopyPipeline** and then click **Az-ureSQLtoAzureBlob** Activity.

14.	Select **Source -> Use Query -> Query**

``
SELECT CustomerID, CompanyName, SalesPerson, ModifiedDate 
FROM [SalesLT].[Customer]
``

``
Note: This Query may change based on your table selection. 
``

15.	Click on **Preview data** to ensure the query works.
16.	Click back on the **CopyPipeline**.
17.	Click the **AzureSQLtoAzureBlob** copy activity.
18.	Click the **Sink** Tab in the Copy Activity GUI.
19.	Click the **+New** button next to **Source Dataset**.
20.	You should now see the list of sink dataset connectors. 
21.	Choose the **Azure Blob storage** dataset and click ****Finish**.
22.	Name the dataset as ****datasetBlobfromSQL**
23.	Fill out the following information: **Linked Service -> AzStorage-Staging, File Path -> Click**. The Browse button and drill down to the **inputsql** container.

``
Note: Make sure **inputsql** container exists or else create one first. 
``

24.	Save the **dataset**
25.	Navigate to the **CopyPipeline**
26.	Click the **Test Run** icon at the top menu to test and run our copy activity.
27.	Final **CopyPipeline** looks like this

### Part 5 – Using Parameters and Triggers (scheduling) in ADF GUI
You can define parameters at the pipeline level and pass arguments while you're invoking the pipeline on-demand or from a trigger. Activities can consume the arguments that are passed to the pipeline. Using parameters, you can build more flexible pipelines. 
And triggers can be used to execute the pipelines on a schedule or on-demand.

1.	Navigate to **CopyPipeline -> Parameters**. Add new parameter. 
2.	Name it as **filename**, let the **Value** be empty.
3.	Click **Save**
4.	Navigate to **datasetBlobfromSQL -> Parameters -> File Name**, and set the value as **@pipeline().parameters.filename**
5.	Click on **Connection** tab to verify the parameters
6.	Navigate to **datasetBlob -> Parameters -> File Name**, and set the value as **@pipeline().parameters.filename**
7.	Click on **Connection** tab to verify the parameters
8.	Navigate to the **CopyPipeline**, do a Test Run. 
9.	It will ask for an input parameter. Enter appropriate name and this will be used as the file name in sink.
10.	Click on **Sync** (and **Publish**). This will write the changes to Master.
11.	Click on **CopyPipeline -> Triggers -> Add** new trigger.
12.	Enter the trigger properties accordingly. In this case, we create a daily tumbling window trigger.
Set Start, End time for **Trigger**. Check **Activated** check-box. 
13.	Click **Next**.
14.	In the **Trigger** Run Parameter window,Set **fileName -> copyfromsql_@{formatDateTime(trigger().outputs.windowStartTime, 'yyyy-MM-dd')}**

``
Note: Expressions can be changed based on requirements. 
``

15.	Click **Finish**.
Make sure you ‘Publish’ for the trigger to activated.

