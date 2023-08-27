# sonarqube-install-ubuntu 22.04LTS

****_SonarQube is an opensource web based tool to manage code quality and code analysis.
It is most widely used in continuous code inspection which performs reviews of code to detect bugs, code smells and vulnerability issues of programming languages 
such as PHP, C#, JavaScript, C/C++ and Java , Also tracks statistics and creates charts that enable developers to quickly identify problems in their code.**_**

**Prerequisites**
Ubuntu 22.04 LTS with minimum 2GB RAM and 1 CPU.
PostgreSQL Version 9.3 or higher
SSH access with sudo privileges
Firewall Port: 9000


STEP 1:LAUNCH an ec2 instance
select ubuntu-22.04LTS 
instance type:t2 small
go to security group and add inbound security rule as custom tcp port no 9000 -save
login to server

Here, We are installing SonarQube 8.9 version and have to install Oracle JAVA/Open JDK, Postgres/MS-SQL as database and Latest browser before installing SonarQube.
NOTE:
MySQL Support for SonarQube is depricated. Increase the vm.max_map_count kernal ,file discriptor and ulimit for current session at runtime]
sysctl -w vm.max_map_count=524288
sysctl -w fs.file-max=131072
ulimit -n 131072
ulimit -u 8192

To Increase the vm.max_map_count kernal ,file discriptor and ulimit permanently .
Open the below config file and Insert the below value as shown below,

 sudo nano /etc/security/limits.conf
 #uncooment and add below lines-save and exit"(cntl+x,y)
 sonarqube   -   nofile   65536
sonarqube   -   nproc    4096

Before installing, Lets update and upgrade System Packages
 sudo apt-get update
 sudo apt-get upgrade

Install wget and unzip package
sudo apt-get install wget unzip -y

Step #1: Install OpenJDk
Install OpenJDK and JRE 11 using following command,

 sudo apt-get install openjdk-17-jdk -y
 sudo apt-get install openjdk-17-jre -y
Check JAVA Version:
 java -version
 Step #3: Install and Setup PostgreSQL 10 Database For SonarQube
 
**** Add and download the PostgreSQL Repo****

sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
 wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -

****Install the PostgreSQL database Server by using following command,****
sudo apt-get -y install postgresql postgresql-contrib
Start PostgreSQL Database server

sudo systemctl start postgresql

Enable it to start automatically at boot time.

 sudo systemctl enable postgresql
 check status
  sudo systemctl status postgresql
Change the password for the default PostgreSQL user.

 sudo passwd postgres
 Switch to the postgres user.

su - postgres
Create a new user by typing:

createuser sonar
Switch to the PostgreSQL shell.

psql
Set a password for the newly created user for SonarQube database.

ALTER USER sonar WITH ENCRYPTED password 'sonar';
Create a new database for PostgreSQL database by running:

CREATE DATABASE sonarqube OWNER sonar;
grant all privileges to sonar user on sonarqube Database.

grant all privileges on DATABASE sonarqube to sonar;
Exit from the psql shell:

\q
Switch back to the sudo user by running the exit command.

exit
Step #4: How to Install SonarQube on Ubuntu 22.04 LTS
 cd /tmp
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
Unzip the archeve setup to /opt directory

 sudo unzip sonarqube-9.9.0.65466.zip -d /opt

error:then use following
ubuntu@ip-172-31-51-141:/tmp$ sudo -i
root@ip-172-31-51-141:~# apt-get install unzip
Move extracted setup to /opt/sonarqube directory

sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
Step #4:Configure SonarQube on Ubuntu 22.04 LTS
We can’t run Sonarqube as a root user , if you run using root user it stops automatically. We have found solution on this to create separate group and user to run sonarqube.

1. Create Group and User:
Create a group as sonar
sudo groupadd sonar
Now add the user with directory access

 sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar 
 sudo chown sonar:sonar /opt/sonarqube -R
Open the SonarQube configuration file using your favorite text editor.

 sudo nano /opt/sonarqube/conf/sonar.properties
Find the following lines.

#sonar.jdbc.username=
#sonar.jdbc.password=
Uncomment and Type the PostgreSQL Database username and password which we have created in above steps and add the postgres connection string.
add these lines and then save and exit

sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube

Edit the sonar script file and set RUN_AS_USER

 sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh
 add the following line
  RUN_AS_USER=sonar
  Type CTRL+X to save and close the file.

2. Start SonarQube:
Now to start SonarQube we need to do following: Switch to sonar user

 sudo su sonar
Move to the script directory

 cd /opt/sonarqube/bin/linux-x86-64/
Run the script to start SonarQube

./sonar.sh start
 Check SonarQube Running Status:
To check if sonaqube is running enter below command,

./sonar.sh status
Step #5:Configure Systemd service
First stop the SonarQube service as we started manually using above steps Navigate to the SonarQube installed path

 cd /opt/sonarqube/bin/linux-x86-64/
Run the script to start SonarQube

./sonar.sh stop
Create a systemd service file for SonarQube to run as System Startup.

 sudo nano /etc/systemd/system/sonar.service
Add the below lines,

/etc/systemd/system/sonar.service

[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target

 sudo systemctl start sonar
Enable the SonarQube service to automatically  at boot time System Startup.

 sudo systemctl enable sonar
check if the sonarqube service is running,

 sudo systemctl status sonar
Successfully, We have covered How to Install SonarQube on Ubuntu 22.04 LTS .
Save and close the file. 
Now stop the sonarqube script earlier we started to run using as daemon.
Start the Sonarqube daemon by running
Step #6: Access SonarQube
To access the SonarQube using browser type server IP followed by port 9000.

http://server_IP:9000 OR http://localhost:9000
Login to SonarQube  with default administrator username and password is admin.
![image](https://github.com/reena-lobo/sonarqube-install-ubuntu/assets/138814138/10ce5815-82d4-433a-82cd-147b8e7448d4)
default username :admin 
passowrd:admin
