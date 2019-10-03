# ETL with Azure Databricks – Setup Instructions
 
## Overview
In this lab, you will create an end-to-end pipeline containing Lookup, Copy and Databricks notebook activities in ADF.<br/>
•	Lookup or GetMetadata activity is used to ensure the source dataset is ready for downstream consumption, before triggering the copy and analytics job.<br/>
•	Copy activity, copies the source file/ dataset to the sink storage. The sink storage is mounted as DBFS in the databricks notebook so that the dataset can be directly consumed by Spark.<br/>
•	Databricks notebook activity, triggers the databricks notebook that transforms the dataset, and adds it to a processed folder/ SQL DW.<br/>
We will not create a scheduled trigger in this template for simplicity, and you may add that if required.
   <img src="images/adf1.jpg"/><br/>
   
### Prerequisite:
1. Create a blob storage account and a container called **sinkdata** to be used as sink. Keep a note of the storage account name, container name and Access key will be referenced later in the template.<br/>
   <img src="images/adf5.jpg"/><br/>
2. Go to **Storage Explorer (preview)** -> **BLOB CONTAINERS** -> **sinkdata** -> **+ New Folder** and give instructions as below:<br/>
    a. Name: **staged_sink**<br/>
    Click Ok<br/>
   <img src="images/adf6.jpg"/><br/>    
3. Now, Upload **Product.csv** file in that **staged_sink** folder.<br/>
   <img src="images/adf7.jpg"/><br/>
4. Ensure you have an **Azure Databricks workspace** or create a new one.<br/>

a. Import the below **Transform** notebook to the databricks workspace. (need not be the same location as below, just remember the path for later usage)<br/>
```
ACTION REQUIRED – Import the notebook for ETL. Import the notebook from URL. Please enter the below URL in the URL field: **https://adflabstaging1.blob.core.windows.net/share/Transformations.html**. Select Import.<br/>
```
   <img src="images/adf2.jpg"/><br/>
b. Now let’s update the **Transformation** notebook with your storage connection information (name and access key). Go to command 5 in the imported notebook above, replace it with the below code snippet after replacing the highlighted values. Ensure this is the same storage account created earlier and contains the sinkdata container.<br/>
   <img src="images/adf3.jpg"/><br/>
c. Generate a databricks access token for **ADF** to access databricks. Save the access token for later use in creating a databricks linked service, which looks something like **'dapi32db32cbb4w6eee18b7d87e45exxxxxx'**
   <img src="images/adf4.jpg"/><br/><br/>
   <img src="images/adf9.jpg"/><br/>
  
### Create Linked Services and datasets
1.	Create new **linked services** in ADF UI by going to **Connections** -> **Linked services** -> **+ new**<br/>

a.	**Source** – for accessing source data. You can use the public blob storage containing the source files for this sample.<br/>
Select **Blob Storage**, use the below **SAS URI** to connect to source storage (read-only access).<br/>
```
https://storagewithdata.blob.core.windows.net/?sv=2017-11-09&ss=b&srt=sco&sp=rl&se=2019-12-31T21:40:53Z&st=2018-10-24T13:40:53Z&spr=https&sig=K8nRio7c4xMLnUV0wWVAmqr5H4P3JDwBaG9HCevI7kU%3D<br/>
```
   <img src="images/adf10.jpg"/><br/>
b.	**Sink** – for copying data into.<br/>
Create a new linked service. Select a storage created in the **prerequisite 1**, in the sink linked service.<br/>
   <img src="images/adf11.jpg"/><br/>
c.	**Databricks** – for connecting to databricks cluster<br/>
Create a **databricks** linked service using **access key generated** in **prerequisite 4.c**. If you have an interactive cluster, you may select that. (in this case we use a New job cluster option)<br/>
   <img src="images/adf12.jpg"/><br/>
   <img src="images/adf13.jpg"/><br/>  
2.	Create **datasets**<br/>

a. Add **Dataset** as shown in image:
   <img src="images/adf14.jpg"/><br/>
b.	Create **'sourceAvailability_Dataset'** to check if source data is available.<br/>
   i. Select a data store as **Azure Blob Storage**
   <img src="images/adf15.jpg"/><br/>
   ii. Select the fromat type as **DelimitedText** of your data.
   <img src="images/adf16.jpg"/><br/>
   iii. Now, create as **'sourceAvailability_Dataset'** and follow instructions as below:<br/>
    **Linked service** - select **sourceBlob_LS**<br/>
    **File path** - **data/source/_success**<br/>
   <img src="images/adf17.jpg"/><br/><br/>  
c.	Source **dataset** – for copying the source data (using binary copy)<br/>
   i. Select a data store as **Azure Blob Storage**<br/>
   <img src="images/adf15.jpg"/><br/>
   ii. Select the fromat type as **Binary** of your data.<br/>
   <img src="images/adf19.jpg"/><br/>
   iii. Now, create as **'sourceFiles_Dataset'** and follow instructions as below:<br/>
    **Linked service** - select **sourceBlob_LS**<br/>
    **File path** - **data/source/Product.csv**<br/>
    <img src="images/adf20.jpg"/><br/><br/>
d.	**Sink dataset** – for copying into the sink destination location<br/>
   i. Select a data store as **Azure Blob Storage**<br/>
   <img src="images/adf15.jpg"/><br/>
   ii. Select the fromat type as **Binary** of your data.<br/>
   <img src="images/adf19.jpg"/><br/>
   iii. Now, create as **'sinkRawFiles_Dataset'** and follow instructions as below:<br/>
   **Linked service** - select **sinkBlob_LS**<br/>
   **File path** - **sinkdata/staged_sink**<br/>
   <img src="images/adf18.jpg"/><br/><br/>
 
### Create activities
a. Create a Lookup activity **Availability flag** for doing a Source Availability check (Lookup or GetMetadata can be used). Select **sourceAvailability_Dataset** created in 2.a.<br/>
   <img src="images/adf21.jpg"/><br/>
b.	Create a **Copy** activity **file-to-blob** for copying dataset from source to sink. In this case the data is binary file. Reference the below screenshots for **source** and **sink** configurations in the copy activity.<br/>
   <img src="images/adf22.jpg"/><br/>
   <img src="images/adf23.jpg"/><br/>
c.	Define **pipeline parameters**<br/>
   <img src="images/adf24.jpg"/><br/>
d.	Create a databricks **activity**<br/>
•	Select the **linked** service created in 1.c<br/>
   <img src="images/adf25.jpg"/><br/>
•	Configure the settings. Create **Base Parameters** as shown in the screenshot and create parameters to be passed to the databricks notebook from **ADF****. Browse and select the correct notebook path uploaded in prerequisite 2.<br/>
  <img src="images/adf26.jpg"/><br/>
•	Run the **pipeline**. You can find link to databricks logs for more detailed spark logs.<br/>
  <img src="images/adf27.jpg"/><br/>
•	You can also verify the data file using **storage explorer**. (For correlating with Data Factory pipeline runs, we appended pipeline run id from data factory to the output folder. This way you can track back the files generated via each run)<br/>
  <img src="images/adf28.jpg"/><br/>
