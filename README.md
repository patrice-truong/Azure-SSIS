# Azure SSIS

This repository contains a project for executing an Azure Data Factory pipeline that moves data from an on-premises SQL Server to an Azure SQL database in the cloud.

The SSIS package used to move data to the cloud is stored in an SSISDB catalog.


![Demo](assets/Azure-SSIS.gif)

## Azure Architecture


![Architecture Diagram](assets/architecture.png)

## References

- Quickstart: Create a Windows virtual machine in the Azure portal: https://learn.microsoft.com/en-us/azure/virtual-machines/windows/quick-create-portal
- Quickstart: Create a data factory by using the Azure portal: https://learn.microsoft.com/en-us/azure/data-factory/quickstart-create-data-factory
- Quickstart: Use the Azure portal to create a virtual network: https://learn.microsoft.com/en-us/azure/virtual-network/quick-create-portal
- Create and configure a self-hosted integration runtime: https://learn.microsoft.com/en-us/azure/data-factory/create-self-hosted-integration-runtime?tabs=data-factory

## Step-by-step walkthrough

### Installation

**Windows Server 2019 with SQL Server 2022 virtual machine**

Create a Windows Server 2019 with SQL Server 2022 virtual machine with a preset configuration.

Refer to the documentation for more details.

**Azure Data Factory**

Create a new Azure Data Factory instance in your Azure subscription


We will be accepting all the default options. Click on "Review+Create"

![adf-defaults](assets/adf-defaults.png)

**Azure blob storage**

In this section, you will:
- create a Blob Storage (without namespace) for SSIS integration runtime
- give "Storage blob data contributor" permissions to ADF managed identity

![storage-blob-data-contributor](assets/storage-blob-data-contributor.png)

![storage-df](assets/storage-df.png)

**Self hosted integration runtime**

In Azure Data Factory Studio, create a Self-Hosted Integration runtime

![azure-self-hosted-1](assets/azure-self-hosted-1.png)

![azure-self-hosted-2](assets/azure-self-hosted-2.png)

![azure-self-hosted-3](assets/azure-self-hosted-3.png)

![azure-self-hosted-4](assets/azure-self-hosted-4.png)

Copy Key1 to your clipboard

**Install software pre-requisites in virtual machine**

1. Log into your virtual machine using Remote access
2. Install Visual Studio 2022 Community from https://learn.microsoft.com/en-us/visualstudio/install/install-visual-studio?view=vs-2022
4. Install the Microsoft Integration Runtime from https://www.microsoft.com/en-us/download/details.aspx?id=39717. Start the IR at the end of the installation process
5. Paste the key from your ADF Integration runtime and click Register

![ir-register](assets/ir-register.png)

![ir-register-finish](assets/ir-register-finish.png)

![ir-success](assets/ir-success.png)


6. Click on Launch Configuration Manager

![ir-configuration-manager](assets/ir-configuration-manager.png)

Your Integration Runtime in Azure Data Factory should now be running

![ir-running](assets/ir-running.png)

7. Enable SSIS package execution by running this command in a prompt (in your on-prem server):

```sh
"\Program Files\Microsoft Integration Runtime\5.0\Shared\dmgcmd.exe" -EnableExecuteSsisPackage
```
![enable-package-execution](assets/enable-package-execution.png)

**Create a link service to your on-prem SQL Server**

In Azure Data Factory Studio, create a new linked service to your source SQL Server

![link-sql-server](assets/link-sql-server.png)

![link-sql-server-2](assets/link-sql-server-2.png)

**Create a new linked service to your Azure blob storage**

![link-blob-storage-1](assets/link-blob-storage-1.png)

![link-blob-storage](assets/link-blob-storage.png)

Publish All if necessary

![publish-all](assets/publish-all.png)

**Create a virtual subnet for SSIS with batch delegation**

- create a virtual network
- in this vnet, create a subnet for SSIS
- add batchAccounts delegation to this subnet

![vnet-ssis-subnet](assets/vnet-ssis-subnet.png)

- create a linked service to this blob storage

**Create an Azure-SSIS IR**

If you have an SSISDB database already present in your target Azure SQL 
In ADF Studio, delete it.
Creating an Azure-SSIS Integration runtime as a proxy will create the SSISDB database for you.


![azure-ssis-1](assets/azure-ssis-1.png)

![azure-ssis-2](assets/azure-ssis-2.png)

![azure-ssis-3](assets/azure-ssis-3.png)

- Check the box "Select a vnet for your Azure-SSIS integration runtime"
- Select your vnet, subnet, injection method (express)
- Check the box "Setup SHIR as a proxy"
- Select your storage linked service
- Click on the "Vnet validation" button

![ir-validation](assets/ir-validation.png)

Click on "Continue" then on "Create"

![ir-create](assets/ir-create.png)

The Azure-SSIS runtime will start right after the creation process is complete. Give it a few minutes and check the IR page in ADF studio

![azure-ir-running](assets/azure-ir-running.png)

There should now be a new SSISDB database in your Azure SQL Server

![ssisdb](assets/ssisdb.png)

**Create SSIS package**

In your VM, create a simple SSIS package that moves data from the local sql server to the Azure SQL database in the cloud.

In SSIS package, select the OLEDB connection that points to the ON-prem sql server and change the ConnectByProxy property to true

![proxy-true](assets/proxy-true.png)

Execute the package locally to make sure it can copy data to the target Azure SQL database

![ssis-package-running-locally](assets/ssis-package-running-locally.png)

**Publish to SSISDB**

- Right click on your Visual Studio project
- Select Deploy

![ssis-ssis-deploy-1](assets/ssis-deploy-1.png)

![ssis-ssis-deploy-2](assets/ssis-deploy-2.png)

Make sure SSIS in SQL Server is selected, click Next

![ssis-ssis-deploy-3](assets/ssis-deploy-3.png)

![ssis-ssis-deploy-4](assets/ssis-deploy-4.png)

Click on Browse

![ssis-ssis-deploy-5](assets/ssis-deploy-5.png)

Create a new folder "ProductProject" and click OK

![ssis-ssis-deploy-6](assets/ssis-deploy-6.png)

Click on Next, then Deploy

![ssis-ssis-deploy-7](assets/ssis-deploy-7.png)

When the wizard completes, click the close button

![ssis-ssis-deploy-8](assets/ssis-deploy-8.png)

Verify that your SSIS project (LoadAdventureWorks) is properly deployed on the target Azure SQL database

![ssis-ssis-deploy-9](assets/ssis-deploy-9.png)

**Execute SSIS package from SSISDB**

- In your Integration Services Catalog, right-click on the LoadAdventureWorks project and select Execute package

![ssisdb-execute](assets/ssisdb-execute.png)

![ssisdb-execute-2](assets/ssisdb-execute-2.png)

![ssisdb-execute-3](assets/ssisdb-execute-3.png)

**Execute SSIS package from ADF**

- Create an ADF pipeline

![adf-pipeline-1](assets/adf-pipeline-1.png)

![adf-pipeline-2](assets/adf-pipeline-2.png)

- Publish your pipeline
- Execute your pipeline 

![adf-pipeline-trigger](assets/adf-pipeline-trigger.png)

- Click on Monitor to monitor execution progress

![adf-pipeline-in-progress](assets/adf-pipeline-in-progress.png)

- After a few seconds, the pipeline executes completes

![adf-pipeline-success](assets/adf-pipeline-success.png)

- The query in the Azure SQL database returns all the products !

![sqldb-products](assets/sqldb-products.png)
