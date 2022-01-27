# docker-fail2ban
Fail2ban Docker image based on Alpine Linux for use with UnRaid OS

Download the container from the UnRaid Community Apps store

Icon URL : https://raw.githubusercontent.com/FrankM77/docker-fail2ban/master/Fail2ban_logo.png

You must add a path to your container log file that you want fail2ban to monitor. 

Important: If you are using Nginx Proxy Manger with Cloudflare tunnels then you need to pass the client IP to your container that you want fail2ban to protect.  For instance I have fail2ban protecting my Jellyfin container but inititally in the Jellyfin logfiles it would log the IP address as 172.18.0.1, so in order to log the real connecting client IP you need to go into Nginx Proxy manager--->proxy host---->Advanced------>Under custom configuration add "real ip header CF-Connecting-IP"  without the quotes. You will then notice that your container log (in my case jellyfin) will have the real ip of the connecting user/client.  


