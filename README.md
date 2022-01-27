# docker-fail2ban
Fail2ban Docker image based on Alpine Linux for use with UnRaid OS

Download the container from the UnRaid Community Apps store

Icon URL : https://raw.githubusercontent.com/FrankM77/docker-fail2ban/master/Fail2ban_logo.png

You must add a path to your container log file that you want fail2ban to monitor. 

Important: If you are using Nginx Proxy Manger with Cloudflare tunnels then you need to pass the client IP to your container that you want fail2ban to protect.  For instance I have fail2ban protecting my Jellyfin container but inititally in the Jellyfin logfiles it would log the IP address as 172.18.0.1, so in order to log the real connecting client IP you need to go into:

Nginx Proxy manager--->proxy host---->Advanced------>Under custom configuration add "real_ip_header CF-Connecting-IP;"  without the quotes. 
You will then notice that your container log (in my case jellyfin) will have the real ip of the connecting user/client.  

PROBLEM: fail2ban is running and according to its logs it is banning the external IP address, but you are still able to access the container website

Solution: Setup API to modify Cloudflare Firewall Tools reference: https://guides.wp-bullet.com/integrate-fail2ban-cloudflare-api-v4-guide/

Add cloudflare configuration file in action.d/cloudflare-apiv4.conf
Add cloudflare config action in jail.d/'your_container'.local. Name the action cloudflare-apiv4 (do not include the .conf file extension)
(example) action = iptables-allports[name='your_container', chain=DOCKER-USER]  
                              cloudflare-apiv4
                              
Make sure you use the same name in both places for 'your-container'
For example: 
Add cloudflare config action in jail.d/vaultwarden.local. Name the action cloudflare-apiv4 (do not include the .conf file extension)
(example) action = iptables-allports[name=vaultwarden, chain=DOCKER-USER]  
                              cloudflare-apiv4



