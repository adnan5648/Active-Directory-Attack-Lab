# Network Configuration

<img width="721" height="487" alt="image" src="https://github.com/user-attachments/assets/da2b8acb-bdfb-46a9-87c4-a0fcc7e6699a" />

The workstation network adapter is configured with the static IP address 10.0.0.20/24. The preferred DNS server is set to 10.0.0.10, which points WS01 to the Domain Controller for Active Directory DNS resolution.

# Join WS01 to the Active Directory Domai 

<img width="1012" height="95" alt="image" src="https://github.com/user-attachments/assets/5f8d07bd-2e05-4f80-974c-9336baf4a7ee" />

The command firstly prompts for ADLAB\Administrator credentials. The Add-Computer command then joins the workstation to the adlab.local domain, places it inside the Workstations OU, renames it to WS01, and restarts the system.

# Enable PowerShell Remoting and Grant Alice Remote Access 

<img width="932" height="266" alt="image" src="https://github.com/user-attachments/assets/5c49009a-bef6-4057-ba1c-a3641aa91ca5" />

Enable-PSRemoting -Force enables WinRM and configures the required firewall rules on WS01. The ADLAB\alice account is then added to the local Remote Management Users group, allowing Alice to establish permitted remote management sessions to the workstation.

# Reset and Enable Alice’s Domain Account 

<img width="1126" height="73" alt="image" src="https://github.com/user-attachments/assets/24178fe3-e53a-45cb-b897-f3eb4a282c93" />

The Set-ADAccountPassword command resets the password for the alice domain account using a new lab password. The Enable-ADAccount command then ensures that the account is enabled and ready for authentication.

# Verify Alice’s Remote Management Membership 

<img width="663" height="115" alt="image" src="https://github.com/user-attachments/assets/16f92035-4a7e-4428-a67a-37db2d41d7c8" />

The command checks the local Remote Management Users group on WS01. The output confirms that ADLAB\alice is successfully added as a member and can use permitted remote management services.

# Allow ICMPv4 Ping Through Windows Firewall 

<img width="1013" height="351" alt="image" src="https://github.com/user-attachments/assets/48911fe2-04db-4d54-9d4f-feb3f876bd7f" />

The command creates a new inbound Windows Firewall rule named Allow ICMPv4 Ping that permits ICMPv4 Echo Request traffic. This allows the machine to respond to standard IPv4 ping requests from other systems in the lab network.
