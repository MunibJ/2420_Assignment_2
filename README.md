#   Title: ACIT 2420 Assignment 2

Description: This program will create a VPC on digital ocean

# Team Members: 
    - Name: 	Munib Javed
      Email:    mjaved@my.bcit.ca
      Student #:  A01221306


# File Structure:
    - README.MD
    - images directory for any images used in the README.MD
    - 

# Instructions:

## Step 1:

Create A DO Inftrastructure that includes the followinhg components:
1. VPC
2. Atleast 2 Droplets
3. Load Balancer 
4. Firewall

## Step 2:
Install a web server on both of your droplets using Caddy, do not configure them yet.

1. Download the caddy file from: wget https://github.com/caddyserver/caddy/releases/download/v2.6.2/caddy_2.6.2_linux_amd64.tar.gz
2. unzip caddy file using tar -xvf caddy_2.6.2_linux_amd64.tar.gz
3. give caddy root permissiosn using sudo chmod 755 caddy or sudo chown root:root caddy
4. create  a caddy folder in /etc using sudo mkdir /etc/caddy
5. create a Caddyfile inside of the caddy folder using sudo touch /etc/caddy/Caddyfile and add the following to the file:
    -http://143.244.181.177 
    {
	root * /var/www
	file_server
    }
6. create a service file called caddy.service and put inside of it: 
    -[Unit]
    Description=Serve HTML in /var/www using caddy
    After=network.target

    [Service]
    Type=notify 
    ExecStart=/usr/bin/caddy run --config /etc/caddy/Caddyfile
    ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
    TimeoutStopSec=5
    KillMode=mixed

    [Install]
    WantedBy=multi-user.target
7. restart the service using sudo systemctl daemon reload
8. start the service using sudo systemctl start caddy
9. enable the service using sudo systemctl enable caddy


## Step 3:
1. create a folder and called it whatever you want but inside that folder, create two folders called html and src
2. Inside of the html folder, create an index.html file and put complete the html file with the following:
    -<!DOCTYPE html>
    <html>
    <body>

    <h1>My First Heading</h1>
    <p>My first paragraph.</p>

    </body>
    </html>
3. Inside of the src folder, run the command *npm init* and *npm i fastify*, this should create a json.package file and a node_modules folder
4. create an index.js file insde of your src folder and put the following code inside of it:
    ```const { createReadStream } = require('fs');
       const fastify = require('fastify')({ logger: true })

// Declare a route
fastify.get('/', async (request, reply) => {
  const stream = await createReadStream('../html/index.html');
  return reply.type('text/html').send(stream);
})
// Run the server!
const start = async () => {
  try {
    await fastify.listen({ port: 3000 })
  } catch (err) {
    fastify.log.error(err)
    process.exit(1)
  }
}
start()```
4. test your server on your local machine by installing
