**AD Attack Lab Architecture**

All systems are connected through an isolated virtual network.
The lab environment is isolated from the main network by placing all virtual machines on a dedicated LAN segment using the 10.0.0.0/24 subnet.
This allows DC01, WS01, and Kali to communicate with each other while preventing the intentionally vulnerable systems from interacting with the host's production network or external devices.


<img width="1536" height="1024" alt="AD Lab Arch" src="https://github.com/user-attachments/assets/1c867fb4-8743-4bc2-8351-3370fa091519" />
