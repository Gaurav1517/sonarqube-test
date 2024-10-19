# sonarqube-test
Integrating SonarQube with Jenkins and testing

Apache maven is one of the best open-source java build tools. It is primarily used by developers working on java projects and devops engineers working on CI/CD pipelines. A developer primarily deals with the development aspects using maven while devops engineers deal with setting up a Continuous Integration pipeline using maven.

Let’s get started with the maven setup.

**Install Latest JDK**
The basic prerequisite for Maven is Java JDK because Maven is a build tool and it requires all the build and compiles related java tools.

Let’s install the latest JDK.

For Redhat Linux,
```bash
yum install java-17-openjdk-devel -y
```

Verify Java JDK installation using the following command.
```bash
java -version
```

**Set envrionment variable
```bash
vim ~/.bashrc
```

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk
export PATH=$JAVA_HOME/bin:$PATH
```

**Install Maven (Latest From Maven Repo)**
Follow the steps given below to install the latest maven package from the official maven repo.

Step 1: Go to Maven Downlaods and get the download link of the latest package.

```bash
wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz
```

Step 2: Untar the mvn package to the /opt folder.
```bash
sudo tar xvf apache-maven-3.9.9-bin.tar.gz -C /opt
```

Step 3: Create a symbolic link to the maven folder. This way, when you have a new version of maven, you just have to update the symbolic link and the path variables remain the same.

```bash
sudo ln -s /opt/apache-maven-3.9.9 /opt/maven
```

Note: Always install specific version of maven that is used across CI environment. So that there wont be any build issues due to version mismatch when you take the code from development to deployment environments

**Add Maven folder to System PATH**
To access the mvn command systemwide, you need to either set the M2_HOME environment variable or add /opt/maven to the system PATH.

We will do both by adding them to the profile.d folder. So that every time the shell starts, it gets sourced and the mvn command will be available system-wide.

Step 1: Create a script file named maven.sh in the profile.d folder.

```bash
sudo vi /etc/profile.d/maven.sh
```

Step 2: Add the following to the script and save the file.
```bash
export M2_HOME=/opt/maven
export PATH=${M2_HOME}/bin:${PATH}
```

Step 3: Add execute permission to the maven.sh script.

```bash
sudo chmod +x /etc/profile.d/maven.sh
```

Step 4: Source the script for changes to take immediate effect.
```bash
source /etc/profile.d/maven.sh
```

Step 5: Verify maven installation
```bash
mvn -version
```
Note: In Ubuntu based systems, you can also add M2_HOME and PATH variables to users .bashrc file as well.

**SONARQUBE INSTALLATION STEPS**

```bash
sudo vi /etc/sysctl.conf
```
--------------------------------------
ADD FOLLOWING LINES AND SAVE WITH :WQ
--------------------------------------
```bash
vm.max_map_count=262144
fs.file-max=65536
```

----------------
VERIFY ENTRIES 
----------------
```bash
sudo sysctl -p
```

------------------------------------
EXECUTE UPDATE AND UPGRADE COMMANDS
------------------------------------
```bash
sudo yum update
sudo yum upgrade
```

-----------------------
INSTALL DOCKER
-----------------------
```bash
sudo yum  install docker.io
docker --version
```

-------------------------
INSTALL DOCKER COMPOSE
-------------------------
```bash
sudo yum install docker compose -y
```

--------------------------------
CREATE DOCKER COMPOSE FILE
--------------------------------
```bash
sudo vi docker-compose.yml
```

--------------------------------------------------------------------
ADD FOLLOWING LINES TO DOCKER COMPOSE FILE AND SAVE WITH :WQ COMMAND
--------------------------------------------------------------------

```bash
version: "3"
services:
  sonarqube:
    image: sonarqube:community
    restart: unless-stopped
    depends_on:
db
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonar
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
sonarqube_data:/opt/sonarqube/data
sonarqube_extensions:/opt/sonarqube/extensions
sonarqube_logs:/opt/sonarqube/logs
    ports:
"9000:9000"
  db:
    image: postgres:12
    restart: unless-stopped
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
    volumes:
postgresql:/var/lib/postgresql
postgresql_data:/var/lib/postgresql/data
volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:
```
------------------------------
RUN DOCKER COMPOSE COMMAND 
------------------------------
```bash
sudo docker compose up -d
```
-----------------------------------------------------
ADD PORT NO TO FIREWALL
----------------------------------------------
```bash
firewall-cmd  --zone=public --add-port=9000/tcp --permanent
 firewall-cmd  --reload
 firewall-cmd  --zone=public --list-ports
 ```

------------------------------------------------------
MONITOR LOGS UNTILL YOU FIND SONARQUBE IS OPERATIONAL
-------------------------------------------------------
```bash
sudo docker-compose logs --follow
```

------------------------
WAIT FOR OUTPUT
 SonarQube is operational

GET ACCESS IN BROWSER 
```bash
192.168.157.133:9000
```

DEFAULT USERNAME: admin
DEFAULT PASSWORD: admin
UPDATE THE PASSWORD -> password

Generate Token in SonarQube
Account > MyAccount > Security > Generate Token 
Enter Token Name: test Type: User Token  Expires in: 30
token-*************


Project > Create a local Project > Project display name: test Project key: test Main branch name : main Next > Choose the baseline for new code for this project Use the global setting > Create Project 

How do you want to analyze your repository?
With Jenkins > Analyze your project with Jenkins GitHub 
Create a Jenkinsfile : Maven 

------------------------------------------------------------------------

Jenkins 
Jenkins Dashboard > Manage Jenkins > Plugins > Available plugins > SonarQube Scanner > Install & Restart 
Again Login to Jenkins Dashboard > Manage Jenkins > System(configuration global setting path) > SonarQube servers
If checked, job administrators will be able to inject a SonarQube server configuration as environment variables in the build.

Environment variables
SonarQube installations
List of SonarQube installations
Add SonarQube > Jenkins Credentials Provider: Jenkins
SonarQube installations
List of SonarQube installations
Name
sonar
Server URL
Default is http://localhost:9000
```bash
http://192.168.157.133:9000
```

**Server authentication token**
SonarQube authentication token. Mandatory when anonymous access is disabled.

sonar-token
Add
Advanced


Add Credentials
Domain

Global credentials (unrestricted)
Kind

Secret text
Scope
?

Global (Jenkins, nodes, items, all child items, etc)
Secret
••••••••••••••••••••••••••••••••••••••••••••
ID
?
sonar-token
Description
Add

