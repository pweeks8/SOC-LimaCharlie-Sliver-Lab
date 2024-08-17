# SOC LimaCharlie Sliver Lab
Credit goes to Eric Capuano for writing ["So you want to be a SOC Analyst?"](https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-intro) and putting together this lab!
### Learning Objective
- Hands-on experience with EDR (Endpoint Detection and Response) and C2s (Command and Control)

### Tools & Requirements
1. VirtualBox or VMWare
2. Windows VM
3. Linux VM
4. Sysmon
5. LimaCharlie
6. Sliver

## Step 1: Set up a Windows/Linux Server VM
1. Install VirtualBox
2. Download and deploy a [Windows VM](https://developer.microsoft.com/en-us/windows/downloads/virtual-machines/)
3. Download and deploy a [Linux Server VM](https://releases.ubuntu.com/22.04.1/ubuntu-22.04.1-live-server-amd64.iso).

## Step 2: Disabling Windows Defender
1. Disable Tamper Protection
![Tamper Protection Disabled](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fda8a8b73-5aeb-4fef-a93a-36ebe5e3e4a3_390x226.png)
2. Disable Defender via Group Policy Editor 
 
![gpedit](https://i.imgur.com/ft4OAGz.png)

## Step 3: Installing Sysmon (Windows VM)
1. Download and install Sysmon to provide granular telemetry on Windows Endpoints.
2. Download and install [SwiftOnSecurity](https://infosec.exchange/@SwiftOnSecurity) Sysmon config
3. Validate Sysmon is running
![sysmon](https://i.imgur.com/gIDLhzw.png)
## Step 4: Installing LimaCharlie (Windows VM)
1. Create a LimaCharlie account
2. Install LimaCharlie on the Windows VM
  ![limacharlie](https://substackcdn.com/image/fetch/w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff821e410-d4d3-4161-a426-9d8ff348806c_610x392.png)
3. Add a rule to allow LimaCharlie to receive the Sysmon event table

## Step 5: Installing/Creating a payload with Sliver C2 (Linux VM)
1. Download and install Sliver
2. Create a directory for sliver: /opt/sliver
3. Generate a C2 session payload using sliver in /opt/sliver by running ``sliver`` and confirm the new implant configuration
![sliver](https://i.imgur.com/VXa2ZwP.png)

## Step 6: Python Server
1. Start a Python server with the command ``python3 -m http.server 80`` to transfer over the payload generated by Sliver to the Windows VM

## Step 7: Start a Command and Control Session
1. Start an Sliver HTTP listener by running the commands ``sliver-server`` and ``http`` while in Sliver
2. Execute the C2 payload on the Windows VM

## Step 8: Observe the EDR(LimaCharlie) Telemetry
1. Conduct hash analysis using VirusTotal
![hash analysis](https://i.imgur.com/Vx9d4dI.png)
This virus did not show up in VirusTotal because VT has never seen the file! Eric Capuano states "This actually makes a file even more suspicious because nearly everything has been seen by VirusTotal".

## Step 9: Stealing credentials with Sliver
1. In Sliver (still connected to the http listener session) run the command ``procdump -n lsass.exe -s lsass.dmp``
![procdump](https://i.imgur.com/PO1nz69.png)

## Step 10: Detecting the stolen creds with LimaCharlie
1. Look at the timeline of the Windows VM sensor and filter for "SENSITIVE_PROCESS_ACCESS". This will show the event where lsass was accessed.
![Alt text](https://i.imgur.com/2fRg32o.png)
![lsassevent](https://i.imgur.com/gwVpgdS.png)
2. Create a D&R(Detection & Response) rule that will alert anytime this event occurs. This rule specifies that the detection will only look at SENSITIVE_PROCESS_ACCESS where the process ends with lsass.exe. The response section generates a detection report with the name LSASS access.
![d&rbutton](https://i.imgur.com/QBXZeeC.png)
![d&rrule](https://i.imgur.com/HtJu3e0.png)
3. Test the new rul LSASS rule
![Alt text](https://i.imgur.com/wvo3q8d.png)
4. Save the rule as LSASS Accessed
![Alt text](https://i.imgur.com/BebmYh7.png)

## Step 11: Detect LSASS Accessed
1. Run the procdump command again
2. Look in the "Detections" tab of LimaCharlie. As you can see, our new rule worked, and the event is captured!
![Alt text](https://i.imgur.com/0Exlnax.png)

## Step 12: Perform a ransomware attack! (Almost)
1. As Eric Capuano states in his post "Volume Shadow Copies provide a convenient way to restore individual files or even an entire file system to a previous state which makes it a very attractive option for recovering from a ransomware attack". So as an attacker, we are deleting the copies so there is no way to recover from the ransomware attack.
2. Run the ``shell`` command, run the ``vssadmin delete shadows /all`` command and run ``whoami``
![Alt text](https://i.imgur.com/Km7lU2T.png)


## Step 13 Detect and Block the Attack
1. Look in the Detections tab of LimaCharlie
![Alt text](https://i.imgur.com/JZ4pBTg.png)
2. Make a new D&R rule for Shadow Copies Deletion. The action:report tells LimaCharlie to create a Detection report and the action:task is what will be used to block the attack by killing the parent process of the `vssadmin delete shadows /all` command. Run the `vssadmin delete shadows /all` command again and run `whoami`. Whoami didnt return anything because the D&R rule worked successfully. The rule terminated the parent process of `vssadmin delete shadows /all` making the shell hang and `whoami` not returning anything.


That's all folks!













