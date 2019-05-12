# Pi-hole with DNS-Over-TLS/HTTPS

Starting from scratch, or wherever you may be at, the following steps must be completed
# Google Cloud Platform
Ensure you have the following firewall rules
![Firewall Rules](img/dp_fw.png)

1. Install Pi-hole Normally
2. Install the dependencies
    - Certbot (If you DO NOT HAVE a valid SSL Certificate)
    - dnsproxy
3. Generate a certificate
    - Ensure you have an A or CNAME record pointing your hostname to the IP address of your server
4. Create the start script and service

# Installing Pi-hole
I'm sure most of you are familiar to this process, but if you are not, all you have to do is run the command below:
```
curl -sSL https://install.pi-hole.net | bash
```

# Installing the dependencies
- Assuming the current user is root (`sudo -i`)
- The operating system I am using is Debian 9. If this does not apply to you, you may have to look up the proper command(s) for your operating system and package manager. 

1. Installing Certbot
    ```
    apt-get update

    apt-get install certbot -t stretch-backports
    ```
2. Installing dnsproxy
- At the time of writing, the latest version of dnsproxy was v0.13.0. 
    ```
    cd /tmp

    wget https://github.com/AdguardTeam/dnsproxy/releases/download/v0.13.0/dnsproxy-linux-amd64-v0.13.0.tar.gz

    tar -xvf dnsproxy*
    
    mv linux-amd64/dnsproxy /usr/local/bin/

    rm -rf *dnsproxy* linux-amd64
    
    cd -
    ```

# Generating an SSL Certificate
- For dnsproxy to work, you will need a vaild SSL certificate. To generate a FREE SSL Certificate, use the following command
    ```
    certbot certonly --pre-hook "service lighttpd stop" --post-hook "service lighttpd start" --standalone -d <HOSTNAME>
    ```
## Configuring Auto Renew
Follow this tutorial https://www.onepagezen.com/letsencrypt-auto-renew-certbot-apache/
I'm not going to go into the details, EXCEPT in your crontab, include the following: 
```
45 2 * * 6 cd /etc/letsencrypt/ && ./certbot-auto renew && /usr/sbin/service dnsproxy restart
```

# Creating the start script and service
## Creating the start script
Enter the following commands below:
```
mkdir /opt/dnsproxy; cd /opt/dnsproxy
```
Using a text editor of your choice, add the following to `start.sh`:
```
#!/bin/bash
HOSTNAME="<YOUR HOSTNAME>"

CERTSPATH=/etc/letsencrypt/live/$HOSTNAME
CERT=$CERTSPATH/cert.pem
PKEY=$CERTSPATH/privkey.pem

DNSPROXY_OPTS="--https-port=443 --tls-port=853 --tls-crt=$CERT --tls-key=$PKEY -u 127.0.0.1:53 -p 0"

# Start the program
/usr/local/bin/dnsproxy $DNSPROXY_OPTS
```

Give the script proper execute permissions
```
chmod a+x /opt/dnsproxy/start.sh
```

## Creating the service 
Using a text editor of your choice, we are going to create the service under the filename `/lib/systemd/system/dnsproxy.service`
```
[Unit]
Description=AdguardTeam/dnsproxy service
After=syslog.target network-online.target

[Service]
Type=simple
User=root
ExecStart=/opt/dnsproxy/start.sh
ExecStop=/usr/bin/pkill dnsproxy
Restart=on-failure
RestartSec=10
KillMode=process

[Install]
WantedBy=multi-user.target
```
1. Enable the service to start on startup
    ```
    sudo systemctl enable dnsproxy
    ```
2. Start the dnsproxy service
    ```
    sudo service dnsproxy start
    ```
3. Test it out
    - dnstls (npm)
    - Firefox

# Credits
Matt Zabojnik
- Had no idea how to use his Pi-hole with Android Pie, neither did I
- Gave me the initiative to try this out and create this tutorial. This would not have happened without him!

Me (Jayke Peters)
- 16 year old
- Been doing stuff like this for 2 years
- Learned python... Some C, some golang, some java lol
