---
layout: post
title: "Start/Stop Azure VMs during off-hours — The Logic App Solution"
date: 2019-12-28 00:00:00 +1000
categories: Azure
---

# Start/Stop Azure VMs during off-hours — The Logic App Solution

Virtual machines in a Cloud are most of their time not productively and waiting for some jobs to do. Especially during off-hours when the offices are closed and the people are sleeping at their homes.

Azure Virtual Machines have the capability to Auto-shutdown at a specific time of the day. This can be configured in the Azure Portal but there is no easy way to configure it to Auto-startup at a specific time of the day.

There is a very large and complex solution available from Microsoft to schedule starting and stopping Azure Virtual Machines based on a Azure Automation Account. Very nice solution but for my problem too complex and too much configuration to get it working.

## Azure Logic App Solution
The Logic App is using the Recurrence trigger to start every day at (for example) 6:00 o’ clock.

### Logic App Recurrence Trigger

There is a command to [start an Azure Virtual Machine](https://docs.microsoft.com/en-us/rest/api/compute/virtualmachines/start) using the Azure REST API. This command can be triggered by a Logic App using the HTTP Action. The command to start a Virtual Machine is the following:

```
https://management.azure.com/subscriptions/{{guid-of-the-subscription}}/resourceGroups/{{name-of-the-resource-group}}/providers/Microsoft.Compute/virtualMachines/{{name-of-the-vm}}/start?api-version=2019-03-01
```

Not everybody is allowed to start or stop a Virtual Machine in Azure. We can set Authentication to [Managed Identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview).


Enable Managed Identity on the Logic App
Giving the Logic App the build-in role [Virtual Machine Contributor](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#virtual-machine-contributor) allows the Logic App to start/stop the Virtual Machine.


Give the Logic App the role Virtual Machine Contributor
Using the Auto-shutdown functionality of the Virtual Machine in the Azure Portal to stop the Virtual Machine at the end of the day and the Logic App to start the Virtual Machine in the morning we reduce the cost of this Virtual Machine.