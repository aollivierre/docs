Below is a practical, step-by-step guide that you can use (or share with other IT administrators) when you need to offboard a Windows device from your environment. It covers three main scenarios: 

1. **Cloud-Native (Azure AD Join) devices**  
2. **Hybrid Azure AD Join devices**  
3. **On-Premise (legacy) devices that are *not* hybrid or cloud-native**  

Throughout this guide, “Microsoft Entra” refers to Microsoft Entra ID (formerly Azure AD), and we use the term “Intune” for Microsoft Intune.  

---  



## **Important Note on Microsoft Entra Connect Sync vs. Microsoft Entra Cloud Sync**  

As of **January 2025**, there is a critical difference to be aware of regarding device synchronization from your on-premises Active Directory to Microsoft Entra ID:

1. **Microsoft Entra Connect Sync (Traditional AD Connect)**  
   - **Capable of syncing device objects** (i.e., computer accounts) from on-prem AD to Microsoft Entra ID.  
   - Uses OU scoping and custom synchronization rules (Sync Rule Editor) to include or exclude certain containers or objects.  
   - For device offboarding (especially hybrid-joined devices), **this** is the solution that ensures the device object removal in on-prem AD will flow up to Microsoft Entra ID.

2. **Microsoft Entra Cloud Sync (Cloud Provisioning Agent)**  
   - **Cannot sync device objects** at this time (as of January 2025).  
   - Ideal for synchronizing users and groups where traditional AD Connect is not feasible or to reduce overhead for certain scenarios.  
   - Because it does not sync device objects, **do not rely on Cloud Sync** for any tasks involving device synchronization.  

### **Recommended OU Scoping Configuration**  

1. **Include Device OUs in Entra Connect Sync**  
   - Ensure the on-prem OU(s) containing your computer objects and servers are **not** excluded in the **traditional Microsoft Entra Connect Sync** configuration.  
   - Verify the Sync Rule Editor also does not have any exclusion rules set for these OUs or the “computer” object class.

2. **Exclude Device OUs (or Computer OUs) from Cloud Sync**  
   - Since Cloud Sync cannot handle devices, it’s best practice to **not** scope the same device OUs for Cloud Sync.  
   - This avoids any confusion about where the device synchronization is happening or not happening.  

3. **Dedicated OU for Disabled Computers**  
   - Maintain a **dedicated OU for disabled computer objects** in on-prem AD.  
   - Exclude that OU from **both** Cloud Sync and Connect Sync, so disabled devices do not attempt to re-sync into Microsoft Entra ID.  
   - This keeps your directory and Entra ID environment clean and prevents re-enabling devices by accident.

By keeping these points in mind, you can ensure a smooth offboarding process for any hybrid-joined devices and prevent orphaned or stale device objects in your environment.





## Prerequisites

1. **Check Device Join Type and Icon**  
   - **Purple icon** in Microsoft Entra ID typically indicates a cloud-native (Azure AD Join) device.  
   - **Blue icon** in Microsoft Entra ID typically indicates a hybrid-joined device.  

2. **Confirm AD DS (Active Directory Domain Services) Membership**  
   - Devices that are strictly on-prem (without hybrid or cloud-native features) may not appear in Microsoft Entra ID.  
   - Hybrid-joined devices will appear both in your on-prem AD and in Microsoft Entra ID with a “Hybrid” join type.  

3. **Permissions**  
   - Ensure you have the proper admin rights in Active Directory (on-premises) and the necessary roles to remove devices from Microsoft Entra ID, Intune, and Autopilot (e.g., Intune Admin, Global Admin, or an equivalent role).  

4. **Scheduling**  
   - Plan for a short window during which you offboard the device to avoid synchronization conflicts.  
   - Note that Azure AD Connect (Microsoft Entra Connect) typically synchronizes changes from your on-prem AD to Microsoft Entra ID every 30 minutes (by default).  

---  

## Common Step: Back Up BitLocker Recovery Keys

Before starting the offboarding process in any scenario, **always** ensure that any BitLocker recovery keys stored in the Microsoft Intune portal are backed up.  

1. **Locate Keys in Intune**  
   - In the [Intune admin console](https://intune.microsoft.com), go to **Devices** \> **All devices** \> select the device \> **Properties** \> **Monitor** \> **Recovery keys** (or similar navigation depending on Intune UI changes).  
   - Export/record the key in a secure place.  

2. **Verify You Have the Right Key**  
   - If you have multiple keys (e.g., if the device had multiple OS drives or had been re-encrypted), confirm you’re saving the latest valid key.  

---

## Scenario 1: Offboarding a **Cloud-Native (Azure AD Join) Device** (Purple Icon)

1. **Confirm Device is Cloud-Native**  
   - In [Microsoft Entra admin center](https://entra.microsoft.com), go to **Devices**.  
   - Look for the device with a **purple icon** and a Join Type of **Azure AD joined**.  

2. **Back Up BitLocker Recovery Keys** (Common Step)  
   - If any keys are in Intune, back them up to a secure location.  

3. **Remove the Device from Autopilot (If Applicable)**  
   - If the device was enrolled via Autopilot, open the [Microsoft Intune Autopilot Devices list](https://intune.microsoft.com/#view/Microsoft_Intune_Enrollment/AutopilotDevices.ReactView/filterOnManualRemediationRequired~/false).  
   - Locate the device’s serial number/Hardware ID.  
   - **Delete** or **De-register** the device from Autopilot.  

4. **Remove the Device from Microsoft Entra ID**  
   - In the Microsoft Entra admin center, under **Devices**, locate the device.  
   - Select **Delete** to remove the entry from Microsoft Entra ID.  

5. **Remove the Device from Intune**  
   - Open the [Microsoft Intune admin center](https://intune.microsoft.com).  
   - Go to **Devices** \> **All devices**, select the device, then select **Delete**.  

6. **Verify Removal**  
   - Ensure the device no longer appears in both the Intune device list and the Microsoft Entra ID device list.  

At this point, the device is fully offboarded from the cloud environment. Because it was a pure Azure AD Join, there should be no residual objects in on-prem Active Directory (unless it also existed there as a stale computer object which you can optionally remove/disable and move to a non-synced OU).  

---

## Scenario 2: Offboarding a **Hybrid Azure AD Join Device** (Blue Icon)

1. **Confirm Device is Hybrid-Joined**  
   - In [Microsoft Entra admin center](https://entra.microsoft.com), navigate to **Devices**.  
   - A **blue icon** with the Join Type listed as “Hybrid Azure AD joined” indicates the device is synced from on-prem AD.  

2. **Back Up BitLocker Recovery Keys** (Common Step)  

3. **Move or Disable the Device in On-Prem AD**  
   - Open **Active Directory Users and Computers** on your on-prem domain controller or management console.  
   - Locate the computer account.  
   - Move the computer object to the “Disabled Computers” OU (or an equivalent OU designated for decommissioned/offboarded devices).  
   - (Optional) You can delete the computer object if you do not use a “Disabled OU” procedure—but typically moving it to a Disabled OU first is recommended to avoid accidental re-adds.  

4. **Allow Microsoft Entra Connect Sync**  
   - Wait for the synchronization to occur (by default up to 30 minutes).  
   - Once the on-prem AD object is removed or moved to a disabled OU, it will be updated accordingly in Microsoft Entra ID.  
   - You can optionally force a sync using **PowerShell** if you have the need and permissions:  
     ```powershell
     Start-ADSyncSyncCycle -PolicyType Delta
     ```  

5. **Remove the Device from Autopilot (If Applicable)**  
   - If you used Autopilot for this device, go to the [Autopilot Devices list](https://intune.microsoft.com/#view/Microsoft_Intune_Enrollment/AutopilotDevices.ReactView).  
   - Locate the device’s serial number/Hardware ID.  
   - **Delete** or **De-register** it.  

6. **Remove the Device from Microsoft Entra ID**  
   - Once the on-prem computer account is either removed or disabled, that change syncs to Microsoft Entra ID.  
   - Verify in the Microsoft Entra admin center that the device object has been removed or shows as disabled. If it remains visible, you can manually **Delete** from the Entra admin center after it’s no longer anchored by the on-prem AD object.  

7. **Remove the Device from Intune**  
   - Open the [Microsoft Intune admin center](https://intune.microsoft.com).  
   - Go to **Devices** \> **All devices**, select the device, then select **Delete**.  

8. **Verify Removal**  
   - Check that the device no longer appears in Intune.  
   - Confirm that the device object has been removed or disabled in both on-prem AD and Microsoft Entra ID.  

---

## Scenario 3: Offboarding an **On-Premise (Legacy AD Joined Only) Device** That Is *Not* Hybrid or Cloud-Native

For devices that are never registered in Microsoft Entra ID (i.e., no hybrid or Azure AD join), the process is simpler, focusing primarily on your on-prem domain.

1. **Back Up BitLocker Recovery Keys** (Common Step)  
   - If your organization uses on-premises management (e.g., MBAM or your own system) for BitLocker recovery keys, ensure those keys are backed up or remain accessible.  

2. **Remove or Disable the Device in On-Prem AD**  
   - Use **Active Directory Users and Computers** to locate the computer object.  
   - Move it to your “Disabled Computers” OU or delete it entirely.  

3. **No Action Required in Microsoft Entra / Intune**  
   - Since these devices are not hybrid or cloud-managed, you will not find any corresponding object in Microsoft Entra ID or Intune.  
   - If you do see a stale entry (for whatever reason), remove it from Microsoft Entra ID or Intune (very unlikely in a purely on-prem device scenario).  

4. **Verify Removal**  
   - Confirm that the device no longer appears in your on-prem AD environment.  

---

## Additional Tips & Notes

- **Timing and Sync Intervals**: If you remove the computer object from on-prem AD, you must allow for an Azure AD Connect sync cycle (or do a manual sync) before the device is cleaned up in Microsoft Entra ID.  
- **Delegated OU**: Some organizations use a delegated “Disabled” OU to hold decommissioned devices for a certain retention period before final deletion. This allows for easy reactivation if needed.  
- **Autopilot**: Always remember to remove the device from Autopilot if it was ever enrolled. This prevents future re-enrollment confusion and potential license usage.  
- **BitLocker Key Retention**: Depending on compliance requirements, you may need to store keys for a set time even after device offboarding. Document your organization’s policies accordingly.  
- **Licenses**: Offboarding a device (especially from Intune) may free up a license seat. Regularly audit your Intune and Microsoft 365 licensing in tandem with device offboardings.  

---

### Summary Workflow Diagram

```powershell
flowchart TD
    A[Start Offboarding] --> B[Backup BitLocker Keys]
    B --> C{Determine Join Type?}
    C --> D[Cloud Native (Purple)]
    C --> E[Hybrid (Blue)]
    C --> F[On-Prem Only]
    D --> D1[Remove from Autopilot]
    D --> D2[Remove from Entra ID]
    D --> D3[Remove from Intune]
    E --> E1[Disable/Remove from On-Prem AD]
    E --> E2[Wait for Entra Connect Sync]
    E --> E3[Remove from Autopilot if used]
    E --> E4[Remove from Intune]
    F --> F1[Disable/Remove from On-Prem AD]
    F --> F2[No Action in Entra/Intune (unless stale object)]
    F --> End[Verification]
    D3 --> End[Verification]
    E4 --> End[Verification]
```

This flow helps visualize at a glance the steps needed for each join type.

---

## Conclusion

By following the steps above—especially ensuring BitLocker recovery keys are secured first—you’ll maintain a thorough, compliant offboarding process for Windows devices across cloud-native, hybrid, and purely on-prem scenarios. Properly managing each step helps maintain security, license compliance, and accurate device inventory.