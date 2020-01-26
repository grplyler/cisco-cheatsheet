# Cisco Utils

**Warning, use at your own risk. I created these scripts with an educational mindset while studying my CCNA**

Bootstrapping and hardening scripts for Cisco routers and Switches

## Usage

1. Clone the repo: 

    ```
    git clone https://github.com/grplyler/cisco-utils
    ```
    
2. Install python requirements (for ftp server):

    ```
    pip install -r requirements.txt
    ```
    
3. Run python ftp_server.py

    ```
    python3 ftp_server.py
    ```
    
4. Pull a script onto a network device (WARNING: Backup to avoid any losses)

    ```
    Switch#> copy ftp://192.168.1.10/sw_base.txt running-config
    ```
    
    *Replace 192.168.1.10 with the IP of the computer connected to the switch or router.*
