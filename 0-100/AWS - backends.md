- It is a site which helps you to rent servers, manage domains, upload objects, autoscale servers, create kubernetes servers.
## EC2 Servers
- Elastic Compute 2
- Elastic - canincrease or decrease the size of machine
  Compute - Machine.
- **Keypair** Whenever you create a server you need a keypair which will help you login into the server.
## Networking Settings
- The server also requires you to specify some ports which needs to be opened for the internet to use your server.
![[Pasted image 20240727004611.png]]

- **SSH - Secure Shell** - Allows you to connect to your remote server.
- **Allow https traffic** - by default the https port is 443. Any requests made to the https website will go to this port.
- **Allow http traffic** - by default the https port is 80. Any requests made to the https website will go to this port.
- Usually only https should be open.
- The instance will have a public ip and a DNS.
## Connecting to the Server
```
// Change permissions for the passkey.
chmod 700 passkey.pem
// Go to the folder containing the passkey file.
ssh -i passkey.pem ubuntu@--instance-ip--

// To exit
exit
```

- If you dont chmod => change permissions error will occur.
- May lead to some errors on windows machines.
- You will get access to the server, you can clone your repo, install node and dependencies and start the server.
#### 1.Installing Node
- Search the web
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```
- Finally export
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

#### 2. Exposing Ports to the web
- Secutrity Groups -> Edit Inbound Rules -> Add rules -> Common TCP,  port, ipv4 and ipv6.

## Setting up reverse proxy - Nginx
- These are servers which receives all requests and passes it to the exposed port. 
- This server is mapped to the DNS.
- It can also be used for load balancing, request filtering etc.
- The 80 port is exposed on these servers.
- You can map your DNS to point to this server
```
sudo apt update
sudo apt install nginx
// This will start nginx and it will be exposed to public domain port 80(default)

sudo rm /etc/nginx/nginx.conf
sudo vi /etc/nginx/nginx.conf

// Add the following code to /etc/nginx/nginx.conf
//ec2-3-111-37-129.ap-south-1.compute.amazonaws.com can be switched with your domain as well.
events {

}

http {
    server {
        listen 80;
        server_name ec2-3-111-37-129.ap-south-1.compute.amazonaws.com;

        location / {
            proxy_pass http://localhost:8080;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_cache_bypass $http_upgrade;
        }
    }
}



// The code basically says to create a http server that maps the domain(server_name) to the localhost:8000.
// Exit and reload the nginx server.
sudo nginx -s reload

```


#### making your ubuntu to point to certain server when requesting a particular domain -
