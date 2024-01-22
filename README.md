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





