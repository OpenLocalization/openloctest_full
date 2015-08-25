<properties
	pageTitle="Monitoring and Troubleshooting Guide for VMM and Hyper-V Site Protection" 
	description="Azure Site Recovery coordinates the replication, failover and recovery of virtual machines located on on-premises servers to Azure or a secondary datacenter. Use this article to monitor and troubleshoot  VMM or Hyper-V Site protection." 
	services="site-recovery" 
	documentationCenter="" 
	authors="anbacker" 
	manager="mkjain" 
	editor=""/>

<tags 
	ms.service="site-recovery" 
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="storage-backup-recovery" 
	ms.date="07/03/2015" 
	ms.author="anbacker"/>
	
# Monitor and troubleshoot protection for VMM and Hyper-V sites

This Monitoring and Troubleshooting Guide enables you to learn tracking the replication health and troubleshooting techniques for Azure Site Recovery.

## Understanding the components

### VMM Site Deployment for replication between on-premises site.

As part of setting up DR between two on-premises location; Azure Site
Recovery Provider needs to be downloaded and installed on the VMM
server. Provider needs internet connectivity to ensure that all the
operations triggered from Azure Portal gets translated to on-premises
operations like enable protection, shutdown primary side virtual
machines as part of failovers etc.

![](media/site-recovery-monitoring-and-troubleshooting/image1.png)

### VMM Site Deployment for replication between on-premises & Azure.

As part of setting up DR between on-premises & Azure; Azure Site
Recovery Provider needs to be downloaded and installed on the VMM server
along with Azure Recovery Services Agent which needs to be installed on
each Hyper-V host.

![](media/site-recovery-monitoring-and-troubleshooting/image2.png)

### Hyper-V Site Deployment for replication between on-premises & Azure

This is same as that of VMM Deployment – only different being Provider &
Agent gets installed on the Hyper-V host itself.

## Monitor configuration, protection and recovery operations

Every operation in ASR gets audited and is tracked under the “JOBS” tab.
In case of any configuration, protection or recovery error navigate to
the JOBS tab and see if there are any failures.

![](media/site-recovery-monitoring-and-troubleshooting/image3.png)

Once you find failures under the JOBS view, select the JOB and click
ERROR DETAILS for that job.

![](media/site-recovery-monitoring-and-troubleshooting/image4.png)

The error details will help you identify possible cause and
recommendation for the issue.

![](media/site-recovery-monitoring-and-troubleshooting/image5.png)

In the above case there seems to be another operation which is in
progress because of which Protection configuration is failing. Ensure
that you resolve the issue as per the recommendation – there-after click
RESART to re-initiate the operation.

![](media/site-recovery-monitoring-and-troubleshooting/image6.png)

Option to RESTART is not available for all operations – for those which
doesn’t have the RESTART option navigate back to the object and redo the
operation once again. Every JOB can be cancelled at any point of time
while in-progress using the CANCEL button.

![](media/site-recovery-monitoring-and-troubleshooting/image7.png)

## Monitor replication health for virtual machine

ASR provider central & remote monitoring through the Azure Portal for
each of the protected entities. Navigate to the PROTECTED ITEMS
there-after select VMM CLOUDS or PROTECTION GROUPS. VMM CLOUDS tab is
only for VMM based deployments and all other scenarios have the
protected entities under PROTECTION GROUPS tab.

![](media/site-recovery-monitoring-and-troubleshooting/image8.png)

There-after select the protected entity under the respective cloud or
the protection group. Once you select the protected entity all allowed
operations are shown in the bottom pane.

![](media/site-recovery-monitoring-and-troubleshooting/image9.png)

As shown above in-case the virtual machine HEALTH is critical – you can
click the ERROR DETAILS button on the bottom to see the error. Based on
the “Possible causes” and “Recommendation” mentioned resolve the issue –
here in this case the virtual machine needs to be re-synchronized which
can be done from the portal itself by clicking the RESYNCHRONIZE button.

![](media/site-recovery-monitoring-and-troubleshooting/image10.png)

![](media/site-recovery-monitoring-and-troubleshooting/image11.png)

Note: If there are any active operations which are in-progress or failed
then navigate to the JOBS view as mentioned earlier to view the JOB
specific error.

## Troubleshoot on-premises issues

Connect to the on-premises Hyper-V manager console, select the virtual
machine and see the replication health.

![](media/site-recovery-monitoring-and-troubleshooting/image12.png)

In this case *Replication Health* is being indicated as Critical – *View
Replication Health* to see the details.

![](media/site-recovery-monitoring-and-troubleshooting/image13.png)

#### Event Viewer

| Scenarios               	| Event Sources                                                                                                                                                                                        	|
|-------------------------	|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
| VMM Site Protection     	|  VMM Server <ul><li> **Applications and Service Logs/Microsoft/VirtualMachineManager/Server/Admin** </li></ul> Hyper-V Host <ul><li> **Applications and Service Logs/MicrosoftAzureRecoveryServices/Replication** (For Azure as Target)</li><li> **Applications and Service Logs/Microsoft/Windows/Hyper-V-VMMS/Admin** </li></ul> |
| Hyper-V Site Protection 	| <ul><li> **Applications and Service Logs/MicrosoftAzureRecoveryServices/Replication** </li><li> **Applications and Service Logs/Microsoft/Azure Site Recovery/Provider/Operational** </li><li>	**Applications and Service Logs/Microsoft/Windows/Hyper-V-VMMS/Admin** </li><ul>|

#### Hyper-V Replication Logging Options

All events pertaining to Hyper-V Replica are logged in the
Hyper-V-VMMS\\Admin log located under **Applications and Services
Logs\\Microsoft\\Windows**. In addition, an Analytic log can be enabled
for Hyper-V-VMMS. To enable this log, first make the Analytic and Debug
logs viewable in the Event Viewer. Open Event Viewer, then in the **View
menu**, click **Show Analytic and Debug logs**.

![](media/site-recovery-monitoring-and-troubleshooting/image14.png)

An Analytic log is visible under Hyper-V-VMMS

![](media/site-recovery-monitoring-and-troubleshooting/image15.png)

In the **Actions** pane, click on **Enable Log**. Once enabled, it
appears in **Performance Monitor** as an Event Trace Session located
under **Data Collector Sets.**

![](media/site-recovery-monitoring-and-troubleshooting/image16.png)

To view the information collected, first stop the tracing session by
disabling the log, and then save the log and re-open it in Event Viewer
or use other tools to convert it as desired.


## Understanding the virtual machine life cycle

![](media/site-recovery-monitoring-and-troubleshooting/image17.png)


## Reaching out for Microsoft Support

### Log collection

For VMM Site protection, refer [ASR Log Collection using Support
Diagnostics Platform (SDP)
Tool](http://social.technet.microsoft.com/wiki/contents/articles/28198.asr-data-collection-and-analysis-using-the-vmm-support-diagnostics-platform-sdp-tool.aspx)
to collect the required logs.

For Hyper-V Branch Office and SMB Site protection, download the
[tool](https://dcupload.microsoft.com/tools/win7files/DIAG_ASRHyperV_global.DiagCab)
& execute it on the Hyper-V host to collect the logs.

For VMware/Physical scenarios, refer [Azure Site Recovery Log Collection
for VMware and Physical site
protection](http://social.technet.microsoft.com/wiki/contents/articles/30677.azure-site-recovery-log-collection-for-vmware-and-physical-site-protection.aspx)
to collect the required logs.

SDP tool collects the log file locally which you can also locate under a randomly named subfolder under **%LocalAppData%\ElevatedDiagnostics**

### Opening a support ticket

To raise support ticket for ASR, reach out to Azure Support using the
URL at <http://aka.ms/getazuresupport>

## KB Articles

-   [How to preserve the drive letter for protected virtual machines
    > that are failed over or migrated to
    > Azure](http://support.microsoft.com/kb/3031135)

-   [How to troubleshoot Azure Recovery
    > Services](http://support.microsoft.com/kb/3005185)

-   [How to Enable Debug Logging for the Azure Site Recovery in Hyper-V
    > Site Protection](http://support.microsoft.com/kb/3033922)

-   [ASR: "The cluster resource could not be found" error when you try
    > to enable protection for a virtual
    > machine](http://support.microsoft.com/kb/3010979)
    
-   [Understand & Troubleshoot Hyper-V Replica
    > Guide](http://www.microsoft.com/en-in/download/details.aspx?id=29016) 
	
	
	
## Common ASR errors and their resolutions

Below are the common errors that you may hit and their resolutions. Each
of the error is documented in a separate WIKI page.

### Setup

-   [The selected certificate cannot be validated. Please select a
    different certificate.](http://social.technet.microsoft.com/wiki/contents/articles/25569.the-selected-certificate-cannot-be-validated-please-select-a-different-certificate.aspx)

-   [The VMM server cannot be registered due to an internal error.
    Please refer to the jobs view in the Site Recovery Portal for more
    details on the error. Run Setup again to register the
    server.](http://social.technet.microsoft.com/wiki/contents/articles/25570.the-vmm-server-cannot-be-registered-due-to-an-internal-error-please-refer-to-the-jobs-view-in-the-site-recovery-portal-for-more-details-on-the-error-run-setup-again-to-register-the-server.aspx)

-   [A connection can’t be established to the Hyper-V Recovery Manager
    vault. Verify the proxy settings or try again
    later.](http://social.technet.microsoft.com/wiki/contents/articles/25571.a-connection-cant-be-established-to-the-hyper-v-recovery-manager-vault-verify-the-proxy-settings-or-try-again-later.aspx)

### Configuration

-   [Hyper-V host cluster contains at least one static network adapter,
    or no connected adapters are configured to use DHCP.](http://social.technet.microsoft.com/wiki/contents/articles/25498.hyper-v-host-cluster-contains-at-least-one-static-network-adapter-or-no-connected-adapters-are-configured-to-use-dhcp.aspx)

-   [The Hyper-V profile isn't enabled in the Capability Profiles for
    cloud](http://social.technet.microsoft.com/wiki/contents/articles/25499.the-hyper-v-profile-isn-t-enabled-in-the-capability-profiles-for-cloud.aspx)

-   [Protection configuration for '%CloudName;' couldn't be applied. A
    newly added Hyper-V host or cluster couldn't be configured because
    cloud protection isn't
    configured.](http://social.technet.microsoft.com/wiki/contents/articles/25500.protection-configuration-for-cloudname-couldn-t-be-applied-a-newly-added-hyper-v-host-or-cluster-couldn-t-be-configured-because-cloud-protection-isn-t-configured.aspx)

-   [VMM does not have permissions to complete an
    action](http://social.technet.microsoft.com/wiki/contents/articles/31110.vmm-does-not-have-permissions-to-complete-an-action.aspx)

### Protection

-   [The operating system is not
    supported](http://social.technet.microsoft.com/wiki/contents/articles/31103.the-operating-system-is-not-supported.aspx)

-   [Enable protection failed since Agent not installed on host
    machine](http://social.technet.microsoft.com/wiki/contents/articles/31105.enable-protection-failed-since-agent-not-installed-on-host-machine.aspx)

-   [A suitable host for the replica virtual machine can't be found -
    Due to low compute
    resources](http://social.technet.microsoft.com/wiki/contents/articles/25501.a-suitable-host-for-the-replica-virtual-machine-can-t-be-found-due-to-low-compute-resources.aspx)

-   [A suitable host for the replica virtual machine can't be found -
    Due to no logical network
    attached](http://social.technet.microsoft.com/wiki/contents/articles/25502.a-suitable-host-for-the-replica-virtual-machine-can-t-be-found-due-to-no-logical-network-attached.aspx)

-   [Cannot connect to the replica host machine - connection could not
    be
    established](http://social.technet.microsoft.com/wiki/contents/articles/31106.cannot-connect-to-the-replica-host-machine-connection-could-not-be-established.aspx)

### Recovery

-   VMM cannot complete the host operation -

    -   [Fail over to the selected recovery point for virtual machine:
        General access denied
        error.](http://social.technet.microsoft.com/wiki/contents/articles/25504.fail-over-to-the-selected-recovery-point-for-virtual-machine-general-access-denied-error.aspx)

    -   [Hyper-V failed to fail over to the selected recovery point for
        virtual machine: Operation aborted Try a more recent recovery
        point.
        (0x80004004)](http://social.technet.microsoft.com/wiki/contents/articles/25503.hyper-v-failed-to-fail-over-to-the-selected-recovery-point-for-virtual-machine-operation-aborted-try-a-more-recent-recovery-point-0x80004004.aspx)

    -   A connection with the server could not be established
        (0x00002EFD)

        -   [Hyper-V failed to enable reverse replication for virtual
            machine](http://social.technet.microsoft.com/wiki/contents/articles/25505.a-connection-with-the-server-could-not-be-established-0x00002efd-hyper-v-failed-to-enable-reverse-replication-for-virtual-machine.aspx)

        -   [Hyper-V failed to enable replication for virtual machine
            virtual
            machine](http://social.technet.microsoft.com/wiki/contents/articles/25506.a-connection-with-the-server-could-not-be-established-0x00002efd-hyper-v-failed-to-enable-replication-for-virtual-machine-virtual-machine.aspx)

    -   [Could not commit failover for virtual
        machine](http://social.technet.microsoft.com/wiki/contents/articles/25508.could-not-commit-failover-for-virtual-machine.aspx)

-   [The recovery plan contains virtual machines which are not ready for
    planned
    failover](http://social.technet.microsoft.com/wiki/contents/articles/25509.the-recovery-plan-contains-virtual-machines-which-are-not-ready-for-planned-failover.aspx)

-   [The virtual machine isn't ready for planned
    failover](http://social.technet.microsoft.com/wiki/contents/articles/25507.the-virtual-machine-isn-t-ready-for-planned-failover.aspx)

-   [Virtual machine is not running and is not powered
    off](http://social.technet.microsoft.com/wiki/contents/articles/25510.virtual-machine-is-not-running-and-is-not-powered-off.aspx)

-   [Out of band operation happened on a virtual machine
    and ](http://social.technet.microsoft.com/wiki/contents/articles/25507.the-virtual-machine-isn-t-ready-for-planned-failover.aspx)commit failover
    failed

-   Test Failover

    -   [Failover could not be initiated since test failover is in
        progress](http://social.technet.microsoft.com/wiki/contents/articles/31111.failover-could-not-be-initiated-since-test-failover-is-in-progress.aspx)

-   Specific to recovery to Azure

    -   [Failover could not be started due to unsupported characters in
        the VM
        name](http://social.technet.microsoft.com/wiki/contents/articles/31107.failover-could-not-be-started-due-to-unsupported-characters-in-the-vm-name.aspx)

### Configuration Server, Process Server, Master Target

Configuration Server (CS), Process Server (PS), Master Targer (MT)

-   [The ESXi host on which the PS/CS is hosted as a VM fails with a purple
    screen of
    death.](http://social.technet.microsoft.com/wiki/contents/articles/31107.vmware-esxi-host-experiences-a-purple-screen-of-death.aspx)


