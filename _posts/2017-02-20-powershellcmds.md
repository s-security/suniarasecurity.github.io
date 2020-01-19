---
layout: post
title: Useful Powershell Commands
date: 2017-09-12
image: '/images/posts/program1.jpg'
categories: ["sysadmin"]
tags: ["sysadmin"]
---

Below are some useful PowerShell commands that are almost all applicable to a system administrator.

<!--more-->

**Back up all production Group Policy Objects**
```
Backup-GPO –All –Path C:\Temp\AllGPO
```

**Check a KB is installed on a Windows machine**
```
Get-HotFix –ID update KB2670838
```
You can also query hotfix information on a remote computer using the below command:

```
Get-HotFix –ID KB2670838 –Computername TestVM.suniarasecurity.com
```


**See who rebooted a server**
This command is excellent when dealing with unexpected reboots.
It will check the system event log for the Event ID 1074 and print the machine name, username, and time that event got generated.
You can add the "Export-CSV" cmdlet to save the output to a CSV file.
```
Get-EventLog –Log System –Newest 100 | Where-Object {$_.EventID –eq ‘1074’} | FT MachineName, UserName, TimeGenerated -AutoSize
```

**Gathering disabled computer accounts from AD**
This command will export all Active directory computer accounts that are **disabled** and print the output. Again, you can add the "Export-CSV" cmdlet to save the output to a file.
```
Get-ADComputer -Filter {(enabled -eq $false)} -ResultPageSize 2000 -resultSetSize $null -Properties Name,OperatingSystem,SamAccountName,DistinguishedName
```

**List All Services**
```
Get-Service [optional wildcard search on service name (not display name)] | `
sort [Status, Name or Displayname]
```
This will list all running services. This can sometimes come in handy for a sysadmin but is more useful during pentesting enumeration stages. The below command will list all running services.

```
Get-Service [[optional wildcard search on service name (not display name)] | `
where Status -eq running | sort [Status, Name or Displayname]
```
