# windows-ad-home-lab
#Windows Server Active Directory Home Lab

##Overview
A virtualized enterprise network lab built on Hyper-V to simulate Active Directory domain administration, DNS configuration, Group Policy Management, and Tier 1/Tier 2 endpoint troubleshooting.

##Environment & Hardware
Host Machine: HP EliteBook (Windows Pro, 16GB RAM)
Hypervisor: Microsoft Hyper-V 
Virtual Network: 'Lab-Private-Net' (Isolated Hyper-V Private Switch)


##Domain Controller ('DC01') Configuration
OS: Windows Server 2025 Standard Evaluation (Desktop Experience)
VM specs: Generation 2, 4 GB Dynamic RAM, 40 GB VHDX
Network Settings
IP Address: '192.168.10.10'
Subnet Mask: '255.255.255.0'
Preferred DNS: '192.168.10.10'
Roles & Domain Setup:
Installed Active Directory Domain Services (AD DS) and DNS Server roles
Promoted 'DC01' to Primary Domain Controller for a new forest/domain: 'corp.local'(NetBIOS:'CORP')

##Client Workstation ('CLIENT01') Configuration
OS: Windows 11 Enterprise Evaluation
VM Specs: Generation 2, 4 GB Dynamic RAM, 40 GB VHDX, Virtual TPM enabled 
Network Settings:
IP Address: 192.268.10.20
Subnet Mask: 255.255.255.0
Preferred DNS: 192.168.10.10 ('DC01')
Domain Join & Troubleshooting Note: 
Joined 'CLIENT01' to the 'corp.local' domain and verified domain membership via 'sysdm.cpl' ('CLIENT01.corp.local').
Issue Encountered: Simultaneously renaming the host and joining the domain triggered a primary DNS suffix registration warning prior to reboot, and signing in without the 'CORP\' prefix defaulted authentication to the local 'CLIENT01' SAM database. 
Resolution: Verified domain join post-reboot and authenticated using explicit 'CORP\username' syntax.

##Active Directory Structure & User Provisioning 
Created departmental Organizational Units (OUs) in Active Directory Users and Computers (ADUC): 'Sales' and 'IT'
Provisioned test domain user account 'Jane Doe' ('CORP\jdoe') inside the 'Sales' OU

##Group Policy Management (GPO) & Verification
GPO Created: 'Block Control Panel-Sales' linked directly to the 'Sales' OU. 
Policy Setting: Enabled 'User Configuration > Policies > Administrative Templates > Control Panel > Prohibit access to Control Panel and PC Settings'. 
Verification: Logged into "CLIENT01" as 'CORP\jdoe' and confirmed Windows Settings and Control Panel access were blocked by domain restriction policy.

###TIcket #1: Account Access & Password Reset
Symptom: User 'CORP\jdoe' unable to authenticate on 'CLIENT01' due to disabled account status/expired credentials.
Troubleshooting & Resolution: Located user object in 'ADUC > Sales OU', renabled account, issued temp password reset with "User must change password at next login" enforced, and verified successful self-service password update at the 'CLIENT01' login prompt. 

###Ticket #2: Domain DNS Resolution Failure
Symptom: Client workstation unable to locate the 'corp.local' domain controller for authentication and policy updates. 
Diagnosis: Used 'ipconfig /all' and 'nslookup corp.local' on 'CLIENT01' to identify that the adapater's Preferred DNS was pointed to an external resolver ('8.8.8.8') instead of the internal AD/DNS server.
Resolution: Reconfigured Primary DNS back to '192.168.10.10' ('DC01'), cleared the local resolver cache via 'ipconfig /flushdns', and verified successful 'corp.local' resolution via 'nslookup'.
