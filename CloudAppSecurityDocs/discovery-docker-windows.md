---
# required metadata

title: Configure automatic log upload for continuous reports using a Docker on Windows | Microsoft Docs
description: This topic describes the process configuring automatic log upload for continuous reports in Cloud App Security using a Docker in Windows.
keywords:
author: rkarlin
ms.author: rkarlin
manager: mbaldwin
ms.date: 7/16/2017
ms.topic: get-started-article
ms.prod:
ms.service: cloud-app-security
ms.technology:
ms.assetid: 308c06b3-f58b-4a21-86f6-8f87823a893a


# optional metadata

#ROBOTS:
#audience:
#ms.devlang:
ms.reviewer: reutam
ms.suite: ems
#ms.tgt_pltfrm:
#ms.custom:

---

# Set up and configure the automatic Log Collector Docker on Windows Server 2016

## Technical requirements

-   OS: Windows sever 2016 or Windows 10

-   Disk space: 250 GB

-   CPU: 2

-   RAM: 4 GB

-   Firewall settings:

    -   Allow the log collector to receive inbound FTP and Syslog traffic.

    -   Allow the log collector to initiate outbound traffic to the portal (for example contoso.cloudappsecurity.com) on port 443.

## Log collector performance

The Log collector can successfully handle log capacity of up to 50 GB per hour. The main bottlenecks in the log collection process are:

-   Network bandwidth - your network bandwidth determines the log upload speed.

-   I/O performance of the virtual machine allocated by your IT - determines the speed at which logs are written to the log collector’s disk. The log collector has a built-in safety mechanism that monitors the rate at which logs arrive and compares it to the upload rate. In cases of congestion, the log collector starts to drop log files. If your setup generally exceeds 50 GB per hour, it is recommended to split the traffic between multiple log collectors.

## Step 1 – Web portal configuration: Define data sources and link them to a log collector

1.  Go to the automated upload setting page:<br></br> In the Cloud App Security portal, click the settings icon [settings icon](./media/settings-icon.png) followed by **Log collectors**.

2.  For each firewall or proxy from which you want to upload logs, create a matching data source:

    ![Windows 1](./media/windows1.png)

    a. Click **Add data source**.

    b. **Name** your proxy or firewall.

    c. Select the appliance from the **Source** list.

    d. Compare your log with the sample of the expected log format. If your log file format does not match this sample, you should add your data source as **Other**.

    e. Set the **Receiver type** to either **FTP**, **FTPS**, **Syslog – UDP**, or **Syslog – TCP**, or **Syslog – TLS**.

    > [!NOTE]
    > Integrating with secure transfer protocols (FTPS and Syslog – TLS) often requires additional setting or your Firewall/Proxy.

    f. Repeat this process for each firewall and proxy whose logs can be used to detect traffic on your network.

3.  Go to the **Log collectors** tab at the top.

    a. Click **Add log collector**.

    b. Give the log collector a **name**.

    c. Enter the **host IP address** of the machine you will use to deploy the Docker on.

    d. Select all **Data sources** that you want to connect to the collector, and click **Update** to save the configuration see the next deployment steps.

    ![Windows2](./media/windows2.png)

    > [!NOTE]
    > - A single Log collector can handle multiple data sources.

    > -   Copy the contents of the screen because you will need the information when you configure the Log Collector to communicate with Cloud App Security. If you selected Syslog, this information will include information about which port the Syslog listener is listening on.

4.  Further deployment information will appear.

    ![Windows3](./media/windows3.png)

5.  **Copy** the run command from the dialog. You can use the copy to
clipboard icon [copy to clipboard icon](./media/copy-icon.png).

6.  **Export** the expected data sources configuration. This configuration describes how you should set the log export in your appliances.

    ![Windows4](./media/windows4.png)

## Step 2 – On-premises deployment of your machine

>[!NOTE]
>The following steps describes the deployment in Windows Server. The deployment steps for other platforms are slightly different.

1.  **Download** the [latest stable Docker for Windows](https://docs.docker.com/docker-for-windows/install/\#download-docker-for-windows).
    
2.  **Run** the InstallDocker.msi installer.

3.  Open a PowerShell terminal on your Windows machine.

4.  **Connect** to the docker hub using the following command: `docker login -u cascollector`

5.  Use the following password `C0llector3nthusiast`

    ![windows5](./media/windows5.png)

6.  **Pull** to the collector image from the docker hub using the following command: `docker pull Microsoft/caslogcollector`

    ![windows6](./media/windows6.png)

7.  Deploy the collector image using the run command generated in the portal.

    ![windows8](./media/windows8.png)

    >[!NOTE]
    >If you need to configure a proxy add the proxy IP address and port under. For example, if your proxy details are 192.168.10.1:8080, your updated run command is:  
 `   Sudo docker run --name casCollector -p 21:21 -p 20000-20099:20000-20099 -e
    "PUBLICIP='192.168.1.1'" -e "PROXY=192.168.10.1:8080" -e
    "TOKEN=41f8f442c9a30519a058dd3bb9a19c79eb67f34a8816270dc4a384493988863a" -e
    "CONSOLE=tenant2.eu1-rs.adallom.com" -e "COLLECTOR=casCollector" --security-opt
    apparmor:unconfined --cap-add=SYS_ADMIN -dt microsoft/caslogcollector starter`

    ![windows9](./media/windows9.png)

9.  Verify the collector is running properly by running the following command: `Docker logs <collector_name>`

You should see the message **Finished successfully!**.
  ![windows10](./media/windows10.png)

## Step 4 - On-premises configuration of your network appliances

Configure your network firewalls and proxies to periodically export logs to the dedicated Syslog port of the FTP directory according to the directions in the dialog, for example:

        BlueCoat_HQ - Destination path: \<<machine_name>>\BlueCoat_HQ\

## Step 5 - Verify the successful deployment in the Cloud App Security portal

Check the collector status in the **Log collector** table and make sure the status is **Connected**. If it is **Created**, it is possible that the log collector connection and parsing has not completed.

![windows11](./media/windows11.png)

You can also go to the **Governance log** and verify that logs are being
periodically uploaded to the portal.

If you encounter problems during deployment, see [Troubleshooting Cloud
Discovery](troubleshooting-cloud-discovery.md).

## See also
 
[Create snapshot Cloud Discovery reports](create-snapshot-cloud-discovery-reports.md)

[Configure automatic log upload for continuous reports](configure-automatic-log-upload-for-continuous-reports.md)

[Working with Cloud Discovery data](working-with-cloud-discovery-data.md)
