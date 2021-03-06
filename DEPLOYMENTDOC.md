# Production Deployment

![Tamarabot](https://github.com/VS-VillageWork/tamara-bot/blob/master/mock3.png)

### Login in CentOS server contact admin for access

## Set Up Project

Pull project in `/var/www` folder 

```bash
cd /var/www && git pull https://github.com/VS-VillageWork/tamara-bot.git
```
change directory into tamara-bot

```bash
cd tamara-bot 
```

create Virtual Enviroment in bot and install requirements.txt

```bash
python3 -m venv dev
```
activate Virtual Environment

```bash
dev/bin/activate
```

Install requirements

```bash
pip install -r requirements.txt
```
## Test 

```bash 
cd ~
```

```bash
cd /var/www/tamara-bot && ./dev/bin/gunicorn --bind 0.0.0.0:7000 wsgi
```

Test with bot emulator using the IP address :7000 test with end point `api/messages`


## Create a Systemd Unit File

The next piece we need to take care of is the Systemd service unit file. Creating a Systemd unit file will allow CentOS’s init system to automatically start Gunicorn and serve our Flask application whenever the server boots.

Create a file ending with .service within the /etc/systemd/system directory to begin:

```bash
sudo nano /etc/systemd/system/tamara-bot.service
```

or 

```bash
sudo vi /etc/systemd/system/tamara-bot.service
```


```bash

[Unit]
Description=Gunicorn instance to serve tamara bot
After=network.target

[Service]
User=user
Group=nginx
Environment=MONGODB_URI=mongodb://localhost:27017
Environment=MICROSOFT_APP_PASSWORD=""
Environment=MICROSOFT_APP_ID=""
ExecStart=/bin/sh -c 'cd /var/www/tamara-bot && ./dev/bin/gunicorn --workers 5 --bind unix:tamarabot.sock -m 007 wsgi'
[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl start tamara-bot
sudo systemctl enable tamara-bot
```

## Configuring NGINX to Proxy Requests

```bash
sudo nano /etc/nginx/nginx.conf
```

Open up a server block just above the other server {} block that is already in the file:
```bash
http {
    . . .

    include /etc/nginx/conf.d/*.conf;

    server {
    }

    server {
        listen 80 default_server;

        . . .
```

We will put all of the configuration for our bot application inside of this new block. We will start by specifying that this block should listen on the default port 80 and that it should respond to our server’s domain name or IP address:

```bash
server {
    listen 80;
    server_name server_domain_or_IP;
}
```

The only other thing that we need to add is a location block that matches every request. Within this block, we’ll set some standard proxying HTTP headers so that Gunicorn can have some information about the remote client connection. We will then pass the traffic to the socket we specified in our Systemd unit file:

```bash
server {
    listen 80;
    server_name server_domain_or_IP;

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://unix:/home/user/tamara-bot/tamarabot.sock;
    }
```

Save and close the file when you are finished.
}

The nginx user must have access to our application directory in order to access the socket file there. By default, CentOS locks down each user’s home directory very restrictively, so we will add the nginx user to our user’s group so that we can then open up the minimum permissions necessary to grant access.

You can add the nginx user to your user group with the following command. Substitute your own username for the user in the command:

```bash
sudo usermod -a -G user nginx
```

Now, we can give our user group execute permissions on our home directory. This will allow the Nginx process to enter and access content within:

```bash
chmod 710 /user
```

With the permissions set up, we can test our Nginx configuration file for syntax errors:

```bash
sudo nginx -t
```

If this returns without indicating any issues, we can start and enable the Nginx process so that it starts automatically at boot:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```
## Contributors

DeveloperPrince

![DeveloperPrince](https://developerprince.herokuapp.com/static/assets/images/logo.png)