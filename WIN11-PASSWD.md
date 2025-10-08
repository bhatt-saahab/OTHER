 
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
```
