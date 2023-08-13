# SIEMxSentinel

This is a lab where I configured Azure Sentinel (Microsoft's Cloud SIEM) workbook to display global RDP brute force attempts on world map.

## 1.Create the honeypot (VM on Azure):
- the first step is to set a VULNERABLE VM on the cloud and make it as exposed as possible to the internet.
This can be done :
* by configuring the NSG (Network Security Group) rules to accept ALL inbound traffic as shown below
![NSG honeypot1](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/545451e1-85ea-4e4b-a8b1-5e4e29e5e9db)
![NSG honeypot2](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/d8bf3947-51eb-445b-ab5b-aef96f4eadb3)

* disabling the firewall on my Windows VM, thus making it "pingable"
![tried pinging it but no reply](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/067dfc8e-3590-4319-85df-cd524e26db06)
![turn off firewall to ping](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/e2730f5d-46b3-4d16-83fa-fb09202024ba)

once the VM is deployed, I can access it via RDP protocol using freerdp - this is a simple solution since the typical rdesktop command raised some NLA (Network Level Authentification) issues. These could be bypassed by implementing a bastion and enabling Kerberos authentification but that's just overkill!
  
## 2.Retrieve the log data in Azure LAW (log analytics workspace)
The goal now is to retrieve the failed login attempts from the Event Viewer security logs.
![failed login in event viewer](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/5a7a1ef8-a3c7-4a09-9da1-da5376e96c41)
![attacker IP address in event viewer](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/c61d4e98-9185-4c62-a43a-ba55c6d8af6b)
I'll use a public API to display the geolocation info related the IP address. A Powershell script will pass the logs data to the API and store the output in a new log file - Credit to Josh Madakor. thank you!!
here's the PS script running and receiving log data from Event Viewer
![running the PS script on the VM](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/750e3784-eff8-4e5c-b5c0-42a7aa4e39f5)

after creating a LAW instance and linking it to my VM, 
![LAW](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/f2021d22-91ec-4c5a-8521-225e3608333a)

I'll add a new MMA-based log file to the workspace. This will be deprecated starting from 31/08/2023. A workaround would be to set an endpoint and retrieve the data using the Azure Monitor Agent (AMA) by defining data collection rules.
Though the law can recieve log data from the VM,
![LAW receiving log data](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/5c146480-7a49-48fc-bc9c-8d277b7ec970)


## 3. Setup the workbook and show data
the log displays all info as RawData. Since custom fields are no longer available, i simply parsed the rawdata field :
![log data parsing](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/d26bd498-7745-409c-aa79-26e377a8e569)
and grouped it to view the failed rdp login attempts by geolocation data (latitude and longitude) on a world map.

here are the results 30mn after deploying the VM
![15mn after setting the honeypot](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/862037bb-d537-4c57-ae7c-ca85386bdc53)

6 hours after deploying the VM
![6 hours after exposure :O](https://github.com/tesnim5hamdouni/SIEMxSentinel/assets/121170828/f1614559-2e69-445f-bffc-3114276d5af7)



