# Ingesting Data into a Aazure Data Lake Storage Gen2 with Azure Data Factory

## Table of Contents

Summary
Pre-requisite
Scenario
Part 1 – Create an Azure Data Factory (v2)
Part 2 – Connect ADF to a code repository to begin using the ADF GUI
Part 3 – Setting up the Connections in the ADF GUI (Azure SQL Database -> Blob)
Part 4 – Setting up the Copy Activity in the ADF GUI (Azure SQL Database -> Blob)
Part 5 – Using Parameters and Triggers (scheduling) in ADF GUI


 
### Summary
This tutorial walks through creating a pipeline copy activity to copy a file from a S3 storage location to an Azure Data Lake Storage Gen2 container, so we can prepare the file to be processed later for transformation.

In this lab we will:
•	Show the graphical user interface for creating a pipeline
•	Copy CSV file via a Copy Activity
•	Copy Structed data from SQL Azure via a Copy Activity
•	Use parameters to make the pipeline easy to change and more reusable

 
 
### Prerequisites
•	Azure Subscription with rights to use/deploy Azure services, and X of Azure credit
•	An ADLS Gen2 storage account with a filesystem (container)
•	Azure Data Factory
•	Visual Studio Team Services Git project (optional)

### Part 1 – Create an Azure Data Factory (v2)
We are going to use the portal to create the Azure Data Factory.

1.	Go to Azure Portal- https://portal.azure.com.
2.	Select New on the left menu, select Data + Analytics, and then select Data Factory.
3.	On the New data factory page, enter adflab-adf for Name.
4.	Make sure you select Version as ‘V2’
5.	Click ‘Create.

``
Note: The name of the Azure data factory must be globally unique. Please modify the name if the Name validation fails.
``
6.	After the creation is complete, you see the Data Factory page. Select the Author & Monitor tile to start the Azure Data Factory user interface (UI) application on a separate tab.

### Part 2 – Connect ADF to a code repository to begin using the ADF GUI (Optional)
One option to be able to sync our code is to connect ADF to a code repository. This section walks through the steps to connect ADF to a Visual Studio Team Services Git project, so we can save our code for later re-use. Note that this is not required but a recommended best practice.

1.	Navigate to the Azure portal within your web browser and navigate to https://portal.azure.com
2.	Open the Azure Data Factory blade adflab-adf.
3.	In the Overview Blade, Click on ‘Author and Monitor’
4.	Click the Configure Code Repository button to begin connecting this Azure data factory to a code repository.
5.	You can either create a new VSTS account for this lab or use an existing one. You can create a new one - here and configure it in ADF GUI under repository settings. You need to create/ select a Project under this Account.
6.	The Repository Settings pane will appear on the right.
7.	Click the Finished button when you have verified your settings.

### Part 3 – Setting up the Connections in the ADF GUI (Azure SQL Database -> ADLS Gen2)
We now want to use the GUI to create another copy activity in the same pipeline to copy the Data from Azure SQL DB to Azure Data Lake Storage Gen2 to be ready for transformation along with the earlier CSV file. Our first step is setting up the connections and linked services need for the source and destination.

21.	In the Left Menu click the Connections menu item.
22.	Click the +New button under Linked Services.
23.	In the right pane you should now see the list of possible Linked Services. 
24.	Click on the Azure SQL Database tile.
25.	Click Continue.
26.	In the right pane you should see the properties to configure the Azure SQL Database account. We will name this linked service AzureSqlDatabase_Source and using the Default runtime. Use the following  

 * Account Selection Method -> Manual,
 * Fully qualified domain name ->adlabserver.database.windows.net,
 * Database name -> adflab,
 * User name -> lab_user,
 * Password -> P@ssw0rd
 
27.	Click the Test Connection to verify settings are entered correctly.
28.	Click Save.

### Part 4 – Setting up the Copy Activity in the ADF GUI (Azure SQL Database -> ADLS Gen2)
We now want to use the GUI to create a Copy Activity in the pipeline to move the files from the Azure SQL Database as source to our Azure Data Lake Storage gen2 destination.

29.	Click the CopyPipeline in the left menu to return to the pipeline GUI.
30.	In the Pipeline GUI, drag the Copy activity (under DataFlow) to the empty pane above General.
31.	Rename the activity to AzureSQLtoADLSGen2.
32.	Click Save.
33.	Click the Source Tab in the Copy Activity GUI.
34.	Click the +New button next to Source Dataset.
35.	You should now see the list of source dataset connectors.
36.	Choose the Azure SQL dataset and click Finish.
We will be using the Linked Service we created earlier.  	37.	You should now add the connection property information.

 *	Name this datasetAzureSqlTable 
 *	Select Table -> [SalesLT].[Customer]
 
40.	Click Preview Data to preview the first several data rows.	 
We will only copy the required columns from source to sink, and refrain from copying sensitive columns by using a query. 	41.	Click on CopyPipeline and then click AzureSQLtoADLSGen2 Activity.
42.	Select ‘Source’ -> Use Query -> Query

```
SELECT CustomerID, CompanyName, SalesPerson, ModifiedDate 
FROM [SalesLT].[Customer]
```
Note: This Query may change based on your table selection. 

43.	Click on Preview data to ensure the query works.
44.	Click back on the CopyPipeline.
45.	Click the AzureSQLtoADLSGen2 copy activity.
46.	Click the Sink Tab in the Copy Activity GUI.
47.	Click the +New button next to Source Dataset.
48.	You should now see the list of sink dataset connectors.
49.	Choose the Azure Data lake Storage Gen2 dataset and click Finish.
50.	Name the dataset as datasetADLSgen2fromSQL 
51.	Fill out the following information: Linked Service -> gen2connection, File Path -> Click the Browse button and drill down to the inputsql container.
```
Note: Make sure inputsql container exists or else create one first. 
```
52.	Save the dataset
53.	Navigate to the CopyPipeline
54.	Click the Debug icon at the top menu to test and run our copy activity.
55.	Final CopyPipeline looks like this

### Part 5 – Using Parameters and Triggers (scheduling) in ADF GUI
You can define parameters at the pipeline level and pass arguments while you're invoking the pipeline on-demand or from a trigger. Activities can consume the arguments that are passed to the pipeline. Using parameters, you can build more flexible pipelines. 
And triggers can be used to execute the pipelines on a schedule or on-demand.

56.	Navigate to CopyPipeline -> Parameters. Add new parameter. 
57.	Name it as filename, let the Value be empty.
58.	Click Save
59.	Navigate to datasetADLSgen2fromSQL -> Parameters -> File Name, and set the value as @pipeline().parameters.filename
60.	Click on Connection tab to verify the parameters
61.	Navigate to datasetADLSGen2 -> Parameters -> File Name, and set the value as @pipeline().parameters.filename
62.	Click on Connection tab to verify the parameters	 
63.	Navigate to the CopyPipeline, do a Debug. 
64.	It will ask for an input parameter. Enter appropriate name and this will be used as the file name in sink. 	
Publish Code Repository (OPTIONAL) only if you had configured it in Step 2.	65.	Click on Publish. This will write the changes to Master.	
We can configure trigger for operationalizing pipelines. 	66.	Click on CopyPipeline -> Triggers -> Add new trigger.
67.	Enter the trigger properties accordingly. In this case, we create a daily tumbling window trigger.
Set Start, End time for Trigger. Check Activated check-box. 
68.	Click Next. Setting system variables to create partitions in the sink during operationalized copy pipeline runs.
69.	In the Trigger Run Parameter window,
Set fileName -> copyfromsql_@{formatDateTime(trigger().outputs.windowStartTime, 'yyyy-MM-dd')}
```
Note: Expressions can be changed based on requirements. 
```
70.	Click Finish.
Make sure you ‘Publish’ for the trigger to activated.	 

71.	Navigate to Monitoring section to see pipeline runs.
72.	We can find the appropriate parameters being passed during each triggered run. 	 
73.	On Successful run of the CopyPipeline, navigate to the storage locations using Storage Explorer or Azure Portal (Storage Account), to verify the files copied. The filename would be defined by the parameter -> fileName.
