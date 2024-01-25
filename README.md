#Azure Database Migration Project 

##Overview

This README provides detailed instructions for setting up a Windows Virtual Machine (VM) in Azure to serve as the cornerstone of your production environment. 
The VM will emulate the functions of a Windows server, replicating the operations of an on-premise system within a company. Throughout the project, 
the VM will act as a repository for the company's database, simulating a secure and dedicated data storage solution.

##Setting up the Production Environment

###Task 1: VM Provisioning
- Navigate to the Azure Portal and select "Virtual machines" from the left-hand menu.
- Click on "Create" to create a new VM.
- Choose the appropriate settings for your VM, including OS, size, and authentication.
  - In this instance the only settings changed were: Name - (my-vm-name) and OS - Windows Pro 11
  - The rest were left as default
- Follow the on-screen instructions to provision the VM.
  - Set Password-based authenification
  - Inbound Port Rules set to 'allow selected ports' and 'RDP (3389)
- Review and Create

###Task 2: Connect Windows VM
- Install MIcrosoft Remote Desktop
- Download RDP file from Azure
- Open RDP in MIcrosoft Remote Desktop
- Allow VM to initialise

###Task 3: Install SQL Server and SSMS (SQL Server Management Studio)
- Once connected to the VM, download and install Microsoft SQL Server Developer version on the VM using the basic option
- Launch the Installation Wizard
- Once complete, select prompt to install SSMS and Azure Data Studio
- Install

###Task 4: Create Production Database
- Download the AdventureWorks2022.bak file
- Move to `C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup\`
- Open SSMS and connect to the SQL Server instance installed previously
- Right-click 'Database' option and select 'Restore Database'
- In subsequent dialog window, select 'Device' as source, locate backup file
  - located at `C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup\`
- CLick 'OK', allow restore process, review and 'OK'


##Migrating local data to Azure SQL Database

###Task 1: Create target database
- Create Azure SQL Database inside previously created Resource Group
- Follow prompts, including the creation of Server for the database within the Resource group
  - Ensure 'SQL Server Authenification' selected for security method
  - Configure 'Compute and Storage', select appropriate storage tier ('basic' in this instance)
- Once created alter firewall rules to allow the IP address of the previously created VM

###Task 2: Migration Preparation 
- Open Azure Data Studio on VM
- In 'Extensions' menu search for and install the necissary extensions:
  - 'SQL Server Schema Compare' and 'Azure SQL Migration'
- Connect Azure Data Studio to both the locally hosted 'AdventureWorks2022' SQL Server and the empty Azure SQL Database hosted on the server created in the previous step

###Task 3: Schema Migration 
- Right click on the source database ('AdventureWorks2022') in Azure Data Studio
- Select 'Schema Compare' option
- In the following tab, ensure that the source is 'AdventureWorks2022' and the target is the Azure hosted SQL database made previously
- Click 'compare' button at top of screen
- Aplly all changes to the target database
- Wait for success dialogue to confirm completion

###Task 4: Data Migration 
- Select local database in Azure Data Studio
- Press 'Azure Data Migration' then 'Migrate to Azure SQL Database'
- Follow steps as laid out in Migration Wizard
  - Select source and target database
  - Confirm target database type, location, etc
  - Enter login credentials
- Create an Azure Database Migration Service
  - Leave default configurations
- Download and install 'Integration Time' as prompted by wizard
  - Once installed will ask for one of the two keys provided by the wizard
  - Click 'register'
  - Click 'Launch Configuration Manager'
  - Return to Azure Data Studio
- Follow Instructions in Migration Wizard
  - Provide login credentials
  - Select all tables to be migrated then click 'update'
  - 'Run Validation' to ensure compatibility and identify potential issues
  - Press 'Start Migration'
- Monitor progress through the Azure Database Migration Service in Azure webpage or in migrations panel of Azure SQL Migration extension in Azure Data Studio

###Task 5: Validate 
- Once migration is successful validate this manually
- Check that the correct number of tables are present
- Randomly go through tables, displaying the top 1000 entries and compare to source database for discrepencies


##Data Backup and Restore

###Task 1: Backup onpremise database
- Find the onpremises database in SSMS
- Right-click, press 'Tasks' then 'Backup'
- Follow subsequent instuctions
  - Choose destination to save backup file
  - Select desired backup type ('Full' in this instance)
  - Click 'OK' to initiate

###Task 2: Upload backup to Blob storage
- Open Azure portal and navigate to Azure storage accounts
- CLick on 'Upload' then drag and drop the backup file
- Set the container to which it will be saved
- Wait until upload is completed

###Task 3: Restore Database in Development Environemnt 
- Create a new VM with the same environment as the previously made one (Follow Task 1 of Setting up the Production Environment)
  - The development Environment is a controlled and isolated area where the database and system can be worked upon and stress-tested without altering the production environment
- Connect to the new Development VM
- Download requisite programs
  - SQL Server, SSMS and Azure Data Studio
- Downoad the backup file stored in the Azure Blob
- Move it to the backup folder located at `C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup\`
- Restore the database
  - Follow steps described in Task 4 of Setting up the Production Environment
 
###Task 4: Automate Backups 
- Create Server Credentials
  - In SSMS right click server name, 'New Query'
  - Execute following T-SQL command
    - CREATE CREDENTIAL [YourCredentialName]
    
      WITH IDENTITY = '[Your Azure Storage Account Name]',

      SECRET = 'Access Key';
      - Replace parts in [] with own info
      - 'Access Key' can be found in the 'Access Keys' tab under 'Security + Networking' in the associated Azure Storage account
- Create management task
  - Open 'Management' node, right click 'Maintenenance Plans', select 'Maintenance Plan Wizard'
  - Follow wizard instructions, setting the back up schedule to weekly and the type to FUll
  - Select database to be backed-up and select location  


##Disaster Recovery Simulation

###Task 1: Mimic Data Loss in Production Environment 
- Connect to the Production VM
- Ensure Azure Data Studio is connected to production database
- Right click production database, click 'New Query'
- Write T-SQL query that will simulate deletion and corruption:
  - Deletion
    - Delete top 100 entries from 'Person.Person'
  - COrruption
    - In 'Purchasing.Vendor' change the top 100 results for the 'BusinessEntityID' column to NULL
- Verify intentional data loss by querying tables in question, which should return 100 fewer results in 'Person.Person' while the top 100 entries in 'BusinessEntityID' for 'Purchasing.Vendor' should be NULL values

###Task 2: Restore Database 
- Find and select the affected database in the Azure SQL Database dashboard through the Azure portal
- Select the 'Restore' option
- In the subsequent window:
  - Select the restore point
    - This will be a point in time before the Data Loss/Corruption happened
- Once completed check the database to ensure the data loss and corruption have been corrected
  - In this case the top 100 entries for 'Person.Person' should have returned and the top 100 entries in 'BusinessEntityID' for 'Purchasing.Vendor' should no longer be NULL values
- Once restored delete the corrupted database as it is no longer needed


##Geo-Replication and Failover

###Task 1: Setup Geo-Replication
- Find and select the target database in the Azure portal
- In the side menu find the 'Replicas' tab under 'Data management'
- Click 'create new' at the top of the directory
- In the subsequent menu:
  - Create a new server, which must be located in a geographically distinct region from the primary database server
  - In this case the primary server is in UK South, and the Geo-Replica is in East US
  - Review and Create
 
###Task 2: Failover and Failback
- Create Failover group
  - Find SQL Server for primary database
  - Under 'Data management' tab select 'Failover groups' then 'Add group'
  - On next page:
    - Enter name for group
    - Select the secondary server for the 'server' option
    - Create
- Select 'failover group' option within either primary or secondary server
- Select the failover group
- Click the 'Failover' option in top menu bar to initiate a planned failover from the primary to the secondary server



##Microsoft Entra Directory Integration

###Task 1: Configure Entra ID 
- Find SQL server that hosts primaru database in Azure portal
- Find 'security' tab, click 'Microsoft Entra'
- Click 'Set admin', select 'user', select your user
  - This will grant privilaged permissions to that user within the environment
- Remember to click 'save' on top menu to save changes

###Task 2: Create DB Reader User
- Go to Microsoft Entra ID page via the Azure portal
- Create new users by going to 'Add' > 'users' > 'create new user'
- Fill in the details
  - Select descriptive names, in this instance creating a DB Reader user, so something along lines of db_reader_{name}
  - Create password, can use the auto-generated one
- Connect to primary database in Azure Data Studio using the Microsoft Entra ID
- Open query to assign the db_datareader role to the user just created
  - This will provide read_only privilages to the user
- USe the following SQL query syntax
  - CREATE USER [db_reader_name@domainname.com] FROM EXTERNAL PROVIDER;

    ALTER ROLE db_datareader ADD MEMBER [db_reader_name@domainname.com]
- Reconnect to primary database and attempt to make changes
  - Should be unable to edit database due to altered privilages
