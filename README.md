# SOC Analyst LimaCharlie EDR Lab

## Objective
The scope of this project was to create a home lab of virtual machines. The Kali-Linux VM as the attacker and the WindowVM as the victim. I used LimaCharlie EDR and Sysmon to work as my threat detection engine as well as track incoming telemetry and for logging. I used Sliver to generate C2 payloads, implant harmful files and operate the host machine in the background.

### Skills Learned

- Completely disable Defender on Windows
- Set up LimaCharlie EDR and Sysmon
- Silver to implant and activate .exe files
- Build D&R rules based on events
- Block attacks with custom rules

### Tools Used

- Virtualbox/ KaliLinux2024/ Windows 10
- LimaCharlie EDR/ Sliver Server/ Sysmon
<br/>
<br/>

## Setting up Attacker and Victim Virtual Machine
![Screenshot 2025-02-09 175701](https://github.com/user-attachments/assets/98966ca1-270b-4b29-bf0d-bb2c5d2d3c7e)

## Disabling Windows Defender
| Windows Security| Group Policy Editor| Admin CMD| Registry Editor|
|----------------------------|----------------------------|----------------------------|----------------------------|
|![Screenshot 2025-02-09 183735](https://github.com/user-attachments/assets/87efc3d6-fd62-4b42-bd77-51ef242af75f)|![Screenshot 2025-02-09 185319](https://github.com/user-attachments/assets/33c59b24-0bd8-4b7c-b68b-590bfc5b7e25)|![Screenshot 2025-02-09 185653](https://github.com/user-attachments/assets/779e2724-e792-4aa0-be0a-767e597c3337)|![Screenshot 2025-02-09 190849](https://github.com/user-attachments/assets/cfad9518-6daa-4633-bfc7-d09dc39280ab)|

<br/>

- Disable Tamper Protection/ All Virus & threat protection settings
- Permanently Disable Defender via Group Policy Editor (Computer Configuration > Administrative Templates > Windows Components > Microsoft Defender Antivirus)
- Safeboot then disabled services in the Registry (Services\Sense,WdBoot,WinDefend,WdNisDrv,WdNisSvc,WdFilter)
  
<br/>

## Installing Sysmon in Windows VM
| Sysmon Install| LimaCharlie Install| LimaCharlie/Sysmon Event Log config|
|----------------------------|----------------------------|----------------------------|
|![Screenshot 2025-02-09 192458](https://github.com/user-attachments/assets/178f0b7e-e9bf-4274-b2c6-b6baef234a32)|![Screenshot 2025-02-09 201938](https://github.com/user-attachments/assets/07459ea8-46c0-48a1-b455-b8034452843e)|![Screenshot 2025-02-09 202231](https://github.com/user-attachments/assets/3be69ab2-dd0f-42f1-b0b2-88b7e033aacb)|

- Created a LimaCharlie organization and added a sensor for Windows
- Configured Lima Charlie to ship the Sysmon event log alongside its own EDR telemetry

## Setting up Attack VM
