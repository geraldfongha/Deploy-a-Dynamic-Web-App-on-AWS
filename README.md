# Deploy-a-Dynamic-Web-App-on-AWS
Hosting a dynamic Web App on AWS cloud with an EC2 instance while using flyway to migrate SQL data to MySQL RDS database. 

## Architecture Overview

The website is hosted on EC2 instances within a Virtual Private Cloud (VPC) configured with public and private subnets spanning two Availability Zones. The infrastructure leverages the following AWS resources:

- **Virtual Private Cloud (VPC)**: A logically isolated section of the AWS cloud where AWS resources are launched.
- **Internet Gateway**: Enables communication between the VPC instances and the internet.
- **Security Groups**: Act as virtual firewalls to control inbound and outbound traffic.
- **Availability Zones**: Multiple Availability Zones are used to increase reliability and fault tolerance.
- **Public Subnets**: Host infrastructure components like the NAT Gateway and Application Load Balancer.
- **Private Subnets**: Host the web servers (EC2 instances) for enhanced security.
- **NAT Gateway**: Allows instances in private subnets to access the internet.
- **EC2 Instance Connect Endpoint**: Enables secure connections to EC2 instances within both public and private subnets.
- **Application Load Balancer**: Distributes incoming web traffic across multiple EC2 instances in an Auto Scaling group.
- **Auto Scaling Group**: Automatically manages the EC2 instances hosting the website, ensuring availability, scalability, fault tolerance, and elasticity.
- **AWS Certificate Manager**: Secures application communications with SSL/TLS certificates.
- **Simple Notification Service (SNS)**: Sends notifications about activities within the Auto Scaling Group.
- **Route 53**: Provides DNS services for registering and managing the website's domain name.
- **S3 Bucket**: Stores the application code and assets.

## Deployment

The deployment of this infrastructure is automated using scripts and configuration files stored in a GitHub repository. The repository includes the following:

- **Reference Diagram**: A visual representation of the AWS infrastructure and its components.
- **Deployment Scripts**: Scripts to provision and configure the necessary AWS resources.
Below is the application code used to complete this task.
#Update ec2 server
sudo yum update -y
#Install Apache
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
#Install PHP version 8
sudo yum install php -y
#Install php extension
sudo yum install php php-fpm php-mysqlnd php-bcmath php-ctype php-fileinfo php-json php-mbstring php-openssl php-pdo php-gd php-tokenizer php-xml -y
#Install the MySQL Community repository
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
#install MySQL Version 8
sudo dnf install mysql80-community-release-el9-1.noarch.rpm -y
dnf repolist enabled | grep "mysql.-community." -y
sudo dnf install mysql-community-server -y
#Start the MySQL server
sudo systemctl start mysqld
sudo mysql -V
#Enable PHP_CURL Module
sudo yum install php-curl -y
#To edit the php.ini file
sudo sed -i 's/^max_execution_time = .*/max_execution_time = 300/' /etc/php.ini
#USE THIS CODE TO SEARCH FOR THE PHP.INI FILE#
cat php.ini | grep memory_limit
cat php.ini | grep max-execution_time
#Enable mod_rewrite on ec2 linux, add apache to group, and restart server
sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf
sudo service httpd restart
#download the nest zip from s3 to the html derectory on the ec2 instance
sudo aws s3 sync s3://kachi-nest-web-files /var/www/html
#unzip the nest zip folder
cd /var/www/html
sudo unzip nest-app.zip
#move all the files and folder from the nest-app directory to the html directory
sudo mv nest-app/* /var/www/html
#move all the hidden files from the nest-app directory to the html directory
sudo mv nest-app/.editorconfig /var/www/html
sudo mv nest-app/.env /var/www/html
sudo mv nest-app/.env.example /var/www/html
sudo mv nest-app/.gitattributes /var/www/html
sudo mv nest-app/.gitignore /var/www/html
sudo mv nest-app/.htaccess /var/www/html
#delete the nest and nest.zip folder
sudo rm -rf nest-app nest-app.zip
#Set permissions 777 for the '/var/www/html' directory and the storage/' directory
sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/
#Restart Apache server
sudo service httpd restart
GO TO HOME DIRECTORY
MIGRATING SGQL DATA TO RDS WITH FLYWAY
1.	Create an S3 bucket and upload the SQL (Script) file.
2.	S3Role
3.	Ec2 endpoint connect
#Download fly
wget -qO- https://repo1.maven.org/maven2/org/flywaydb/flyway-commandline/9.22.3/flyway-commandline-9.22.3-linux-x64.tar.gz | tar -xvz && sudo ln -s pwd/flyway-9.22.3/flyway /usr/local/bin
cd flyway-9.22.3
NOTE: To run the flyway commands you must be in the flyway directory
#delete flyway sql directory
rm -rf sql
#create own sql directory
mkdir sql
#copy sql file from S3 to sql directory in flyway
aws s3 cp s3://s3://fongha-sql-files/V1__shopwise.sql
#Migrates data to database
flyway -url=jdbc:mysql://kachi-rds-db.cso3n7l0ywqv.eu-west-2.rds.amazonaws.com:3306/applicationdb \
-user=cach \
-password=Ebunoluwa \
-locations=filesystem:sql \
migrate
#TO EDIT THE .env FILE
cd /var/www/html
nano .env
i. APP_URL=enter your domain name
ii. DB_HOST=enter your RDS Endpoint
iii. DB_DATABASE=enter your RDS database name
iv. DB_USERNAME=enter your RDS username name
v. DB_PASSWORD=enter your RDS database password 
-Create target group
-Create Application Load Balance
-Create Route 53
-Create HTTPS listerner
#To Edit the AppServiceProvider.Phpfile
cd /var/www/html
nano .env
Ls
Cd app
Ls
Cd Providers
Ls
Type: sudo vi AppServiceProvider.php
Copy and paste:
if (env('APP_ENV') === 'production') {\Illuminate\Support\Facades\URL::forceScheme('https');}
Create Launch template
Create Auto Scaling Group


## Usage

1. Clone the GitHub repository to your local machine.
2. Follow the instructions in the repository's documentation to set up the required AWS resources.
3. Deploy the website code and assets to the appropriate S3 bucket.
4. Access the website using the registered domain name.

## Contributing

Contributions to this project are welcome. If you find any issues or have suggestions for improvements, please open an issue or submit a pull request in the GitHub repository.
