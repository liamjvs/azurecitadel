---
title: "Management"
description: "Create a management baseline for the connected machines. Enable update management and inventory."
layout: single
draft: false
series:
 - arc-servers-hack-proctor
weight: 6
---

## Introduction

Azure Automation delivers a cloud-based automation and configuration service that supports consistent management across your Azure and non-Azure environments. It comprises process automation, configuration management, update management, shared capabilities, and heterogeneous features.

In the previous challenge, you onboarded Azure Arc virtual machines to Log Analytics. Azure Arc virtual machines can be managed using cloud based tooling such as Azure Automation and Azure Monitor, which will make use of the Log Analytics connected agents.

In this challenge you will implement a update management strategy and report inventory data.

## Update Management

* Create an Automation account

  In the portal, navigate to **Automation Accounts** and **Create**.
  
  * Name: `arc-automation`
  * Resource group: `arc-hack`
  * Create Azure Run As account: `Yes`

* Enable Update Management for the Azure Arc virtual machines

  Open the blade for the Automation Account and select **Update management**.
  
  Select the `arc-hack-workspace-team1` Log Analytics workspace created in the previous labs, and **Enable**.

* Schedule an update deployment for a weekly security update and full update monthly

  Reselect the Update management pane, **Schedule update deployment**.
  
  * Name: `arc-windows-security-weekly`
  * Operating system: `Windows`
  * Groups to update: `Select your Subscription or Resource Group scope`
  * Machines to update: `Leave blank`
  * Update classification: `Security updates`
  * Include/exclude updates: `Leave blank`
  * Schedule settings: `Recurring every 1 week`
  * Pre-scripts + Post scripts: `Leave blank`
  * Maintenance window (minutes): `120`
  * Reboot options: `Reboot if required`
  
  Repeat these steps for other update classifications and for Linux operating systems.

* Report update compliance

  From the Update management pane, check **Machines**, **Missing updates**, **Deployment schedules** and **History** tabs.

* Trigger an update deployment and measure its success

  Reselect the Update management pane, **Schedule update deployment**.
  
  * Name: `arc-windows-update-now`
  * Operating system: `Windows`
  * Groups to update: `Select your Subscription or Resource Group scope`
  * Machines to update: `Type: Machines`
  * Update classification: `Select all`
  * Include/exclude updates: `Leave blank`
  * Schedule settings: `Set in at least 5 minutes time, to occur once`
  * Pre-scripts + Post scripts: `Leave blank`
  * Maintenance window (minutes): `120`
  * Reboot options: `Reboot if required`
  
  From the Update management pane, check **History** tab to check progress of the job.

## Inventory

* Enable Inventory for the Azure Arc virtual machines

  Open the blade for the Automation Account and select **Inventory** from Configuration Management.
  
  Select the `arc-hack-workspace-team1` Log Analytics workspace created in the previous labs, and **Enable**.

* Review the software and services inventory

  Reselect the Inventory pane, check **Machines**, **Software**, **Windows Registry**, **Windows Services** and **Linux Daemons** tabs.

* (optional) Install additional software to an Azure Arc virtual machine, view the software in the Inventory

  Reselect the Inventory pane after software installation, and check **Machines**, **Software**, **Windows Registry**, **Windows Services** and **Linux Daemons** tabs. It may take 15-20 minutes after installation for the Software to be visible in the data.

## Log Analytics 

* (optional) Write a Log Analytics query to report the installed Windows Updates and the needed Windows Updates on Azure Arc connected Windows virtual machines

  In the portal, navigate to **Log Analytics workspace** and open `arc-hack-workspace-team1`.
  
  Open **Logs** from General.
  
  Software update data is stored in the `Update` table. Type `Update` into the query and **Run** to see all data.
  
  To show just the `Installed` or `Needed` updates, the following queries will work:
  
  ```Kusto
  Update
  | where UpdateState contains "Installed"
  ```
  
    ```Kusto
  Update
  | where UpdateState contains "Installed"
  ```
  
  Summarize can reduce the information displayed, such as using a `count` of `Computer` or `count` of `Title` (name of the update).
  
    ```Kusto
    Update
  | where UpdateState contains "Needed"
  | summarize count(Computer) by Title, UpdateState, Classification, Product, MSRCSeverity
    ```

* (optional) Write a Log Analytics query to report the Python version on Azure Arc connected Linux virtual machines


  Inventory data is stored in the `ConfigurationData` table. Type `ConfigurationData` into the query and **Run** to see all data.
  
  To show just the `Software`, the following queries will work:
  
  ```Kusto
  ConfigurationData
  | where ConfigDataType == "Software"
  ```
  
  This query can be refined to show the Python data with a query such as:

  ```kusto
  ConfigurationData
  | where ConfigDataType == "Software"
  | where SoftwareName contains "python"
  | summarize arg_max(TimeGenerated, *) by SoftwareName, CurrentVersion
  | render table
  | summarize by Publisher, SoftwareName, Computer
  ```

## Azure Monitor Workbooks

* Create an update assessment Workbook to visualize update compliance and detail missing updates

  An `Update Assessment.json` sample Workbook is available in the GitHub Azure Monitor Community here: [Github.com/microsoft/AzureMonitorCommunity/Azure Services/Azure Monitor/Workbooks](https://github.com/microsoft/AzureMonitorCommunity/blob/master/Azure%20Services/Azure%20Monitor/Workbooks/Update%20Assessment.json)
  
  In the portal, navigate to **Monitor**.
  
  In **Workbooks**, choose **New** and **`</>` (Advanced Editor icon)**.
  
  Paste the contents of the `Update Assessment.json` template into the editor (replace any existing content within the input field). Check the **Template Type** is set to `Gallery Template`.
  
  **Apply** the changes, select **Workspace:** `arc-hack-workspace-team1` and **Servers:** `All`, then **Done Editing**.
  
  A preview of the Update Assessment Workbook will display and should be populated with compliance data. If not, check the **Workspace** and **Servers** scopes.
  
  Click the **Save** icon to save the Workbook, and consider whether to use the **Pin** icons next to each chart to save the charts to the Dashboard.

## Success criteria

Screen share with your proctor to show that you achieved:

1. Update Management solution is enabled
1. Deployment schedules are in place for both security and full updates
1. Report on the current update compliance state for all Azure Arc virtual machines
1. Show the update history
1. Inventory solution is enabled
1. Software, Windows Services and Linux Daemons display in the inventory
1. Show update compliance with an Azure Monitor Workbook