Experiment 1 — Docker: Multi-Container Application
20 marks
Aim: Deploy the Metro Reservation application using Tomcat and MySQL in separate Docker containers.
Step 1. Open the project
Open PowerShell:
cd C:\Users\Delta\Downloads\metro-reservation


Check that you have pom.xml, src and target.
Step 2. Build the WAR file
mvn clean package -DskipTests


Check:
dir target


You should see a .war file, for example:

metro-reservation-1.0.war
Step 3. Create Dockerfile
Inside the project folder, create a file named Dockerfile.

Paste:
FROM tomcat:9.0
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war


Explanation:
- FROM downloads the Tomcat image.
- COPY copies the WAR into Tomcat.
- ROOT.war deploys the application at the root URL.
Make sure there is only one WAR file in target.
Step 4. Create docker-compose.yml
In the same folder, create docker-compose.yml.

Paste:
services:
  app:
    build: .
    ports:
      - "8082:8080"
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: metro
      MYSQL_USER: metro
      MYSQL_PASSWORD: metro123
    volumes:
      - data:/var/lib/mysql

volumes:
  data:


Explanation:
Component	Purpose
app	Runs the Tomcat application
db	Runs MySQL
ports	Maps host port 8082 to container port 8080
depends_on	Starts the database service before the application
volumes	Preserves MySQL data

Step 5. Build and run containers
docker compose up -d --build


Step 6. Verify containers
docker compose ps


You should see both app and db running.
Step 7. Open application
Open:

http://localhost:8082/

Because the WAR is named ROOT.war, the application is deployed at the root URL.
Step 8. Verify MySQL
docker compose exec db mysql -u metro -pmetro123 -e "SHOW DATABASES;"


You should see the metro database.
Step 9. Check logs
docker compose logs


Step 10. Stop containers
docker compose down


This removes the containers but keeps the named database volume.

To start again:
docker compose up -d



Experiment 1 — Important scenario questions
Question	Answer
MySQL container stops?	Restart using docker compose restart db.
Preserve data after deletion?	Use Docker volumes.
Why separate containers?	Independent deployment, scaling and isolation.
Prevent unauthorized access?	Strong passwords, restricted ports and database users.
Database connectivity fails?	Check logs, hostname, credentials, ports and network.

Important database connection details:

If the Java application has JDBC configuration, use:
Setting	Value
Host	db
Port	3306
Database	metro
Username	metro
Password	metro123
JDBC URL	jdbc:mysql://db:3306/metro

The Compose file creates the database, but the Java application must also contain the MySQL JDBC driver and connection configuration to actually use it. depends_on does not guarantee MySQL is ready to accept connections.

Experiment 2 — Jenkins Freestyle Project
20 marks
Aim: Configure Jenkins to fetch a project from GitHub, build it using Maven and generate a WAR file.

GitHub repository:

https://github.com/deepthisagar7/metro-reservation.git
Step 1. Open Jenkins
Open:

http://localhost:8080

Log in to Jenkins.
Step 2. Install plugins
Go to:

Manage Jenkins → Plugins → Available plugins

Install:
- Git
- Maven Integration
If already installed, skip this step.
Step 3. Configure JDK
Go to:

Manage Jenkins → Tools → JDK installations
1. Click Add JDK.
2. Name: JDK17
3. Enable automatic installation if needed.
4. Save.
If JDK 17 is already configured, reuse it.
Step 4. Configure Maven
Under:

Manage Jenkins → Tools → Maven installations
1. Click Add Maven.
2. Name: Maven3
3. Enable Install automatically.
4. Choose an available version.
5. Save.
Step 5. Create a Freestyle project
1. Open Jenkins Dashboard.
2. Click New Item.
3. Name: Metro-Reservation.
4. Select Freestyle project.
5. Click OK.
Step 6. Configure GitHub
Under Source Code Management:
1. Select Git.
2. Enter repository URL:
https://github.com/deepthisagar7/metro-reservation.git


3. Leave credentials empty if the repository is public.
4. Branch:
*/main


If the repository uses master, select */master instead.
Step 7. Configure JDK and Maven
Select the configured JDK JDK17 if your Jenkins job provides a JDK selection.

Under Build Steps:
1. Click Add build step.
2. Select Invoke top-level Maven targets.
3. Maven Version: Maven3.
4. Goals:
clean package -DskipTests


Step 8. Save and build
1. Click Save.
2. Click Build Now.
3. Wait for the build to finish.
Step 9. Verify output
Open:

Build History → Build Number → Console Output

Look for:
BUILD SUCCESS


The generated WAR file should be in:
target/metro-reservation-1.0.war


Experiment 2 complete
Experiment 2 — Important scenario questions
Question	Answer
Build fails?	Check Console Output and fix errors.
GitHub changes not reflected?	Trigger a new build or configure SCM polling/webhooks.
Maven not found?	Configure Maven under Manage Jenkins → Tools.
JDK version error?	Configure the correct JDK.
Where is WAR generated?	Inside the target folder.

Remember this workflow:
1. Install Git and Maven plugins.
2. Configure JDK and Maven.
3. Create Freestyle project.
4. Configure GitHub repository.
5. Configure Maven build step.
6. Enter clean package -DskipTests.
7. Build Now → Console Output → BUILD SUCCESS.


Experiment 3 — Nagios Monitoring using Docker
20 marks
Aim: Run Nagios using Docker and monitor server resources and HTTP service availability.

You already have Nagios running with the correct port mapping:

8084:80

Your current Docker setup shows:
jasonrivers/nagios:latest
0.0.0.0:8084->80/tcp


Step 1. Pull Nagios image
docker pull jasonrivers/nagios:latest


Step 2. Run Nagios container
If Nagios is already running, skip this step.

Otherwise:
docker run -d --name nagios -p 8084:80 jasonrivers/nagios:latest


Step 3. Verify container
docker ps


You should see Nagios running with port mapping:
8084->80


Step 4. Open Nagios dashboard
Open:

http://localhost:8084

Default credentials for this image:
Username	Password
nagiosadmin	nagios

Step 5. View monitoring dashboard
After login:
1. Click Tactical Overview.
2. Click Hosts to view monitored hosts.
3. Click Services to view monitored services.
4. Check their status.








howtoforge.com


Step 6. Monitor CPU, memory and disk
Nagios uses monitoring plugins to check system resources.
Resource	What it checks
CPU	Processor utilization
Memory	RAM usage
Disk	Available disk space
Uptime	Time since server started

These checks require suitable plugins and host configuration. Starting the Nagios container alone does not configure them automatically.
Step 7. Configure HTTP monitoring
Nagios uses the check_http plugin to check whether a web server responds.

Example service configuration:
define service {
    use                 generic-service
    host_name           localhost
    service_description HTTP
    check_command       check_http
}


This is an example configuration; localhost must correspond to a configured Nagios host, and the HTTP plugin must be installed.

For your Metro Reservation application, the web server is available on port 8082 if Experiment 1 is running. From inside the Nagios container, use host.docker.internal to reach the Windows host.
Step 8. Configure alerts
Nagios can send notifications when a service or host fails.

Examples:
- Server is DOWN.
- HTTP service is unavailable.
- CPU usage is too high.
- Disk space is low.
Notifications require configured contacts, notification commands and thresholds.
Step 9. Stop and restart Nagios
Stop:
docker stop nagios


Start:
docker start nagios


Remove:
docker rm -f nagios


Experiment 3 complete
Experiment 3 — Important scenario questions
Question	Answer
Application server is down?	Nagios reports the host as DOWN and can alert the administrator.
HTTP service unavailable?	check_http reports a CRITICAL state.
Disk space low?	Configure disk monitoring with warning and critical thresholds.
CPU usage high?	Configure CPU monitoring plugins and thresholds.
Why use Nagios?	Continuous monitoring and early detection of infrastructure failures.

====

# **Complete Software Engineering Lab Practical Guide**

All four experiments from your lab manual, with setup, commands, code, execution, verification, and cleanup.

We'll use your Windows 11 + PowerShell + Docker Desktop setup. The instructions are kept as short and exam-friendly as possible.

## **Experiments and marks**

| Experiment                              | Marks   |
| --------------------------------------- | ------- |
| 1. Jenkins Pipeline Automation          | 30      |
| 2. Kubernetes Deployment                | 30      |
| 3. Dockerfile for Maven Web Application | 20      |
| 4. AWS EC2 Deployment                   | 20      |
| **Total**                               | **100** |

# **Experiment 1 – Jenkins Pipeline Automation (30 Marks)**

**Aim:** Automate checkout, build, testing, and artifact archiving using a Jenkins pipeline.

Already completed successfully

## **Step 1: Start Jenkins**

Jenkins is installed as a Windows service. Check it:

```
Get-Service Jenkins
```

If it is stopped:

```
Start-Service Jenkins
```

Open:

[Jenkins Dashboard](http://localhost:8080?utm_source=chatgpt.com)

## **Step 2: Check prerequisites**

```
java -version
mvn -version
git --version
```

Jenkins needs Java, Maven, and Git available to its service environment. If you change the system PATH, restart Jenkins from Administrator PowerShell:

```
Restart-Service Jenkins
```

## **Step 3: Create a Jenkins pipeline**

1. Open Jenkins dashboard.
2. Click **New Item**.
3. Enter `FoodDeliveryPipeline`.
4. Select **Pipeline**.
5. Click **OK**.
6. Scroll to the Pipeline section.
7. Select **Pipeline script**.

## **Step 4: Enter the pipeline script**

```

node {
    stage('Checkout') {
        git 'https://github.com/sitakameshwari/online-food-delivery.git'
    }

    stage('Build') {
        bat 'mvn clean package -DskipTests'
    }

    stage('Test') {
        bat 'mvn test'
    }

    stage('Archive') {
        archiveArtifacts artifacts: 'target/*.war'
    }
}

```

Click **Save**.

## **Step 5: Execute**

1. Click **Build Now**.
2. Click the build number.
3. Open **Console Output**.
4. Check that all four stages succeed.

## **Step 6: Verify artifact**

Open the successful build and select **Artifacts**. You should see the generated WAR file.

**Remember:** Checkout → Build → Test → Archive.

# **Experiment 2 – Kubernetes Deployment (30 Marks)**

**Aim:** Create, deploy, scale, and manage a Kubernetes deployment using Minikube.

## **Step 1: Start Minikube**

Open PowerShell:

```
minikube start --driver=docker
```

Wait until the cluster starts.

## **Step 2: Verify Kubernetes**

```
kubectl get nodes
```

Expected: a node named `minikube` with status `Ready`.

## **Step 3: Create a deployment YAML**

Create a working folder:

```
mkdir C:\KubernetesLab
cd C:\KubernetesLab
notepad deployment.yaml
```

Paste this short YAML:

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: food
spec:
  replicas: 3
  selector:
    matchLabels:
      app: food
  template:
    metadata:
      labels:
        app: food
    spec:
      containers:
      - name: nginx
        image: nginx

```

Save the file.

**YAML indentation:** Use spaces, never tabs.

## **Step 4: Deploy**

```
kubectl apply -f deployment.yaml
```

## **Step 5: Check deployment and pods**

```
kubectl get deployments
kubectl get pods
```

You should see one deployment and three pods.

## **Step 6: Describe deployment**

```
kubectl describe deployment food
```

This displays deployment details, replica status, labels, and events.

## **Step 7: Scale the deployment**

Increase the number of pods to five:

```
kubectl scale deployment food --replicas=5
```

Verify:

```
kubectl get pods
```

You should now see five pods.

## **Step 8: Update the container image**

```
kubectl set image deployment/food nginx=nginx:1.27
```

Check rollout:

```
kubectl rollout status deployment/food
```

## **Step 9: View logs**

Get the pod names:

```
kubectl get pods
```

Copy one pod name and run:

```
kubectl logs POD_NAME
```

Replace `POD_NAME` with the actual name.

## **Step 10: Delete deployment**

```
kubectl delete deployment food
```

Verify:

```
kubectl get deployments
kubectl get pods
```

## **Optional: Open Kubernetes Dashboard**

If the practical asks you to show the Kubernetes dashboard:

```
minikube dashboard
```

Minikube should open the dashboard in your browser.

**Remember:** Start → Apply → Get → Describe → Scale → Set image → Logs → Delete.

You do not need a `service.yaml` when the question only asks for a Deployment.

# **Experiment 3 – Dockerfile for Maven Web Application (20 Marks)**

**Aim:** Create a Docker image for a Maven-based web application and run it in a container using Tomcat.

## **Step 1: Clone project**

```
cd C:\
git clone https://github.com/sitakameshwari/online-food-delivery.git
cd online-food-delivery
```

If already cloned, navigate into its existing directory instead.

## **Step 2: Build Maven project**

```
mvn clean package
```

Check the WAR file:

```
dir target\*.war
```

## **Step 3: Create Dockerfile**

```
notepad Dockerfile
```

Paste:

```

FROM tomcat:9
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]

```

Save it in the same directory as `pom.xml`.

## **Step 4: Build Docker image**

```
docker build -t foodapp .
```

Verify:

```
docker images
```

## **Step 5: Run container**

```
docker run -d -p 8081:8080 --name foodcontainer foodapp
```

We use host port 8081 because Jenkins is using 8080.

## **Step 6: Verify**

```
docker ps
docker logs foodcontainer
```

Open:

[http://localhost:8081](http://localhost:8081?utm_source=chatgpt.com)

## **Step 7: Stop and remove**

```
docker stop foodcontainer
docker rm foodcontainer
```

## **Step 8: Push image to Docker Hub (if required)**

```
docker login
docker tag foodapp YOUR_USERNAME/foodapp:v1
docker push YOUR_USERNAME/foodapp:v1
```

Replace `YOUR_USERNAME` with your Docker Hub username.

**Remember:** Maven build → Dockerfile → Build image → Run container → Verify.

# **Experiment 4 – Deploy Web Application on AWS EC2 (20 Marks)**

**Aim:** Create an AWS EC2 instance and deploy a web application using Docker.

We'll use Docker on Ubuntu EC2 rather than `apt install tomcat9`. This avoids the Tomcat package availability issue you encountered earlier.

## **Step 1: Create an AWS account**

1. Open [AWS website](https://aws.amazon.com/?utm_source=chatgpt.com).
2. Click **Create an AWS Account**.
3. Enter your email address and account name.
4. Verify your email.
5. Create a password.
6. Enter contact information.
7. Add a payment method as requested.
8. Complete phone verification.
9. Select the support plan offered during signup.
10. Sign in to the AWS Management Console.

AWS may request payment details even for accounts using Free Tier offers. Check current pricing and eligibility before launching resources.

## **Step 2: Launch an EC2 instance**

1. In the AWS Console, search for **EC2**.
2. Open the EC2 dashboard.
3. Click **Launch instance**.
4. Enter the instance name: `FoodDeliveryServer`.
5. Select **Ubuntu Server** as the AMI.
6. Choose an eligible small instance type, such as `t3.micro` if available under your account's offer.
7. Create a key pair:
   - Name: `food-key`
   - Type: RSA
   - Format: `.pem`
8. Download and safely keep the key file.

## **Step 3: Configure security group**

Under Network settings, create or select a security group.

Add these inbound rules:

| Type       | Port | Source    |
| ---------- | ---- | --------- |
| SSH        | 22   | My IP     |
| Custom TCP | 8081 | 0.0.0.0/0 |

Port 8081 will be used to access the deployed application. Avoid opening SSH to everyone.

Click **Launch instance**.

## **Step 4: Connect to EC2 from Windows**

1. Open EC2 → Instances.
2. Select `FoodDeliveryServer`.
3. Copy its **Public IPv4 address**.
4. Open PowerShell and navigate to where your `.pem` file is saved.

For example:

```
cd C:\Users\YOUR_USERNAME\Downloads
```

Connect:

```
ssh -i food-key.pem ubuntu@PUBLIC_IP
```

Replace `PUBLIC_IP` with your instance's public IP address.

Type `yes` if asked to trust the host.

You are now connected to the Ubuntu server.

## **Step 5: Install Docker on EC2**

Run these commands inside the Ubuntu terminal:

```
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

Verify:

```
docker --version
```

## **Step 6: Get the application**

There are two ways to do this. For the simplest repeatable practical, build the image on your Windows machine and push it to Docker Hub first.

On your Windows machine, inside the project directory:

```
mvn clean package
docker build -t foodapp .
docker login
```

Tag and push the image:

```
docker tag foodapp YOUR_USERNAME/foodapp:v1
docker push YOUR_USERNAME/foodapp:v1
```

Replace `YOUR_USERNAME` with your Docker Hub username.

## **Step 7: Pull and run the image on EC2**

Back in the Ubuntu SSH terminal:

```
sudo docker pull YOUR_USERNAME/foodapp:v1
```

Run the container:

```
sudo docker run -d -p 8081:8080 --name foodcontainer YOUR_USERNAME/foodapp:v1
```

## **Step 8: Verify deployment**

```
sudo docker ps
sudo docker logs foodcontainer
```

Copy the EC2 Public IPv4 address and open this URL in your browser:

`http://PUBLIC_IP:8081`

If the application loads, deployment is successful.

## **Step 9: Stop and remove the container**

```
sudo docker stop foodcontainer
sudo docker rm foodcontainer
```

When you finish the practical, **stop or terminate the EC2 instance** to avoid unexpected charges. Terminate it if you no longer need it.

**Remember:** Create account → Launch EC2 → Configure security group → SSH → Install Docker → Pull image → Run container → Open public IP.

# **Final revision sheet – all four experiments**

These are the main commands to memorize before the practical.

## **1. Jenkins**

```
node {
    stage('Checkout') {
        git 'https://github.com/sitakameshwari/online-food-delivery.git'
    }
    stage('Build') {
        bat 'mvn clean package -DskipTests'
    }
    stage('Test') {
        bat 'mvn test'
    }
    stage('Archive') {
        archiveArtifacts artifacts: 'target/*.war'
    }
}
```

**Flow:** Checkout → Build → Test → Archive.

## **2. Kubernetes**

```
minikube start --driver=docker
kubectl get nodes
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods
kubectl describe deployment food
kubectl scale deployment food --replicas=5
kubectl set image deployment/food nginx=nginx:1.27
kubectl rollout status deployment/food
kubectl logs POD_NAME
kubectl delete deployment food
```

**Flow:** Start → Apply → Get → Describe → Scale → Update → Logs → Delete.

## **3. Docker**

**Dockerfile:**

```
FROM tomcat:9
COPY target/*.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
```

**Commands:**

```
mvn clean package
docker build -t foodapp .
docker images
docker run -d -p 8081:8080 --name foodcontainer foodapp
docker ps
docker logs foodcontainer
docker stop foodcontainer
docker rm foodcontainer
```

**Flow:** Build WAR → Create Dockerfile → Build image → Run container → Verify.

## **4. AWS EC2**

**Ubuntu commands:**

```
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
docker --version
sudo docker pull YOUR_USERNAME/foodapp:v1
sudo docker run -d -p 8081:8080 --name foodcontainer YOUR_USERNAME/foodapp:v1
sudo docker ps
sudo docker logs foodcontainer
```

**Flow:** AWS account → EC2 instance → Security group → SSH → Install Docker → Pull image → Run container → Access public IP.

## **What to prepare before the practical**

**Your machine setup**

-

  Ready

  &#x20;Java 21



-

  Ready

  &#x20;Maven



-

  Ready

  &#x20;Git



-

  Ready

  &#x20;Jenkins



-

  Ready

  &#x20;Docker Desktop



-

  Installed

  &#x20;kubectl



-

  Installed

  &#x20;Minikube



-

  Account needed

  &#x20;AWS account and EC2 access

  **Important files to keep**

- `deployment.yaml`

- `Dockerfile`

- Jenkins pipeline script

- `food-key.pem` (for AWS)

**Two important cautions:**

1. The Dockerfile using `tomcat:9` assumes your application is compatible with Tomcat 9. If it uses Jakarta EE 9+, it may need Tomcat 10 instead.
2. AWS Free Tier eligibility and credits depend on account type and current offers. Check the estimated charges before launching; terminate the instance when finished.

You have the complete procedure for all four experiments now. The Jenkins experiment is already working; the remaining hands-on verification is Kubernetes, Docker, and AWS.
