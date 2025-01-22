Below is a concise, step-by-step guide on how to enroll Windows 11 devices into Intune using Windows Autopilot. The process covers both *new* out-of-the-box devices and *existing* devices that you plan to reimage or reset. Citations are included in brackets for easy reference. 

---

## 1. Overview of Autopilot Enrollment

Windows Autopilot is a collection of technologies that simplifies the deployment and configuration of new Windows devices. It allows devices to be shipped directly to end users, who can then securely join them to your organization’s Azure AD (Microsoft Entra), and automatically enroll them into Intune without IT needing to touch the hardware.  

---

## 2. Preparing for Autopilot

### 2.1 Create a Dynamic Security Group in Microsoft Entra

1. In the [Microsoft Intune admin center](https://endpoint.microsoft.com/), go to **Groups** and create a new security group.  
2. Change the membership type to **Dynamic Device**.  
3. Use the query `(device.devicePhysicalIds -any (_ -eq "[OrderID]:CloudNative"))` if you want to dynamically capture devices tagged for your cloud-native Autopilot scenario.  

> *Reference: [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints)*

### 2.2 Create and Assign a Windows Autopilot Deployment Profile

1. In Intune, go to: **Devices** > **Windows** > **Windows enrollment** > **Windows Autopilot deployment profiles**.  
2. Click **Create profile**, select **Windows PC**.  
3. Configure OOBE (out-of-box experience) settings (e.g., skip privacy settings, automatically configure region, etc.).  
4. Assign the profile to the dynamic group you created above.  

> *Reference: [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints)*

### 2.3 Import Devices into Autopilot

- **Option 1**: Request a .csv file from your OEM partner containing the hardware hashes, and import it into Intune.  
- **Option 2**: Have your OEM/partner import devices through **Partner Center** on your behalf.  

> *Reference: [3](https://www.youtube.com/watch?v=qaur2ZsQt3g)*

---

## 3. Enrolling *New* Windows 11 Devices

1. **Power on and connect to internet**: Take the new device out of the box, power it on, and ensure it has network connectivity (Wi-Fi or Ethernet).  
2. **Autopilot profile application**:  
   - During OOBE, the device contacts the Autopilot service.  
   - The Autopilot profile you assigned is automatically applied.  
3. **Azure AD Join and Intune Enrollment**:  
   - The device joins Azure AD (Microsoft Entra) and enrolls in Intune silently, based on your deployment profile settings.  

> *References: [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints), [6](https://www.linkedin.com/pulse/windows-autopilot-ultimate-step-by-step-deployment-guide-robin-hobo), [1](https://andrewstaylor.com/2024/09/02/enrolling-windows-devices-into-intune-a-definitive-guide/)*

---

## 4. Enrolling *Existing* Windows 11 Devices

### 4.1 (Recommended) Perform a Clean Installation Using a USB Stick

A clean installation via USB is preferable to the built-in “Reset this PC” option, as it ensures a fully fresh Windows image [1][7].  

1. **Create a Bootable USB**:  
   - Use the official [Windows 11 Media Creation Tool](https://www.microsoft.com/software-download/windows11) to download and create installation media on a USB drive (≥8 GB).  
   - *References for clean installation:*  
     - [1](https://www.reddit.com/r/Windows11/comments/1e1yj36/reset_vs_fresh_install_from_usb_stick/)  
     - [5](https://www.youtube.com/watch?v=ecJD1ORz4kI)  
     - [7](https://answers.microsoft.com/en-us/windows/forum/all/windows-11-reset-vs-windows-new-installation/3a4867ab-f12b-46f4-8d97-97b89ef366cd)  

2. **Boot and Install**:  
   - Insert the USB drive into the target device.  
   - Restart and press **F12** (or the relevant key) to boot from the USB.  
   - Follow on-screen prompts, choosing **Custom: Install Windows only (advanced)**.  

3. **Complete OOBE**:  
   - Once Windows 11 is installed, connect to the internet.  
   - During OOBE, the device checks Autopilot registration; if present, it applies the Autopilot profile.  

> *Note:* If Autopilot registration does not occur automatically, see *Manual Enrollment* below.  

### 4.2 Manual Enrollment (If Autopilot Is Not Used)

1. **Install the Company Portal App**:  
   - From the **Microsoft Store**, install **Company Portal**.  
   - Alternatively, navigate to **Settings** > **Accounts** > **Access work or school**.  
2. **Sign In with Work Credentials**:  
   - Follow the prompts to enroll the device into Intune.  

> *Reference: [2](https://www.prajwaldesai.com/enroll-windows-11-devices-in-intune/)*

### 4.3 Post-Enrollment Steps

1. **Enable Windows Hello PIN** (if your organization requires it).  
2. **Rename the device** if needed.  
3. **Confirm Intune Enrollment**:  
   - In **Settings** > **Accounts** > **Access work or school**, ensure the device shows as connected to your organization.  
   - Or check the **Company Portal** app for enrollment status.  

> *References: [5](https://itso.hkust.edu.hk/services/workplace-services/device-management/reinstall-device-Intune), [2](https://www.prajwaldesai.com/enroll-windows-11-devices-in-intune/)*

---

## 5. Final Checks

- **Validate Device in Intune**: Ensure the device shows up under **Devices** in the Intune portal.  
- **Policy & App Deployment**: Confirm that your required policies and apps (e.g., Microsoft 365 Apps) are installing correctly.  
- **Monitoring**: Use **Endpoint analytics** and **Device compliance** reports in Intune to monitor device health and compliance.  

> *References: [1](https://andrewstaylor.com/2024/09/02/enrolling-windows-devices-into-intune-a-definitive-guide/), [4](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints), [10](https://www.softlanding.ca/blog/windows-autopilot-what-is-it-and-how-to-use-it/), [11](https://learn.microsoft.com/en-us/mem/intune/fundamentals/deployment-guide-enrollment-windows)*

---

## 6. References

**Autopilot & Intune Enrollment**  
[1] [Andrew Staylor – Enrolling Windows Devices into Intune](https://andrewstaylor.com/2024/09/02/enrolling-windows-devices-into-intune-a-definitive-guide/)  
[2] [Prajwal Desai – Enroll Windows 11 Devices in Intune](https://www.prajwaldesai.com/enroll-windows-11-devices-in-intune/)  
[3] [YouTube – Autopilot Device Import Demo](https://www.youtube.com/watch?v=qaur2ZsQt3g)  
[4] [Microsoft Learn – Cloud-Native Windows Endpoints](https://learn.microsoft.com/en-us/mem/solutions/cloud-native-endpoints/cloud-native-windows-endpoints)  
[5] [HKUST – Reinstall Device & Intune](https://itso.hkust.edu.hk/services/workplace-services/device-management/reinstall-device-Intune)  
[6] [LinkedIn – Windows Autopilot Step-by-Step Guide](https://www.linkedin.com/pulse/windows-autopilot-ultimate-step-by-step-deployment-guide-robin-hobo)  
[7] [YouTube – Autopilot Quick Start](https://www.youtube.com/watch?v=r7bYyuHXCE0)  
[8] [YouTube – Autopilot Deep Dive](https://www.youtube.com/watch?v=UgHBDbYwy2w)  
[9] [Reddit – Questions Around Autopilot](https://www.reddit.com/r/Intune/comments/1gthiqk/several_questions_around_setting_up_autopilot_and/)  
[10] [Softlanding – Windows Autopilot Overview](https://www.softlanding.ca/blog/windows-autopilot-what-is-it-and-how-to-use-it/)  
[11] [Microsoft Learn – Windows Enrollment Guide](https://learn.microsoft.com/en-us/mem/intune/fundamentals/deployment-guide-enrollment-windows)  

**Clean Installation (USB Method)**  
[1] [Reddit – Reset vs. Fresh Install from USB](https://www.reddit.com/r/Windows11/comments/1e1yj36/reset_vs_fresh_install_from_usb_stick/)  
[2] [ElevenForum – System Backup & Restore](https://www.elevenforum.com/t/windows-11-complete-system-backup-and-restore-not-using-macrium-reflect.16925/)  
[3] [ASUS – FAQ 1045873](https://www.asus.com/support/faq/1045873/)  
[4] [ASUS Canada – FAQ 1045873](https://www.asus.com/ca-en/support/faq/1045873/)  
[5] [YouTube – Create Bootable USB](https://www.youtube.com/watch?v=ecJD1ORz4kI)  
[6] [YouTube – Windows 11 Clean Install](https://www.youtube.com/watch?v=6s0cdMpixl8)  
[7] [Microsoft Answers – Reset vs. New Installation](https://answers.microsoft.com/en-us/windows/forum/all/windows-11-reset-vs-windows-new-installation/3a4867ab-f12b-46f4-8d97-97b89ef366cd)  
[8] [ASUS Canada – FAQ 1039507](https://www.asus.com/ca-en/support/faq/1039507/)  

---

**By following these steps and references, you can efficiently set up, reimage, and enroll Windows 11 devices into Intune via Windows Autopilot—ensuring a consistent, cloud-native endpoint experience for your organization.**