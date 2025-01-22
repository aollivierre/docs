Below is an **updated guide** on enrolling Windows 11 devices into Intune using Windows Autopilot, incorporating feedback and best practices. It covers both *new* out-of-the-box devices and *existing* devices. Citations are kept in place for easy reference.

---

## 1. Introduction to Windows Autopilot

Windows Autopilot automates and streamlines the deployment of new or reset/reimaged Windows devices. It can:

- **Azure AD-join** (Microsoft Entra join) devices automatically.  
- **Enroll devices in Intune** during the Out-of-Box Experience (OOBE).  
- **Apply configuration profiles** and **apps** without IT intervention.  

> *References: [1](https://andrewstaylor.com/2024/09/02/enrolling-windows-devices-into-intune-a-definitive-guide/), [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints), [6](https://www.linkedin.com/pulse/windows-autopilot-ultimate-step-by-step-deployment-guide-robin-hobo)*

---

## 2. Prepare for Autopilot

### 2.1 Dynamic Security Group with ZTD ID

1. Go to **Microsoft Intune admin center** \(\[https://endpoint.microsoft.com\]\) > **Groups** > **New Group**.  
2. Set **Membership type** to **Dynamic Device**.  
3. Use a query that references **ZTDId**. For instance:  
   ```txt
   (device.devicePhysicalIds -any (_ -contains "[ZTDId]:"))
   ```
   This approach automatically captures devices stamped with a Zero Touch Deployment ID instead of relying on a manually assigned “OrderID” value.

> *Reference: [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints)*

### 2.2 Create and Assign a Windows Autopilot Deployment Profile

1. In Intune, go to: **Devices** > **Windows** > **Windows enrollment** > **Windows Autopilot deployment profiles**.  
2. Select **Create profile**, choose **Windows PC**.  
3. Configure OOBE settings, such as skipping privacy prompts and automatically configuring the region/time zone.  
4. Assign the profile to the **dynamic group** created above.

> *Reference: [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints)*

### 2.3 Import Devices into Autopilot

There are **three** common ways to register devices with Autopilot:

1. **Powershell Script from OSD Deploy** (Recommended for lab/testing or quick additions)  
   - Use the [OSD PowerShell module](https://github.com/OSDeploy/OSD) to capture and register the device’s hardware hash directly from a running OS or from the OOBE by pressing **Shift + F10**.  
   - This method is flexible and does not require CSV files.

2. **CSV Upload**  
   - Obtain a `.csv` file with device hardware hashes from your OEM partner or by manually extracting them with PowerShell.  
   - In Intune, go to **Devices** > **Windows** > **Windows enrollment** > **Devices** > **Import** to upload.

3. **OEM/Partner Import**  
   - Have your OEM or partner use **Partner Center** to register the devices in your tenant on your behalf.

> *References: [3](https://www.youtube.com/watch?v=qaur2ZsQt3g), [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints), [7](https://www.youtube.com/watch?v=r7bYyuHXCE0)*

---

## 3. Enrolling *New* Windows 11 Devices

1. **Power on & Connect to Network**  
   - Unbox the device, connect it to Wi-Fi or Ethernet, and power it on.  
2. **Autopilot Profile Application**  
   - During OOBE, the device checks for a registered hardware hash.  
   - If found, it applies the assigned **Autopilot profile**.  
3. **Azure AD Join and Intune Enrollment**  
   - The device joins Azure AD (Microsoft Entra) and enrolls in Intune silently.  

> *References: [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints), [6](https://www.linkedin.com/pulse/windows-autopilot-ultimate-step-by-step-deployment-guide-robin-hobo), [1](https://andrewstaylor.com/2024/09/02/enrolling-windows-devices-into-intune-a-definitive-guide/)*

### 3.1 Naming Templates

- You can **predefine a naming convention** in the Autopilot profile.  
- For example: **ORG-%SERIAL%** will prefix devices with your organization name or code, followed by the device’s serial number.

> *Reference: [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints)*

---

## 4. Enrolling *Existing* Windows 11 Devices

### 4.1 Clean Reinstall (Recommended)

A **clean installation** using a USB stick is often better than using “Reset this PC,” as it ensures a fully fresh Windows 11 image [1][7].  

1. **Create Bootable USB**  
   - Use [Windows 11 Media Creation Tool](https://www.microsoft.com/software-download/windows11) to build an 8GB+ USB installer.  
2. **Boot and Install**  
   - Restart, press **F12** (or similar) to select the USB device.  
   - Choose **Custom: Install Windows only (advanced)** and format existing partitions if desired.  
3. **OOBE & Autopilot**  
   - Upon first boot, if the device is **registered for Autopilot**, it will pull down the profile automatically.  

> *References: [1](https://www.reddit.com/r/Windows11/comments/1e1yj36/reset_vs_fresh_install_from_usb_stick/), [5](https://www.youtube.com/watch?v=ecJD1ORz4kI), [7](https://answers.microsoft.com/en-us/windows/forum/all/windows-11-reset-vs-windows-new-installation/3a4867ab-f12b-46f4-8d97-97b89ef366cd)*

### 4.2 Converting Existing Devices Already in Intune

If a device is **already enrolled in Intune** and marked as **Corporate** (not BYOD), you can enable **Autopilot conversion** directly in Intune:

1. In the **Microsoft Intune admin center**, locate **Devices** already enrolled and set to **Corporate**.  
2. Configure **Convert all targeted devices to Autopilot** for these devices.  
   - When they are reset or reimaged, they will pick up the Autopilot profile.  

> *Reference: [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints)*

### 4.3 Manual Enrollment (Not Recommended)

- If Autopilot registration is **not** possible, users can manually enroll devices.  
- **However, this is not ideal** for corporate devices and is generally more suitable for **BYOD** scenarios.  

1. **Install Company Portal App** from the Microsoft Store.  
2. Sign in with corporate credentials.  
3. Complete the enrollment prompts in **Settings** > **Accounts** > **Access work or school**.

> *References: [2](https://www.prajwaldesai.com/enroll-windows-11-devices-in-intune/), [5](https://itso.hkust.edu.hk/services/workplace-services/device-management/reinstall-device-Intune)*

---

## 5. Post-Enrollment Steps

1. **Windows Hello PIN**: If required by policy, the user will be prompted to set a PIN.  
2. **Device Naming**:  
   - If not already named via the Autopilot template, you can rename the device **in Intune** or **locally** in Windows.  
3. **Temporary Access Pass**:  
   - Consider using a [Temporary Access Pass](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-authentication-temporary-access-pass) for secure user sign-ins, especially during pre-provisioning in an office environment.  
4. **Verify Enrollment**:  
   - In **Settings** > **Accounts** > **Access work or school**, confirm the device is connected to your organization.  
   - Check **Intune Portal** to ensure compliance policies, apps, and configurations are applying.  

---

## 6. Additional Recommendations

### 6.1 Hybrid Join vs. Cloud Native

- For **existing devices**, you may maintain Hybrid Azure AD Join if that’s your current environment.  
- For **new devices**, consider going **Cloud Native** to reduce reliance on VPN and on-premises infrastructure.  

### 6.2 Pre-Provisioning (White Glove)

- **Pre-provisioning** allows IT to set up devices in advance, so users get a fully provisioned experience at first sign-in.  
- This is especially helpful if you have a custom image or a large suite of corporate apps.  

### 6.3 Autopilot 1.0 vs. Autopilot 2.0 (Device Prep)

- **Autopilot 2.0** (sometimes called “Device Prep”) is an evolving feature set.  
- Many organizations remain on the “traditional” Autopilot 1.0 approach until Autopilot 2.0 matures.  
- Evaluate **Windows Autopilot Device Prep** in a test environment before production rollout.

### 6.4 Imaging Considerations

- If you replace the OEM image with your own custom image (via SCCM, OSD Cloud, or other methods), ensure it is still **Windows 11 Pro/Enterprise** and remains **Autopilot-compatible** (i.e., no offline domain join or heavy customizations that conflict with cloud-native enrollment).  

---

## 7. References

1. [Andrew Staylor – Enrolling Windows Devices into Intune](https://andrewstaylor.com/2024/09/02/enrolling-windows-devices-into-intune-a-definitive-guide/)  
2. [Prajwal Desai – Enroll Windows 11 Devices in Intune](https://www.prajwaldesai.com/enroll-windows-11-devices-in-intune/)  
3. [YouTube – Autopilot Device Import Demo](https://www.youtube.com/watch?v=qaur2ZsQt3g)  
4. [Microsoft Learn – Cloud-Native Windows Endpoints](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints)  
5. [HKUST – Reinstall Device & Intune](https://itso.hkust.edu.hk/services/workplace-services/device-management/reinstall-device-Intune)  
6. [LinkedIn – Windows Autopilot Step-by-Step Guide](https://www.linkedin.com/pulse/windows-autopilot-ultimate-step-by-step-deployment-guide-robin-hobo)  
7. [YouTube – Autopilot Quick Start](https://www.youtube.com/watch?v=r7bYyuHXCE0)  
8. [YouTube – Autopilot Deep Dive](https://www.youtube.com/watch?v=UgHBDbYwy2w)  
9. [Reddit – Autopilot & Intune Questions](https://www.reddit.com/r/Intune/comments/1gthiqk/several_questions_around_setting_up_autopilot_and/)  
10. [Softlanding – Windows Autopilot Overview](https://www.softlanding.ca/blog/windows-autopilot-what-is-it-and-how-to-use-it/)  
11. [Microsoft Learn – Windows Enrollment Guide](https://learn.microsoft.com/en-us/mem/intune/fundamentals/deployment-guide-enrollment-windows)  

**Clean Installation References**  
- [1] [Reddit – Reset vs. Fresh Install from USB](https://www.reddit.com/r/Windows11/comments/1e1yj36/reset_vs_fresh_install_from_usb_stick/)  
- [5] [YouTube – Create Bootable USB](https://www.youtube.com/watch?v=ecJD1ORz4kI)  
- [7] [Microsoft Answers – Reset vs. New Installation](https://answers.microsoft.com/en-us/windows/forum/all/windows-11-reset-vs-windows-new-installation/3a4867ab-f12b-46f4-8d97-97b89ef366cd)  

---

### Conclusion

Following the steps above will help you **efficiently** and **securely** enroll both new and existing Windows 11 devices into Intune via Windows Autopilot. By leveraging ZTD IDs, scripting options (OSD PowerShell), dynamic groups, and best practices around imaging and naming, organizations can maximize the benefits of a **cloud-native** endpoint environment.