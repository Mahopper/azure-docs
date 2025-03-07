---
title: Azure Virtual Desktop disaster recovery plan
description: Make a disaster recovery plan for your Azure Virtual Desktop deployment to protect your data.
services: virtual-desktop
author: Heidilohr

ms.service: virtual-desktop
ms.topic: how-to
ms.date: 05/24/2022
ms.author: helohr
manager: femila
ms.custom: contperf-fy22q4
---

# Azure Virtual Desktop disaster recovery

To keep your organization's data safe, you should adopt and manage a business continuity and disaster recovery (BCDR) strategy. A sound BCDR strategy keeps your apps and workloads up and running during planned and unplanned service or Azure outages. These plans should cover the session host virtual machines (VMs) managed by customers, as opposed to the Azure Virtual Desktop service that's managed by Microsoft. For more information about management areas, see [Azure Virtual Desktop disaster recovery concepts](disaster-recovery-concepts.md).

The Azure Virtual Desktop service is designed with high availability in mind. Azure Virtual Desktop is a global service managed by Microsoft, with multiple instances of its independent components distributed across multiple Azure regions. If there's an unexpected outage in any of the components, your traffic will be diverted to one of the remaining instances or Microsoft will initiate a full failover to redundant infrastructure in another Azure region.

To make sure users can still connect during a region outage in session host VMs, you need to design your infrastructure with high availability and disaster recovery in mind. A typical disaster recovery plan includes replicating virtual machines (VMs) to a different location. During outages, the primary site fails over to the replicated VMs in the secondary location. Users can continue to access apps from the secondary location without interruption. On top of VM replication, you'll need to keep user identities accessible at the secondary location. If you're using profile containers, you'll also need to replicate them. Finally, make sure your business apps that rely on data in the primary location can fail over with the rest of the data.

To summarize, to keep your users connected during an outage, you'll need to do the following things:

- Replicate the VMs to a secondary location.
- If you're using profile containers, set up data replication in the secondary location.
- Make sure user identities you set up in the primary location are available in the secondary location. To ensure availability, make sure your Active Directory Domain Controllers are available in or from the secondary location.
- Make sure any line-of-business applications and data in your primary location are also failed over to the secondary location.

## Active-passive and active-active disaster recovery plans

There are two different types of disaster recovery infrastructure: active-passive and active-active. Each type of infrastructure works a different way, so let's look at what those differences are.

Active-passive plans are when you have a region with one set of resources that's active and one that's turned off until it's needed (passive). If the active region is taken offline by an outage or disaster, the organization can switch to the passive region by turning it on and directing all the users there.

Another option is an active-active deployment, where you use both sets of infrastructure at the same time. While some users may be affected by outages, the impact is limited to the users in the region that went down. Users in the other region that's still online won't be affected, and the recovery is limited to the users in the affected region reconnecting to the functioning active region. Active-active deployments can take many forms, including:

- Overprovisioning infrastructure in each region to accommodate affected users in the event one of the regions goes down. A potential drawback to this method is that maintaining the additional resources costs more.
- Have extra session hosts in both active regions, but deallocate them when they aren't needed, which reduces costs.
- Only provision new infrastructure during disaster recovery and allow affected users to connect to the newly provisioned session hosts. This method requires regular testing with infrastructure-as-code tools so you can deploy the new infrastructure as quickly as possible during a disaster.

For more information about types of disaster recovery plans you can use, see [Azure Virtual Desktop disaster recovery concepts](disaster-recovery-concepts.md).

Identifying which method works best for your organization is the first thing you should do before you get started. Once you have your plan in place, you can start building your recovery plan.

## VM replication

First, you'll need to replicate your VMs to the secondary location. Your options for doing so depend on how your VMs are configured:

- You can configure replication for all your VMs in both pooled and personal host pools with Azure Site Recovery. For more information about how this process works, see [Replicate Azure VMs to another Azure region](../site-recovery/azure-to-azure-how-to-enable-replication.md). However, if you have pooled host pools that you built from the same image and don't have any personal user data stored locally, you can choose not to replicate them. Instead, you have the option to build the VMs ahead of time and keep them powered off. You can also choose to only provision new VMs in the secondary region while a disaster is happening. If you choose these methods, you'll only need to set up one host pool and its related app groups and workspaces.
- You can create a new host pool in the failover region while keeping all resources in your failover location turned off. For this method, you'd need to set up new app groups and workspaces in the failover region. You can then use an Azure Site Recovery plan to turn on host pools.
- You can create a host pool that's populated by VMs built in both the primary and failover regions while keeping the VMs in the failover region turned off. In this case, you only need to set up one host pool and its related app groups and workspaces. You can use an Azure Site Recovery plan to power on host pools with this method.

We recommend you use [Azure Site Recovery](../site-recovery/site-recovery-overview.md) to manage replicating VMs to other Azure locations, as described in [Azure-to-Azure disaster recovery architecture](../site-recovery/azure-to-azure-architecture.md). We especially recommend using Azure Site Recovery for personal host pools because, true to their name, personal host pools tend to have something personal about them for their users. Azure Site Recovery supports both [server-based and client-based SKUs](../site-recovery/azure-to-azure-support-matrix.md#replicated-machine-operating-systems).

If you use Azure Site Recovery, you won't need to register these VMs manually. The Azure Virtual Desktop agent in the secondary VM will automatically use the latest security token to connect to the service instance closest to it. The VM (session host) in the secondary location will automatically become part of the host pool. The end-user will have to reconnect during the process, but apart from that, there are no other manual operations.

If there are existing user connections during the outage, before the admin can start failing over to the secondary region, you need to end the user connections in the current region.

To disconnect users in Azure Virtual Desktop (classic), run this cmdlet:

```powershell
Invoke-RdsUserSessionLogoff
```

To disconnect users in Azure Virtual Desktop, run this cmdlet:

```powershell
Remove-AzWvdUserSession
```

Once you've signed out all users in the primary region, you can fail over the VMs in the primary region and let users connect to the VMs in the secondary region.

## Virtual network

Next, consider your network connectivity during the outage. You'll need to make sure you've set up a virtual network (VNET) in your secondary region. If your users need to access on-premises resources, you'll need to configure this VNET to access them. You can establish on-premises connections with a VPN, ExpressRoute, or virtual WAN.

We recommend you use Azure Site Recovery to set up the VNET in the failover region because it preserves your primary network's settings and doesn't need peering.

## User identities

Next, ensure that the domain controller is available at the secondary location. 

There are three ways to keep the domain controller available:

   - Have one or more Active Directory Domain Controllers in the secondary location
   - Use an on-premises Active Directory Domain Controller
   - Replicate Active Directory Domain Controller using [Azure Site Recovery](../site-recovery/site-recovery-active-directory.md)

## Replicating user and app profile data

If you're using profile containers, the next step is to set up data replication to the secondary location.

You have five options to store FSLogix profiles:

   - Storage Spaces Direct (S2D)
   - Network drives (VM with extra drives)
   - Azure Files
   - Azure NetApp Files
   - Third-party storage services available on the Azure Marketplace

For more information, check out [Storage options for FSLogix profile containers in Azure Virtual Desktop](store-fslogix-profile.md).

If you're setting up disaster recovery for user profiles, then you'll need to either use the storage service to replicate the data to another region or use FSLogix Cloud Cache to manage the replication without using the underlying storage service to replicate the data.

Let's go over the five options for user profile disaster recovery plans in more detail in the following sections.

### Native Azure replication

One way you can set up disaster recovery is to set up native Azure replication. For example, you can set up native replication with  Azure Files Standard storage account replication, Azure NetApp Files replication, or Azure Files Sync for file servers.
    
>[!NOTE]
>NetApp replication is automatic after you first set it up. With Azure Site Recovery plans, you can add pre-scripts and post-scripts to fail over non-VM resources replicate Azure Storage resources.

### Storage Spaces Direct

Another option you can use is Storage Spaces Direct. Since Storage Spaces Direct handles replication across regions internally, you don't need to manually set up the secondary path.

### Network drives (VM with extra drives)

You can use VMs with extra drives for disaster recovery, too. If you replicate the network storage VMs using Azure Site Recovery like the session host VMs, then the recovery keeps the same path, which means you don't need to reconfigure FSLogix.

### Azure Files

Azure Files supports cross-region asynchronous replication that you can specify when you create the storage account. If the asynchronous nature of Azure Files already covers your disaster recovery goals, then you don't need to do extra configuration.

If you need synchronous replication to minimize data loss, then we recommend you use FSLogix Cloud Cache instead.

>[!NOTE]
>This section doesn't cover the failover authentication mechanism for Azure Files.

### Azure NetApp Files

You can also use Azure NetApp Files to replicate your Azure resources. Learn more about Azure NetApp Files at [Create replication peering for Azure NetApp Files](../azure-netapp-files/cross-region-replication-create-peering.md).

### FSLogix configuration

The FSLogix agent can support multiple profile locations using the standard [VHDLocations](/fslogix/profile-container-configuration-reference#vhd-locations) option. This method doesn't have anything to do with the Cloud Cache, so if you'd rather use the cache, skip ahead to [FSLogix Cloud Cache](#fslogix-cloud-cache). This option also doesn't replicate data, but instead allows you access to multiple storage providers that the FSLogix agent can look in to find or create your user profile. This option separately requires storage replication so that the profile can be made available in the secondary region.

To configure the registry entries:

1. Open the **Registry Editor**.
2. Go to **Computer** > **HKEY_LOCAL_MACHINE** > **SOFTWARE** > **FSLogix** > **Profiles**.
   
     > [!div class="mx-imgBorder"]
     > ![A screenshot of the Profiles window in the Registry Editor. VHDLocation is selected.](media/regedit-profiles.png)

3. Right-click on **VHDLocations** and select **Edit Multi-String**.

     > [!div class="mx-imgBorder"]
     > ![A screenshot of the Edit Multi-String window. The value data lists the Centrual US and East US locations.](media/multi-string-edit.png)

4. In the **Value Data** field, enter the locations you want to use.
5. When you're done, select **OK**.

If the first location is unavailable, the FSLogix agent will automatically fail over to the second, and so on.

We recommend you configure the FSLogix agent VHDLocation registry setting with both storage locations in both of the Azure locations you've deployed them. To configure the VHDLocation registry setting, you'll need to set up two different group policies. The first group policy is for the session hosts located in the primary region with the corresponding storage locations ordered with the primary first and the secondary second. The second group policy would be for the session hosts in the secondary location with the storage options reversed, so that the secondary storage location is listed first for only the VMs in the secondary or failover site.

For example, let's say your primary session host VMs are in the Central US region, and the profile container is also in the Central US region for performance reasons. In this case, you'd configure the FSLogix agent with a path to the storage in the Central US region listed first. Next, you'd configure the storage service you used in the previous example to replicate to the West US region. Once the path to Central US fails, the agent will try to load the profile in West US instead.

### VHDLocations

VHDLocations contributes to business continuity, but this setting wasn't only designed to be one part of a complete high availability or disaster recovery solution. The VHDLocations setting enables users to use a replicated or new profile in the event of a disaster, keeping users productive even in the event of an outage.

Here's how VHDLocations works, as well as some things you should consider if you plan to make VHDLocations part of your disaster recovery strategy:

- If the primary storage is unavailable for whatever reason and a user signs in, the FSLogix agent won't be able to access the existing user profile from that primary share. The user can still sign in, but FSLogix will either use the profile it finds in the secondary storage location (if you've already replicated it with storage replication) or it'll create a new profile on the secondary share. Because the user is now using either a replicated or new profile, they won’t be using their original profile. When they use this secondary profile, any updates they make will apply only to the secondary profile. They won't be able to access their original profile until the primary storage becomes available again and they sign back in.

- Once the primary storage is available again, the user won't be able to merge changes they made in the secondary or new profile back into the original profile. When a user signs in after the primary share is available again, they will return to using their original profile as it was before the disaster. Any changes they made in the secondary or new profile during the disaster will be lost.

### FSLogix Cloud cache

FSLogix supports replicating user and Office containers from the agent running on the session host itself. While you'll need to deploy multiple storage providers in multiple regions to store the replicated profiles, you won't need to configure the storage service's replication capabilities with multiple entries like you did with the VHDLocations settings in the previous section. However, before you start configuring FSLogix Cloud cache, you should be aware this method requires extra processing and storage space on the session host itself. Make sure you review [Cloud Cache to create resiliency and availability](/fslogix/cloud-cache-resiliency-availability-cncpt) before you get started.

You can configure FSLogix Cloud Cache directly in the registry based on the VHDLocations example in the previous section. However, we recommend you configure the cloud cache using a group policy instead. To create or edit a group policy object, go to **Computer Configuration** > **Administrative Templates** > **FSLogix** > **Profiles Containers (and Office 365 Containers, if necessary) > Cloud Cache - Cloud Cache Locations**. Once you've created or edited your policy object, you'll need to enable it, then list all storage provider locations you want the FSLogix to replicate it to, as shown in the following image. 

> [!div class="mx-imgBorder"]
> ![A screenshot of the FSLogix Cloud Cache Group Policy Cloud Cache Locations is selected.](media/fslogix-locations.png)

## Back up your data

You also have the option to back up your data. You can choose one of the following methods to back up your Azure Virtual Desktop data:

- For Compute data, we recommend only backing up personal host pools with [Azure Backup](../backup/backup-azure-vms-introduction.md). 
- For Storage data, the backup solution we recommend varies based on the back-end storage you used to store user profiles:
  - If you used Azure Files Share, we recommend using [Azure Backup for File Share](../backup/azure-file-share-backup-overview.md). 
  - If you used Azure NetApp Files, we recommend using either [Snapshots/Policies](../azure-netapp-files/snapshots-manage-policy.md) or [Azure NetApp Files Backup](../azure-netapp-files/backup-introduction.md).

## App dependencies

Finally, make sure that any business apps that rely on data located in the primary region can fail over to the secondary location. Also, be sure to configure the settings the apps need to work in the new location. For example, if one of the apps is dependent on the SQL backend, make sure to replicate SQL in the secondary location. You should configure the app to use the secondary location as either part of the failover process or as its default configuration. You can model app dependencies on Azure Site Recovery plans. To learn more, see [About recovery plans](../site-recovery/recovery-plan-overview.md).

## Disaster recovery testing

After you're done setting up disaster recovery, you'll want to test your plan to make sure it works.

Here are some suggestions for how to test your plan:

- If the test VMs have internet access, they'll take over any existing session host for new connections, but all existing connections to the original session host will remain active. Make sure the admin running the test signs out all active users before testing the plan. 
- You should only do full disaster recovery tests during a maintenance window to not disrupt your users.
- Make sure your test covers all business-critical applications and data.
- We recommend you only failover up to 100 VMs at a time. If you have more VMs than that, we recommend you fail them over in batches 10 minutes apart.

## Next steps

If you have questions about how to keep your data secure in addition to planning for outages, check out our [security guide](security-guide.md).
