#   Title: ACIT 2420 Assignment 2

Description: This program will create a VPC on digital ocean

# Team Members: 
```
      Name: 	  	Munib Javed
      Email:      	mjaved@my.bcit.ca
      Student #:  	A01221306

```
# Instructions:

## Step 1: Creating Digital Ocean Infrastructure  

Create A DO Infrastructure by following the video below:
```https://vimeo.com/775412708/4a219b37e7```

It should include these components:
```
1. VPC
2. At least 2 Droplets
3. Load Balancer 
4. Firewall
```
<br>
<br>


## Step 2: Creating a Regular Users on your Droplets
Follow the guide in the link before to create a regular user on your droplets. 
```
https://learn.bcit.ca/d2l/le/content/877753/viewContent/8037537/View
```
<br>
<br>

## Step 3: Adding Caddy to Droplets
Install a web server on both of your droplets using Caddy, do not configure them yet.

1. Download the caddy file from: 
```
wget https://github.com/caddyserver/caddy/releases/download/v2.6.2/caddy_2.6.2_linux_amd64.tar.gz
```

2. unzip caddy file using `tar -xvf caddy_2.6.2_linux_amd64.tar.gz`
3. give caddy root permissions using `sudo chmod 755 caddy` or `sudo chown root:root caddy`
4. create  a caddy folder in /etc using `sudo mkdir /etc/caddy`
5. create a Caddyfile inside of the caddy folder using sudo touch /etc/caddy/Caddyfile and add the following to the file:
   ```
   http://143.244.181.177 
    {
	root * /var/www
	file_server
    }
    ```
6. restart the service using `sudo systemctl daemon reload`
7. start the service using `sudo systemctl start caddy`
8. enable the service using `sudo systemctl enable caddy`
<br>
<br>


## Step 4: Write your Webapp 
1. create a folder called `www` inside of your /var folder,and inside the `www` folder, create two folders called `html` and `src`
2. Inside of the html folder, create an index.html file and put complete the html file with the following:
    ```
    -<!DOCTYPE html>
    <html>
    <body>

    <h1>My First Heading</h1>
    <p>My first paragraph.</p>

    </body>
    </html>
    ```
3. Inside of the src folder, run the command `npm init` and `npm i fastify`, this should create a json.package file and a node_modules folder
4. create an index.js file inside of your src folder and put the following code inside of it:
    ```
      const { createReadStream } = require('fs');
      const fastify = require('fastify')({ logger: true })
  
    fastify.get('/', async (request, reply) => {
    const stream = await createReadStream('../html/index.html');
    return reply.type('text/html').send(stream);
    })
    const start = async () => {
      try {
        await fastify.listen({ port: 3000 })
      } catch (err) {
        fastify.log.error(err)
        process.exit(1)
      }
    }
    start()
    ```
4. test your server on your local machine by installing node on your local machine following the url instructions :
    ```
    https://www.fastify.io/docs/latest/Reference/Routes/
    ```

Node Instructions: Inside your local machine, install node using the following command:
```
1.curl https://get.volta.sh | bash
2.source ~/.bashrc
3.volta install node
4.which npm
```
The output of `which npm` should be `/home/user/.volta/bin/npm`

5. after you install it, run it by typing `node index.js` in your terminal
6. go to your browser and type in `localhost:3000` and you should see your html file
7. if you can see the HTML file, it means it worked and now you can transfer the html folder and src folder over to both your droplets using the rsync command. Details about the command can be found in link below:
    ```
    https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories-on-a-vps
    ```
8. Your html and src folders should go inside of the /var/www folder on both droplet `(you need to create another www folder in /var again on both droplets)`
<br>
<br>

## Step 5: Writing Caddyfile Server Block 

1. create a  file called `Caddyfile` on your local machine and add the following to it:
      ```
      http://  
        {   
        root * /var/www/html
        reverse_proxy /app localhost:5050
        file_server
        }
      ```
2. transfer the `Caddyfile` from your local host to your droplets using rsync command. 
3. create a caddy folder in your droplets /etc folder using `sudo mkdir /etc/caddy`
3. then move the file into your caddy folder using `sudo mv caddyfile /etc/caddy`

## Step 6: Installing Volta/Node on your droplets
1. Install volta on your droplets using the following commands:
    ```
    curl https://get.volta.sh | bash
    source ~/.bashrc
    volta install node
    which npm
    ```
<br>
<br>



## Step 6: write a caddy service file
Your service file should restart the service on failure. Your service file should require a configured network

1. create a service file called node.service and put inside of it: 
    ```
    [Unit]
    Description=Serve HTML in /var/www using caddy
    After=network.target

    [Service]
    Type=notify 
    ExecStart=/usr/bin/caddy run --config /etc/caddy/Caddyfile
    ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
    TimeoutStopSec=5
    KillMode=mixed

    [Install]
    WantedBy=multi-user.target```
   ```
2. move the service file to your /etc/systemd/system folder using `sudo mv caddy.service /etc/systemd/system`
3. enable the service using `sudo systemctl enable caddy.service`
4. start the service using `sudo systemctl start caddy.service`
5. check the status of the service using `sudo systemctl status caddy.service`

## Step 7: Writing a Node Service File
1. create a service file called node.service and put inside of it: 
    ```
    [Unit]
    Description=Making something
    After=network-online.target
    Wants=network-online.target

    [Service]
    ExecStart=/home/user/.volta/bin/node /var/www/src/index.js
    User=your-droplet-user-name
    Group=your-droplet-user-name
    Restart=always
    RestartSec=10
    TimeoutStopSec=90
    SyslogIdentifier=hello_web

    [Install]
    WantedBy=multi-user.target
    ```
2. move the service file to your /etc/systemd/system folder using `sudo mv node.service /etc/systemd/system`
3. enable the service using `sudo systemctl enable node.service`
4. start the service using `sudo systemctl start node.service`
5. check the status of the service using `sudo systemctl status node.service`

## Step 8: Testing your Webapp
1. inside of one of your html files, change it up to say something else compared to the other droplets so you can differentiate between the two.
2. go to your browser and type in the ip address of your load balancer and you should see the html file you changed
3. if you see the html file, it means it worked and you can now go to your load balancer and add the droplets to the load balancer
4. Refresh your browser and you should see the other droplet's html file
5. You test the `/app` route by going to your browser and typing in the ip address of your load balancer followed by `/app` and you should see the html file you changed.

The end result should look like this:

![](/images/question9server1.png)
![](/images/question9server2.png)


`Congrats! You have successfully created a load balancer and deployed a webapp on it.`

