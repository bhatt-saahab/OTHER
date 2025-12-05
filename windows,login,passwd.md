 
This guide provides a step-by-step method to reset a forgotten Windows password using the Sticky Keys trick in Windows Recovery Environment (WinRE). It’s tested and effective for Windows 10/11 as of October 08, 2025.

**Warnings:**
- Requires admin access or WinRE entry.
- Back up files if possible.
- Temporarily modifies Sticky Keys; restoration steps included.
- Seek technical assistance if unsure.

Follow each section carefully.

## 1. Enter WinRE
Access the recovery mode to start the process.

1. Restart your computer while holding **Shift**, or force restart 3 times if locked out.
2. From the blue screen, select **Troubleshoot** > **Advanced options** > **Command Prompt**.

   *Note:* If prompted for a password, try skipping or using another account. Alternatively, use a Windows recovery USB (search online for creation instructions).

## 2. Backup and Replace Sticky Keys File
Modify system files to enable the trick.

1. In Command Prompt, back up the Sticky Keys file:
   ```cmd
   copy C:\Windows\System32\sethc.exe C:\Windows\System32\sethc.exe.bak
   ```
   Confirm: “1 file(s) copied.”

2. Replace Sticky Keys with Command Prompt:
   ```cmd
   copy C:\Windows\System32\cmd.exe C:\Windows\System32\sethc.exe
   ```
   If prompted to overwrite, type `Y` and press Enter.

   *Explanation:* This replaces `sethc.exe` (Sticky Keys) with `cmd.exe`, allowing a command prompt to open later.

## 3. Reboot to Login Screen
Restart the system to access the login interface.

1. Reboot the computer:
   ```cmd
   wpeutil reboot
   ```

2. Wait for the Windows login or lock screen to appear.

## 4. Access Command Prompt via Sticky Keys
Trigger the modified Sticky Keys to open a command prompt.

1. At the login/lock screen, press **Shift** 5 times quickly.
2. A Command Prompt window opens with admin privileges (`Administrator: C:\Windows\System32\cmd.exe`).

## 5. Reset Password
Change the account password using the command prompt.

1. Set a new password (replace `USERNAME` with your account name, `NEW_PASSWORD` with your desired password):

2. ```cmd
   net user #see user name 
     ```
   ```cmd
   net user USERNAME NEW_PASSWORD
   ```
   Confirm: “The command completed successfully.”

   *Tip:* If unsure of your username, type `net user` to list all users.

3. Close the prompt:
   ```cmd
   exit
   ```

## 6. Log In
Test the new credentials.

1. At the login screen, enter your username and new password.
2. Log in to Windows.

   *Troubleshooting:* If login fails, verify username/password accuracy and retry the steps.

## 7. Restore Sticky Keys
Revert changes to restore normal Sticky Keys functionality.

1. Open Command Prompt as administrator (Right-click Start > Search “cmd” > Run as administrator).
2. Restore the original file:
   ```cmd
   copy C:\Windows\System32\sethc.exe.bak C:\Windows\System32\sethc.exe
   ```
   If prompted to overwrite, type `Y` and press Enter.

3. (Optional) Disable Sticky Keys shortcut:
   ```cmd
   reg add "HKCU\Control Panel\Accessibility\StickyKeys" /v Flags /t REG_SZ /d 506 /f
   ```
   This disables the 5x Shift trigger. To re-enable, use `/d 510`.
## Additional Tips
- **Error Recovery:** If issues occur, restore the backup in WinRE using:
   ```cmd
   copy C:\Windows\System32\sethc.exe.bak C:\Windows\System32\sethc.exe
   ```
- **Security:** Update to a strong password post-login (Settings > Accounts > Sign-in options).
- **Quick Restore:** After login, use this to restore Sticky Keys in one command:
   ```cmd
   copy C:\Windows\System32\sethc.exe.bak C:\Windows\System32\sethc.exe /Y
   ```
- For visual guidance, search YouTube for “Windows Sticky Keys password reset.”

You’re done! If issues persist, share details for further assistance.

# Complete Step-by-Step Guide: Resetting Windows Passwords Using Hiren's BootCD PE x64

Hiren’s BootCD PE x64 is a powerful, bootable rescue environment based on Windows PE (Preinstallation Environment). It includes tools for diagnosing, repairing, and recovering Windows systems, including resetting forgotten local passwords. **Note:** This guide focuses on local accounts only. It won't work for Microsoft online accounts or encrypted drives (e.g., BitLocker without the key). Always back up data before proceeding.

This guide covers **two reliable methods** using tools from Hiren's BootCD PE:
- **Method 1: Lazesoft Password Recovery** (Graphical, beginner-friendly).
- **Method 2: NT Password Edit** (Manual, for advanced users or when Method 1 fails).

## Prerequisites
- A USB drive (4–8 GB minimum).
- A working computer to create the bootable USB.
- Access to the locked Windows PC.

---

## Step 1: Download Hiren's BootCD PE x64
1. Visit the official website: [https://www.hirensbootcd.org/download/](https://www.hirensbootcd.org/download/).
2. Download the **Hiren’s BootCD PE x64 ISO file** (free, ~3–4 GB).

---

## Step 2: Create a Bootable USB Using Rufus
1. Download **Rufus** from its official site: [https://rufus.ie/](https://rufus.ie/).
2. Insert your USB drive into the working computer.
3. Open Rufus and configure:
   - **Device**: Select your USB drive.
   - **Boot selection**: Click **SELECT** and choose the downloaded Hiren’s BootCD PE ISO file.
   - **Partition scheme**: Choose **MBR** (for BIOS/legacy systems) or **GPT** (for UEFI systems—check your PC's firmware).
4. Click **START** and wait for the process to complete (5–10 minutes). Safely eject the USB when done.

---

## Step 3: Boot from the USB Drive
1. Insert the USB into the locked Windows PC.
2. Restart the computer.
3. Enter the boot menu by pressing the appropriate key (common ones: **F12**, **ESC**, **F9**, **F8**, **F10**, or **DEL**—check your motherboard manual).
4. Select the USB device from the boot menu (it may appear as "USB HDD" or "Hiren's BootCD").
5. Wait for Hiren's BootCD PE to load (it boots into a Windows 10-style desktop—see **Screenshot 1** for reference).

**Tip:** If booting fails, ensure Secure Boot is disabled in BIOS/UEFI settings.

---

## Method 1: Reset Password Using Lazesoft Password Recovery ⭐ (Recommended for Beginners)
This tool automates the reset process with a simple GUI.

### Step 4: Launch Lazesoft Password Recovery
1. On the Hiren's desktop, click the **Start Menu** (bottom-left corner).
2. Navigate to: **All Programs** > **Security** > **Passwords**.
3. Select **Lazesoft Password Recovery** (highlighted in **Screenshot 2**).

### Step 5: Perform the Reset
1. In Lazesoft, select **Reset / Unlock Local Password**.
2. Choose your Windows installation (usually the first option).
3. Select the user account (e.g., **Administrator** or your username).
4. Click the large blue **RESET / UNLOCK** button (**Screenshot 3** shows this screen).
5. Wait for the process to complete (1–2 minutes).
6. Click **Finish** to exit.

### Step 6: Reboot and Test
1. Close Lazesoft and all programs.
2. Remove the USB and restart the PC.
3. At the Windows login screen, the account should now allow entry without a password (or with a blank one).

**Summary of Method 1:**
1. Download Hiren’s BootCD PE x64.
2. Create bootable USB with Rufus.
3. Boot from USB.
4. Open **Passwords** > **Lazesoft Password Recovery**.
5. Select account > **Reset/Unlock**.
6. Restart > Login without password.

---

## Method 2: Reset Password Using NT Password Edit 🔵 (For Manual Control)
Use this if Lazesoft fails or for precise edits (e.g., setting a custom password). It directly modifies the Windows SAM (Security Accounts Manager) file.

### Step 4: Launch NT Password Edit
1. On the Hiren's desktop, click the **Start Menu**.
2. Navigate to: **All Programs** > **Security** > **Passwords**.
3. Select **NT Password Edit**.

### Step 5: Fix Missing C: Drive (Common Issue)
Hiren's PE may not auto-mount your Windows drive as C:. Assign it manually:
1. Right-click **This PC** on the desktop > **Manage**.
2. In the left panel, select **Disk Management**.
3. Locate your Windows partition (NTFS format, often labeled "Windows" or ~100+ GB—see **Screenshot 4**).
4. Right-click the partition > **Change Drive Letter and Paths**.
5. Click **Add** > Select **Assign the following drive letter: C**.
6. Click **OK**. The C: drive should now appear in File Explorer.

### Step 6: Load the SAM File
1. In NT Password Edit, click the **Browse (...)** button.
2. Navigate to: **C:\Windows\System32\config\SAM** (**Screenshot 5** shows this path).
3. Click **Open**. A list of user accounts will load.

### Step 7: Edit the Account
1. Select the user account from the list (e.g., **Administrator**).
2. **Do not** attempt to crack/view the password—this tool only edits it.

   **Options:**
   - **Clear Password**: Click **Clear Password** (removes it entirely).
   - **Set New Password**: Enter your desired password in the field > Click **Set Password**.

### Step 8: Save and Reboot
1. Click **Save Changes** (confirm if prompted).
2. Close NT Password Edit.
3. Remove the USB and restart the PC.

### Step 9: Login to Windows
- If cleared: Press **Enter** at the blank password field.
- If set: Use the new password.

**Summary of Method 2:**
1. Boot Hiren’s BootCD PE.
2. Assign "C:" drive letter in Disk Management.
3. Open **NT Password Edit**.
4. Load **C:\Windows\System32\config\SAM**.
5. Select user > Clear or set password.
6. Save changes > Reboot.

---

## Troubleshooting Tips
- **Boot Issues:** Update BIOS/UEFI or try a different USB port.
- **BitLocker Error:** You'll need the recovery key—PE can't bypass encryption.
- **No Changes After Reset:** Run `chntpw -l C:\Windows\System32\config\SAM` in Command Prompt for verification.
- **UEFI vs. Legacy:** Mismatch in Rufus settings can prevent booting—toggle in BIOS.

