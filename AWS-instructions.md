# Create new VPC
  1. Fo to VPC
  2. Launch VPC Wizard (make sure you're in a right availability zone)
  3. Choose VPC with a Single Public Subnet (for production choose VPC with Public and Private Subnets)
  4. Type VPC name
  5. Create VPC
You'll automatically have one subnet and two route tables

# Create security group
  1. In VPC, click Security groups on the menu on the left side
  2. Click create security group
  3. Web security group
    * Create name
    * Description 80/443
    * Add inbound rules:
      * Type: HTTP Destination: Anywhere
      * Type: HTTPS Destination: Anywhere
    * Add tag
  4. SSH security group
    * Create name
    * Description 22
    * Add Inbound rule:
      * Type: SSH Destination: MyIP
    * Add tag

Now you have two security groups: HTTP, HTTPS and SSH


# EC2
  1. Create instance
  2. Choose AMI
    * Choose Ubuntu 20.04
  3.   Instance type
    * Choose t2.micro or t3.micro depending on location
  4. Configure instance
    * Choose network
    * Enable auto-assign public IP
    * Check "Protect against accidental termination"
  5. Add storage
    * Change volume type to g3
  6. Add tags
    * Name - type name
  7. Configure security group
    * Choose existing
    * Check SSH and WEB security groups
    * Click launch
  8. Key pair pop-up
    * Choose create a new key and type the name
    * Download it
    * Launch

9. iTerm
  * to change permission configs `chmod 400 name_of_key`
  or
  * SSH shortcut `ssh -i "name_of_file" ubuntu@publicIPaddressfromInstance`
  * type "yes" and hit enter

# Elastic IP
  1. Go to Elastic IP on the left menu in VPC
  2. Allocate elastic IP
  3. Click Allocate

  Now you have an ip you can name it "my IP"

  4. Click Action
  5. Associate elastic IP
    * Choose running instance
    * Choose IP
    * Click Associate

Now your Elastic IP is your public IP

* in iTerm do SSH shortcut again, type "yes" and hit enter

# Route 53
  1. copy elastic IP
  2. Go to Route53
  3. Click Hosted zones on the left menu
  4. Click on domain
  5. Create record
  6. Paste Elastic IP in Values
  7. Click Create record

  You can create alias with www and then use it in ssh shortcut instead of IP

* in iTerm type `sudo nano ~/.ssh/config`. It might be blank
* then type
```
Host <name>
  User ubuntu
  Hostname <DNS or elastic IP>
  IdentityFile <path to the key file>
```
* hit ctrlX to exit -> hit Y -> hit enter

###### You can use pwd command to get the ultimate path to the key file

Now you can do `ssh <host name>`

# Configure Ubuntu
* To update and upgrade ubuntu in terminal do ssh and in ubuntu type `sudo apt update && sudo apt upgrate -y`
* Reboot EC2 instance using Instance state drop down menu
* In ubuntu run the following command `sudo apt install nginx -y`
* Type `sudo systemctl enable nginx`
* To check status type `sudo systemctl status nginx` (to quit enter Q)
* Type `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v16.2.0/install.sh | bash` (to find this go to nvm github get curl from install and update section)
* Exit ubuntu and do ssh shortcut again
* `nvm install --lts`
* Now you can check `node -v`
* `nvm install stable`
* To change node version `nvm use 14` or `nvm use stable`
* In another tab go to `~/.ssh` and then type `node -v` check the node version and copy that
* `nvm install <copied version>` in ubuntu and to use this version try `nvm use <copied version>`

# Configure NGINX
* `cd /etc/nginx/sites-available` if you go to sites-enabled there are sites thatyou're running but for configuration we need sites-available
* `sudo touch <file name>` or `sudo nano <file name>`
* `sudo nano <file name>`
* type
```
server {
      listen 80;
      server_name kvaknastya.com www.kvaknastya.com;
      location / {
              proxy_pass http://127.0.0.1:3000;
              proxy_http_version 1.1;
              proxy_set_header Upgrade $http_upgrade;
              proxy_set_header Connection 'upgrade';
              proxy_set_header Host $host;
              proxy_cache_bypass $http_upgrade;
              proxy_redirect off;
      }

}
```
* Save and exit ubuntu
* `sudo systemctl reload nginx` - you should do it everytime you do a configuration change
* `sudo ln -s /etc/nginx/sites-available/NSKvak-online /etc/nginx/sites-enabled/NSKvak-online`
* Check nginx `sudo nginx -t`
Now you should have 502 bad gateway

# Get project on instance

## Front-end
* In ubuntu do git clone for backend and frontend
* `node -v` to check versions
* Cd to home and front-end file
* `npm i`
* `npm start`

## Back-end
* Open another tab
* Do ssh to open ubuntu
* Cd to home
* Git clone backend file from github
* Go back to `sudo nano <file>`
* Append the following code after previous location
```
location /api/ {
              proxy_pass http://127.0.0.1:3001;
      }
```
* Save and exit ubuntu
* Check nginx `sudo nginx -t`
* `sudo systemctl reload nginx` - you should do it everytime you do a configuration change
* Cd to home and back-end file
* `npm i`
* Don't forget to sudo nano .env and copy paste .env content from your code to server
* `npm start`

## P.S. 
* For front-end and back-end put this on nginx config
```
  server {
        listen 80;
        server_name yourdomain.com  www.yourdomain.com ;
        location / {
                proxy_pass http://127.0.0.1:3000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
                proxy_redirect off;
        }
        location /api/ {
                proxy_pass http://127.0.0.1:3001;
        }
  }
```

  ###### Make sure your code is running on these ports and there are no other apps running on these ports.
* For static websites use the following
```
  server{
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    location / {
      root/home/ubuntu/myproject/build;
      index index.html;
    }
    server_name yourdomain.com www.yourdomain.com;
    location / {
      root/home/ubuntu/myproject/build;
      try_files $uri/index.html
    }
  }
```
* For frontend only use 
```
server {
      listen 80;
      server_name yourdomain.com  www.yourdomain.com ;
      location / {
              proxy_pass http://127.0.0.8080;
              proxy_http_version 1.1;
              proxy_set_header Upgrade $http_upgrade;
              proxy_set_header Connection 'upgrade';
              proxy_set_header Host $host;
              proxy_cache_bypass $http_upgrade;
              proxy_redirect off;
      }
}
```




### Kill an App Command (if you accidently closed the tab that's running your code)
  * `sudo lsof -t -i:3000` (or another port number like 8080)
  * `sudo kill -9 <pid from the first command>`

# You can create react app on server
* ssh to your server
* `npx create-react-app <name>`
* cd to the react folder
* `npm i`
* `npm run build`
* `cd build`
* `pwd` copy the path
* In a new terminal tab go to ssh server again
* `cd /etc/nginx/sites-available/`
* `cp <name of previous file> <name of a new file>` - it will copy server configs from previous file to the new one. Or you can do nano and type it manually
* Check your new file `cat <new file name>`
* Then open server config using nano `sudo nano <new file name>`
* Delete content inside location and type new one
```
location / {
              root /home/ubuntu/test/build;
              index index.html;
      }
```
* Save and exit
* cd to sites-enabled `cd ../sites-enabled`
* `sudo rm <old file name>` make sure you're deleting this file from sites-enabled not sites-available
* `sudo systemctl reload nginx`
* `sudo ln -s /etc/nginx/sites-available/<new file name>   /etc/nginx/sites-enabled/<create another name>`

If you want to change code in server
* go to the file
* go to src file
* do `sudo nano App.js`
* save and exit
* `npm run build`


# Setup subdomain
* Copy your elastic IP
* Go to Route 53
* Create record
* Name record however you want, paste elastic IP to value and click create record
* In ubuntu open server config usinng nano and change server_name to subdomain
* `sudo systemctl reload nginx`
* `sudo nginx -t`
now your subdomain shows the static site
* `sudo ln -s /etc/nginx/sites-available/NSKvak-online /etc/nginx/sites-enabled/NSKvak-online`
* `sudo systemctl reload nginx`
Now your primary domain shows 502.

# Setup MongoDB
* Go to MongoDB.com
* Create a new project
* after the project is created, click on it's name and choose cluster
* Setup database
### Network Access
* While waiting, go to Network Access, add IP address
* Add Elastic IP 
* Add your home IP using "create your home IP" button

### Database Access
* Click on Database access on the left menu and create database user
* Choose password
* Username: dbAdmin
* Create password and copy it
* In Ubuntu go to backend, create .env file using nano, and in nano paste the password
* On MongoDB Databases click connect and choose connect application
* Copy the string
* Go to .env using `nano .env` and add `MONGODB_URI=<copied string>`
* Grab the password you had there and paste it into the string where the password should be.
* Instead of myFirstDatabase type your own database name (name whatever you want), keep ? after the database
* Add `SECRET_KEY=<your key>`
* `npm start` - run it to create database
Now you can check your database on mongodb website

# Update code
* Just do git push in refular file 
* In ubuntu go to the file folder and do git pull
* `npm start`

# Process management
* Go to ubuntu
* npm i pm2 -g
* cd to backend folder

#### backend
* Name the process: Type `pm2 start --name <file name> npm -- start` - for the last dash there is spaces before and after. 
#### Start/ Restart/ Stop
* Start process: `pm2 start <process name>`
* Get status: `pm2 status`
* Stop: `pm2 stop 1` or use `pm2 stop all` to stop all
* To use name flag `-name<Name>`
* To rename a current process `pm2 restart <Current name>-name<New Name>`
* To delete do `pm2 delete <number of the process>`
* To restart `pm2 restart <number of process>
#### Do the same for frontend


* don't forget to save `pm2 save`
* After you make everything online and saved it type `pm2 sturtup`. 
* It will run the script  that will tell you to copy and then paste ina. command line
* After it says pm2 save, reboot ec2 instance

# Enable HTTPS
* Go to your SSH
* `sudo apt update`
* `sudo apt install certbot python3-certbot-nginx`
* `sudo ufw status`
* `sudo certbot --nginx -d <first domain> -d <secind domain>`
* Don't need certificate
* 2. redirect




