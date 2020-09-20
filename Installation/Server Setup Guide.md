# Setting up an Autolab server

## Installation Requirements

### Environment

- Operating System: Linux (Instructions tested with Ubuntu 16 & 18).
- RAM: 8 GB (Prefered)
- Storage: 40 GB

### Hosting

- Use an Amazon EC2 or Google Instance
- Allow HTTP traffic (port 80) and HTTPS traffic (port 443) from the hosting service.

### Email Account

- An email account with password is required for sending user registration and notifications emails to Autolab users.
- For eg. autolab-mydomain@gmail.com can be created for use. 
- The app password can obtained by enabling 2 factor authetication on the account (if using google account).

### Permissions

- Requires root access

## 1. Download

1. Make sure you have **git**.

   ```bash
   sudo apt-get install git 
   ```

2. Enter sudo mode.

   ```bash
   sudo -i
   ```

3. Get the installation files.

   ```bash
   git clone https://github.com/uncc-autolab/autolab-oneclick
   cd autolab-oneclick
   ```

## 2. Configure

### Setting up email service

1. Generate a new secret key for Devise Auth Configuration:

   ```python
   python -c "import random; print(hex(random.getrandbits(512))[2:-1])"
   ```

2. Update the values in *server/configs/devise.rb*

   ```ruby
    config.secret_key = <GENERATED_SECRET_KEY>
    config.mailer_sender = 'myemail@domain.com' # set email 
   ```

3. Configure email in *server/configs/production.rb*.  Update the address, port, user_name, password and domain with your email service information.

   ```ruby
   config.action_mailer.default_url_options = {protocol: 'http', host: 'subdomain.domain.com'}
   config.action_mailer.smtp_settings = {
         address:              'smtp.gmail.com',
         port:                 587,
         user_name:            'myemail@domain.com',
         password:             <Get This App Password From Email Account >,
         domain:               'domain.com',
     }
   ```

   

### Setting up SSL

1. Install Nginx and make sure its running.

   ```bash
   sudo apt install nginx
   systemctl status nginx
   ```

2. Install Certbot on Ubuntu 18.04 with Nginx (follow the instructions in the given URL to get exact installation command)

   ```bash
   # https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx
   sudo snap install --classic certbot
   ```

3. Get the SSL certificates

   ```bash
   sudo certbot certonly --nginx
   ```

   The certificates are installed in:

   ```bash
   # /etc/letsencrypt/live/yourdomain.com
   ```

4. Copy SSL certificate and key to *server/ssl* directory. 

   ```bash
   cp /etc/letsencrypt/live/yourdomain.com/cert.pem server/ssl
   cp /etc/letsencrypt/live/yourdomain.com/privkey.pem server/ssl
   ```

5. Configure Nginx in *server/configs/nginx.conf*

   ```
   server_name subdomain.domain.com 				# set your web address here
   ssl_certificate /etc/nginx/ssl/cert.pem 
   ssl_certificate_key /etc/nginx/ssl/privkey.pem
   ```

6. Stop the nginx on your host machine

   ```bash
   systemctl stop nginx
   ```

### Setting up an autograding VM for your course (Optional)

1. Create your course folder and the associated files in *course-vms*

   ```
   | - autolab-oneclick
   		| - course-vms
   				| - MY_COURSE
   						| - bashrc 
   						| - Dockerfile
   						| - requirements.txt
   ```

2. Edit *autolab-oneclick/cover/start.sh* to include docker build command for your autograding VM.

   ```bash
   echo "Building autograding VM for MY_COURSE"
   docker build -t autograding_MY_COURSE vmms/MY_COURSE/;
   ```

3. Add a mapping for your autograding VM in  *autolab-oneclick/server/docker-compose.yml*. This is useful in updating your VM post-installation.

   ```
   tango:
     build: ./Tango
     command: sh start.sh
     ports:
       - '8600:8600'
     privileged: true
     volumes: 
     - ./Tango/config.py:/opt/TangoService/Tango/config.py
     - ./Tango/start.sh:/opt/TangoService/Tango/start.sh
     - ./Tango/vmms/MY_COURSE:/opt/TangoService/Tango/vmms/MY_COURSE  # <-- Add this line for MY_COURSE
   ```

   

## 3. Install

```bash
cd autolab-oneclick # If not in this directory already
chmod +x install.sh # Give execute permission
./install.sh -s     # Install. This will take a while
```

Verify your installation by ensuring docker three containers are running.

```
docker ps
```







 



​	