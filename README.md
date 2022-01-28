![This is an image](https://raw.githubusercontent.com/FrankM77/docker-fail2ban/master/Fail2ban_logo.png)
# docker-fail2ban

Useful Links:

https://hub.docker.com/r/crazymax/fail2ban

http://www.fail2ban.org/wiki/index.php/Main_Page

https://forums.unraid.net/topic/119186-fail2ban-setup-with-nginx-and-cloudflare-tunnels/

Icon URL : https://raw.githubusercontent.com/FrankM77/docker-fail2ban/master/Fail2ban_logo.png

Fail2ban Docker image based on Alpine Linux for use with UnRaid OS

##Steps:

1. Download the container from the UnRaid Community Apps store
2. In Unraid template create a path to your container's log file that you want fail2ban to monitor
3. Start fail2ban and check logs to make sure server is ready



You must add a path to your container log file that you want fail2ban to monitor. 

**Important**: If you are using Nginx Proxy Manger with Cloudflare tunnels then you need to pass the client IP to your container that you want fail2ban to protect.  For instance I have fail2ban protecting my Jellyfin container but inititally in the Jellyfin logfiles it would log the IP address as 172.18.0.1, so in order to log the real connecting client IP you need to go into:

Nginx Proxy manager--->proxy host---->Advanced------>Custom Nginx Configuration add "real_ip_header CF-Connecting-IP;"  without the quotes. 
You will then notice that your container log (in my case jellyfin) will have the real ip of the connecting user/client.  

**PROBLEM**: fail2ban is running and according to its logs it is banning the external IP address, but you are still able to access the container website

Solution: Setup API to modify Cloudflare Firewall Tools reference: https://guides.wp-bullet.com/integrate-fail2ban-cloudflare-api-v4-guide/

-Step 1:
Add cloudflare configuration file in action.d/cloudflare-apiv4.conf
You can find a copy of my working file above in the master branch or using this link : https://github.com/FrankM77/docker-fail2ban/blob/master/cloudflare-apiv4.conf
Go to the bottom of the file and change the cfuser and cftoken to your own.

This will create firewall rule on cloudflare. I tested the ban and unban action by manually typing the CURL commands and verifying on cloudflare that it made the changes. I recommend doing the same.  

Here are the CURL commands for testing:

**BAN** (replace cfuser and cftoken with your credentials. Replace <IP> with the IP you want banned. eg.. "value":"1.2.3.4"
```
curl -s -X POST "https://api.cloudflare.com/client/v4/user/firewall/access_rules/rules" \
            -H "X-Auth-Email: <cfuser>" \
            -H "X-Auth-Key: <cftoken>" \
            -H "Content-Type: application/json" \
            --data '{"mode":"block","configuration":{"target":"ip","value":"<ip>"},"notes":"Fail2ban <name>"}'
```
  
  **UNBAN** (replace value=<IP> with IP you want unbanned, eg. value=1.2.3.4
  ```
  curl -s -X DELETE "https://api.cloudflare.com/client/v4/user/firewall/access_rules/rules/$( \
              curl -s -X GET "https://api.cloudflare.com/client/v4/user/firewall/access_rules/rules?mode=block&configuration_target=ip&configuration_value=<ip>&page=1&per_page=1&match=all" \
             -H "X-Auth-Email: <cfuser>" \
             -H "X-Auth-Key: <cftoken>" \
             -H "Content-Type: application/json" | awk -F"[,:}]" '{for(i=1;i<=NF;i++){if($i~/'id'\042/){print $(i+1);}}}' | tr -d '"' | sed -e 's/^[ \t]*//' | head -n 1)" \
             -H "X-Auth-Email: <cfuser>" \
             -H "X-Auth-Key: <cftoken>" \
             -H "Content-Type: application/json"
```
                                                                                            
-Step 2 Add cloudflare config action in jail.d/'your_container'.local. Name the action cloudflare-apiv4 (do not include the .conf file extension)
(example) action = iptables-allports[name='your_container', chain=DOCKER-USER]  
                   cloudflare-apiv4
                              
Make sure you use the same name in both places for 'your-container'
For example: 
Add cloudflare config action in jail.d/vaultwarden.local. Name the action cloudflare-apiv4 (do not include the .conf file extension)
(example) action = iptables-allports[name=vaultwarden, chain=DOCKER-USER]  
                   cloudflare-apiv4 

                                                                                            
**Setup Vaultwarden Jail**
*Vaultwarden jail.d/vaultwarden.local*
                                                                                            
```
# path_f2b/jail.d/vaultwarden.local

[vaultwarden]
enabled = true
port = 80,443,8081
filter = vaultwarden
#banaction = %(banaction_allports)s
action = iptables-allports[name=vaultwarden, chain=DOCKER-USER]
         cloudflare-apiv4
#Path to vaultwarden log inside the container
logpath = /vaultwarden/vaultwarden.log
maxretry = 2
bantime = 300
findtime = 300
bantime.increment = true
```  
                                                                                            
**Setup Vaulwarden Filter**
*Vaultwarden filter.d/vaultwarden.local*
                                                                                            
```
                                                                                            
# path_f2b/filter.d/vaultwarden.local

[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$
ignoreregex =
```                                                                                            


