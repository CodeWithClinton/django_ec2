Here's the formatted text for a GitHub README file:

---

# Deploying a Django Project to an AWS EC2 Instance

Deploying a Django project to an AWS EC2 instance involves several steps, including setting up the EC2 instance, preparing your Django project, configuring the server environment, and setting up a production server with Nginx and Gunicorn. Here's a step-by-step guide to help you through the process:

## Step 1: Set Up Your EC2 Instance

### Create an AWS EC2 Instance:
1. Log in to your AWS Management Console.
2. Go to the EC2 Dashboard and click on "Launch Instance."
3. Choose an Amazon Machine Image (AMI). For a basic setup, the Ubuntu Server is a good choice.
4. Select an instance type (e.g., t2.micro for free tier).
5. Configure instance details, add storage, and configure security groups (allow SSH, HTTP, and HTTPS).
6. Review and launch the instance. You will be prompted to create or use an existing key pair for SSH access.

### Connect to Your EC2 Instance:
Open your terminal and connect to your instance using SSH:

```bash
ssh -i /path/to/your-key.pem ubuntu@your-ec2-public-ip
```

## Step 2: Prepare Your Django Project

### Clone Your Project Repository:
Install Git if it's not already installed:

```bash
sudo apt update
sudo apt install git
```

Clone your Django project repository:

```bash
git clone https://github.com/yourusername/your-django-repo.git
cd your-django-repo
```

### Set Up a Virtual Environment:
Install Python and virtual environment tools:

```bash
sudo apt install python3-pip python3-venv
```

Create and activate a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

### Install Dependencies:
Install your project's dependencies:

```bash
pip install -r requirements.txt
```

## Step 3: Configure the Server Environment

### Install and Configure PostgreSQL:
Install PostgreSQL:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

Set up a PostgreSQL database and user for your Django project:

```bash
sudo -u postgres psql
CREATE DATABASE yourdbname;
CREATE USER yourdbuser WITH PASSWORD 'yourpassword';
ALTER ROLE yourdbuser SET client_encoding TO 'utf8';
ALTER ROLE yourdbuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE yourdbuser SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE yourdbname TO yourdbuser;
\q
```

Update your Django settings to use the PostgreSQL database.

### Run Database Migrations:

```bash
python manage.py migrate
```

## Step 4: Set Up Gunicorn and Nginx

### Install Gunicorn:

```bash
pip install gunicorn
```

### Create a Gunicorn Systemd Service File:
Create and edit the service file:

```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Add the following configuration:

```ini
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/your-django-repo
ExecStart=/home/ubuntu/your-django-repo/venv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/ubuntu/your-django-repo.sock yourprojectname.wsgi:application

[Install]
WantedBy=multi-user.target
```

Reload systemd to apply the changes:

```bash
sudo systemctl daemon-reload
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

### Install and Configure Nginx:

Install Nginx:

```bash
sudo apt install nginx
```

Create an Nginx configuration file for your project:

```bash
sudo nano /etc/nginx/sites-available/yourproject
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ubuntu/your-django-repo;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/your-django-repo.sock;
    }
}
```

Enable the Nginx configuration:

```bash
sudo ln -s /etc/nginx/sites-available/yourproject /etc/nginx/sites-enabled
sudo nginx -t
sudo systemctl restart nginx
```

### Update Firewall Settings:
Allow Nginx through the firewall:

```bash
sudo ufw allow 'Nginx Full'
```

---

This guide provides a clear, step-by-step process for deploying a Django project to an AWS EC2 instance.