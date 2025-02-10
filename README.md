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

- Disable Tamper Protection/ All Virus & threat protection settings.
- Permanently Disable Defender via Group Policy Editor (Computer Configuration > Administrative Templates > Windows Components > Microsoft Defender Antivirus).
- Safeboot then disabled services in the Registry (Services\Sense,WdBoot,WinDefend,WdNisDrv,WdNisSvc,WdFilter).
  
<br/>

## Installing Sysmon in Windows VM
| Sysmon Install| LimaCharlie Install| LimaCharlie/Sysmon Event Log config|
|----------------------------|----------------------------|----------------------------|
|![Screenshot 2025-02-09 192458](https://github.com/user-attachments/assets/178f0b7e-e9bf-4274-b2c6-b6baef234a32)|![Screenshot 2025-02-09 201938](https://github.com/user-attachments/assets/07459ea8-46c0-48a1-b455-b8034452843e)|![Screenshot 2025-02-09 202231](https://github.com/user-attachments/assets/3be69ab2-dd0f-42f1-b0b2-88b7e033aacb)|

- Created a LimaCharlie organization and added a sensor for Windows.
- Configured Lima Charlie to ship the Sysmon event log alongside its own EDR telemetry.

## Setting up Attack VM
![Screenshot 2025-02-09 212656](https://github.com/user-attachments/assets/74e6fcb5-6809-48ea-ab80-ada9db5377b1)
- Installed Sliver and created a workign directory

## Attack setup against Windows VM
| Create C2 playload| Implant file transfer| Verify active implant session|
|----------------------------|----------------------------|----------------------------|
|![Screenshot 2025-02-09 215346](https://github.com/user-attachments/assets/46114902-5277-4643-adc5-5e72c342a275)|![Screenshot 2025-02-09 220319](https://github.com/user-attachments/assets/17f9cc92-24e0-4e02-8be1-cf3fe13d9846)|![Screenshot 2025-02-09 221551](https://github.com/user-attachments/assets/1b2db58d-cd5a-45f1-b78f-732d2ecbf15b)|

- Generate my C2 session payload using my Attacking VM's IP (generate --http [Linux_VM_IP] --save /opt/sliver).
- I confirm the implant config and creat a tempo web server to pull the payload down to the victim machine.
- I verify the session ID and log the session ID

## Sliver Session
![Screenshot 2025-02-09 221928](https://github.com/user-attachments/assets/e3ee6303-8b53-4b3d-9238-d6345c26a974)
- Grab basic info about the session abd that privileges my implant has (info, whoami, getprevis)

## Observing EDR Telemetry
| Processes| Network| File System|
|----------------------------|----------------------------|----------------------------|
|![Screenshot 2025-02-09 222419](https://github.com/user-attachments/assets/99999140-f7ca-4ac1-a720-1146d76d6a9b)|![Screenshot 2025-02-09 223048](https://github.com/user-attachments/assets/5ba25a3e-65b2-49e5-b128-5189639e7540)|![Screenshot 2025-02-09 223757](https://github.com/user-attachments/assets/5250f12b-6547-4b7b-8909-7d30ba50655e)|

- Checking Processes in LimaCharlie we can see the attackers C2 implant. Selecting the "FAIR_ANALOG.exe" we notice that this process doesn't carry a valid signature as it is not signed. We are also able to identify the destination IP this process is communicating with.
- We can also search the IP under Network and see that "FAIR_ANALOG.exe" is returned in the network list.
- Under File System we can check the location that the implant is running and grab the hash.

## OSINT
![Screenshot 2025-02-09 224112](https://github.com/user-attachments/assets/1de2ba44-d8a5-4eed-8d9d-535d8bbad41e)
- Inspecting the "FAIR_ANALOG.exe" file hash on Echotrail and VirusTotal comes up with zero results.
  - This does not mean that the file is not malicious. It's just not common/known in their query for the hash of the EXE. Since the payload was created by use it's not likely to appear on any OSINT tools.

## Create Detection and Response rule
| Filter Timeline Event| D&R Rule| Test Event| Detection Alert|
|----------------------------|----------------------------|----------------------------|----------------------------|
|![Screenshot 2025-02-09 224334](https://github.com/user-attachments/assets/eab8d6ad-02ab-43fd-bb19-d5640e0f33c6)|![Screenshot 2025-02-09 233939](https://github.com/user-attachments/assets/9cba0600-ff80-4a94-8f6d-9f27853f70fa)|![Screenshot 2025-02-09 234039](https://github.com/user-attachments/assets/48a302e7-a2c1-474a-b712-3262993315f9)|![Screenshot 2025-02-09 234648](https://github.com/user-attachments/assets/8afeef80-0ab1-4a13-9922-2ed93db557f6)|

- Ran the command "procdump -n lsass.exe -s lsass.dmp" since "lsass.exe" would generate events for the EDR.
- I filtered out the timeline for "FAIR_ANALOG" and found the specific event I wanted to create a D&R for.
- When it detects SENSITIVE_PROCESS_ACCESS that matches the Target Event it will generate a detection report.
- I reran the "procdump -n lsass.exe -s lsass.dmp" and now we can see tha alert in the Detection tab of LimaCharlie.

## Block attacks with D&R
![Screenshot 2025-02-10 142440](https://github.com/user-attachments/assets/37d1eeae-f430-4aa3-abda-caf4f8cf1c0f)

- Uses Silver C2 enter the victims shell
- Run "vssadmin delete shadows /all" command which can stimulate a typical ransomware attack pattern.
- The ran "whoami" to verify I still had an active system shell.

| Detection| Metadata| Event Timeline|
|----------------------------|----------------------------|----------------------------|
|![Screenshot 2025-02-10 142613](https://github.com/user-attachments/assets/bbe22668-d338-4e60-bbce-25f507beb309)|![Screenshot 2025-02-10 142813](https://github.com/user-attachments/assets/cab19567-c866-4216-b718-8cca619c7f50)|![Screenshot 2025-02-10 143201](https://github.com/user-attachments/assets/15a900de-d850-462d-a7fe-b7be41136a2b)|

- The default Sigma rules detect the action.
- When inspecting the alert the Sigma rules contain references  to understand the detection and why it exists.
- Inspected the alert on the timeline and created D&R based on the event.
