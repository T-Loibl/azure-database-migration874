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
- Once complete, select prompt to install SSMS
- Install

###Task 4: Create Production Database
- Download the AdventureWorks2022.bak file
- Move to `C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup\`
- Open SSMS and connect to the SQL Server instance insatlled previously
- Right-click 'Database' option and select 'Restore Database'
- In subsequent dialog window, select 'Device' as source, locate backup file
  - located at `C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Backup\`
- CLick 'OK', allow restore process, review and 'OK' 
