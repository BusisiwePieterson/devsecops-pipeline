# devsecops-pipeline

![images](images/0_qpYyrWiXHCZRtAAn.png)

This project aims to develop a robust and secure Continuous Integration/Continuous Deployment (CI/CD) pipeline that leverages several industry-standard tools to ensure high code quality, security, and operational efficiency. The pipeline integrates Jenkins, Trivy, SonarQube, ArgoCD, OWASP, Grafana, Prometheus, and email notifications to automate the build, test, and deployment processes while continuously monitoring and improving application performance and security.

### Benefits
- Improved Code Quality: Continuous integration of SonarQube ensures code quality standards.
- Enhanced Security: Trivy and OWASP tools provide comprehensive security checks.
- Automated Deployments: ArgoCD enables efficient and consistent application deployments.
- Proactive Monitoring: Prometheus and Grafana offer real-time insights into system performance.
Team Awareness: Email notifications keep the team informed about the pipeline status.

##


Step 1: Launch EC2 (Ubuntu 22.04):

- First, you'll have an EC2 instance running Ubuntu 22.04 with Jenkins, SonarQube, and Docker installed and configured. This setup will enable you to use Jenkins for CI/CD, SonarQube for code quality analysis, and Docker for containerized deployments.


![images](images/Screenshot_2.png)

Next, create an Elastic IP and attach it to the EC2 instance you created. This will ensure the instance retains the same IP address whenever it is stopped and started.

![images](images/Screenshot_3.png)

![images](images/Screenshot_4.png)

Now, Connect to the instance using SSH.

![images](images/Screenshot_5.png)

Clone the code that you will use for the project.

Run `git clone https://github.com/N4si/DevSecOps-Project.git`

![images](images/Screenshot_7.png)

Run `sudo apt-get update` to update all the packages.

![images](images/Screenshot_8.png)

### Install Docker and Run the App using a Container

On the EC2 instance, you will set up Docker by running the following script:

```
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

Then build and run your application using Docker containers:

```
docker build -t netflix .
docker run -d --name netflix -p 8081:80 netflix:latest

#to delete
docker stop <containerid>
docker rmi -f netflix
```

![images](images/Screenshot_9.png)

Run `docker images` to view a list of created images. You will see that the image was built successfully.

![images](images/Screenshot_11.png)

Next, to ensure that you have access to Jenkins, SonarQube, and your app, open the security groups. 

- The application is running on port **8081**.
- Jenkins on port **8080.**
- Sonarqube on port **9000**

![images](images/Screenshot_12.png)

Now that your application port is open, go to your browser to access the application. Enter `<ec2-public-ip>:8081` in the address bar.

![images](images/Screenshot_14.png)


You will notice that the movies are not showing because you need an API key.

**Get the API Key:**

- Open a web browser and navigate to TMDB (The Movie Database) website.
- Click on "Login" and create an account.
- Once logged in, go to your profile and select "Settings."
- Click on "API" from the left-side panel.
- Create a new API key by clicking "Create" and accepting the terms and conditions.
- Provide the required basic details and click "Submit."
- You will receive your TMDB API key.


![images](images/Screenshot_13.png)

Now recreate the Docker image with your api key:

`docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .`

![images](images/Screenshot_16.png)

![images](images/Screenshot_19.png)

#### Install SonarQube and Trivy


Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.

`docker run -d --name sonar -p 9000:9000 sonarqube:lts-community`


![images](images/Screenshot_21.png)

Access SonarQube in your browser.

![images](images/Screenshot_22.png)

### Install Trivy
Trivy is a vulnerability scanner for containers and other artifacts like operating systems or package managers. It helps to detect vulnerabilities in these artifacts by scanning them against a database of known security vulnerabilities. Trivy can be used to enhance the security of your containerized applications by identifying potential security risks early in the development lifecycle.

```
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy        
```

![images](images/Screenshot_23.png)

to scan image using trivy

![images](images/Screenshot_24.png)

### Install Jenkins for Automation

Now its time to install Jenkins, The integration of Jenkins with SonarQube helps teams maintain code quality standards and identify and address potential issues early in the development process.

Install Jenkins on the EC2 instance to automate deployment:

```
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
openjdk version "17.0.8" 2023-07-18
OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)

#jenkins
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

To check if Jenkins is running, you can use the following command: `sudo service jenkins status`

![images](images/Screenshot_25.png)

Open Jenkins in your browser by navigating to `<ec2-public-ip>:8080` and copy the command `cat /var/lib/jenkins/secrets/initialAdminPassword` to retrieve your Jenkins administrator password.

![images](images/Screenshot_26.png)

### Install Necessary Plugins in Jenkins

*Go to Manage Jenkins → Plugins → Available Plugins →*

![images](images/Screenshot_27.png)

**Install below plugins**

1. Eclipse Temurin Installer (Install without restart)

2. SonarQube Scanner (Install without restart)

3. NodeJs Plugin (Install Without restart)

4. Email Extension Plugin

![images](images/Screenshot_28.png)


### Configure Java and Nodejs in Global Tool Configuration

*Go to Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save*

![images](images/Screenshot_29.png)

![images](images/Screenshot_30.png)

###  Create SonarQube Token

*Go to Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text*. It should look like this.

After adding sonar token

Click on Apply and Save

The Configure System option is used in Jenkins to configure different server

Global Tool Configuration is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.

![images](images/Screenshot_31.png)

![images](images/Screenshot_32.png)

![images](images/Screenshot_33.png)

![images](images/Screenshot_34.png)

### Create a Jenkins webhook



Create a CI/CD pipeline in Jenkins to automate your application deployment.

![images](images/Screenshot_35.png)

Create a Jenkinsfile and define your pipeline within it.


![images](images/Screenshot_125.png)

Now, run your Jenkins pipeline and monitor SonarQube status to ensure your code meets quality and security standards.

![images](images/Screenshot_38.png)

![images](images/Screenshot_39.png)

**Next, install Dependency-Check and Docker Tools in Jenkins, follow these steps:**

1. Go to the "Dashboard" in your Jenkins web interface.
2. Navigate to "Manage Jenkins" → "Manage Plugins."
3. Click on the "Available" tab and search for "OWASP Dependency-Check."
4. Check the checkbox for "OWASP Dependency-Check" and click on the "Install without restart" button.

**Configure Dependency-Check Tool:**

1. After installing the Dependency-Check plugin, configure the tool.
2. Go to "Dashboard" → "Manage Jenkins" → "Global Tool Configuration."
3. Find the section for "OWASP Dependency-Check."
4. Add the tool's name, e.g., "DP-Check."
5. Save your settings.

**Install Docker Tools and Docker Plugins:**

- Go to "Dashboard" in your Jenkins web interface.
- Navigate to "Manage Jenkins" → "Manage Plugins."
- Click on the "Available" tab and search for "Docker."
- Check the following Docker-related plugins:
    - Docker
    - Docker Commons
    - Docker Pipeline
    - Docker API
    - docker-build-step
- Click on the "Install without restart" button to install these plugins.



![images](images/Screenshot_40.png)

**Add DockerHub Credentials:**

- To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:
- Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."
- Click on "System" and then "Global credentials (unrestricted)."
- Click on "Add Credentials" on the left side.
- Choose "Secret text" as the kind of credentials.
- Enter your DockerHub credentials (Username and Password) and give the - credentials an ID (e.g., "docker").
- Click "OK" to save your DockerHub credentials.

![images](images/Screenshot_41.png)

![images](images/Screenshot_42.png)

![images](images/Screenshot_43.png)

![images](images/Screenshot_44.png)

![images](images/Screenshot_45.png)

Now, you have installed the Dependency-Check plugin, configured the tool, and added Docker-related plugins along with your DockerHub credentials in Jenkins. You can now proceed with configuring your Jenkins pipeline to include these tools and credentials in your CI/CD process.

![images](images/Screenshot_126.png)

Run your Jenkis pipeline again

![images](images/Screenshot_49.png)

Now check DockerHub, you will see that the docker image was pushed to Docker Hub.


![images](images/Screenshot_48.png)

Access your application that is running in a Docker container by navigating to `<public-ip>:8081` in your browser.

![images](images/Screenshot_50.png)


   
### Prometheus and Grafana for Monitoring

Set up Prometheus and Grafana to monitor your application.

First, create a dedicated Linux user for Prometheus and download Prometheus:

```
sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
```



![images](images/Screenshot_52.png)

Extract Prometheus files, move them, and create directories:

```
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

![images](images/Screenshot_53.png)

Set ownership for directories:

```
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

Create a systemd unit configuration file for Prometheus:

```
sudo nano /etc/systemd/system/prometheus.service
```

![images](images/Screenshot_54.png)

Add the following content to the `prometheus.service` file:

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

![images](images/Screenshot_55.png)

Here's a brief explanation of the key parts in this `prometheus.service` file:

- `User` and `Group` specify the Linux user and group under which Prometheus will run.

- `ExecStart` is where you specify the Prometheus binary path, the location of the configuration file (`prometheus.yml`), the storage directory, and other settings.

- `web.listen-address` configures Prometheus to listen on all network interfaces on port 9090.

- `web.enable-lifecycle` allows for management of Prometheus through API calls.

Enable and start Prometheus:

```
sudo systemctl enable prometheus
sudo systemctl start prometheus
```
Verify Prometheus's status:

```
sudo systemctl status prometheus
```

![images](images/Screenshot_56.png)

Go to your security groups and open port 9090 for Prometheus.

![images](images/Screenshot_57.png)

You can access Prometheus in a web browser using your server's IP and port 9090:

`http://<your-server-ip>:9090`

![images](images/Screenshot_58.png)

### Installing Node Exporter

Create a system user for Node Exporter and download Node Exporter:

```
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
```

![images](images/Screenshot_59.png)

Extract Node Exporter files, move the binary, and clean up:

```
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
```

![images](images/Screenshot_60.png)

Add the following content to the `node_exporter.service` file:

Replace `--collector.logind` with any additional flags as needed.

Enable and start Node Exporter:

![images](images/Screenshot_61.png)

Enable and start Node Exporter:

```
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

Verify the Node Exporter's status:

```
sudo systemctl status node_exporter
```

![images](images/Screenshot_62.png)

You can access Node Exporter metrics in Prometheus.

#### Configure Prometheus Plugin Intergration

Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

**Prometheus Configuration:**

To configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the `prometheus.yml` file. Here is an example `prometheus.yml` configuration for your setup:

Make sure to replace `<your-jenkins-ip>` and `<your-jenkins-port>` with the appropriate values for your Jenkins setup.

![images](images/Screenshot_63.png)

Check the validity of the configuration file:

```
promtool check config /etc/prometheus/prometheus.yml
```

Reload the Prometheus configuration without restarting:

```
curl -X POST http://localhost:9090/-/reload
```

You can access Prometheus targets at `http://<your-prometheus-ip>:9090/targets`


![images](images/Screenshot_65.png)

Go to your Security Groups and open port **9091** for node-exporter

![images](images/Screenshot_64.png)


![images](images/Screenshot_66.png)

### Grafana Installation

Install Grafana on Ubuntu 22.04 and Set it up to Work with Prometheus

Install Dependencies:

First, ensure that all necessary dependencies are installed:
```
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
```


**Add the GPG key for Grafana:**

```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

![images](images/Screenshot_67.png)

Open port `3000` for grafana

![images](images/Screenshot_68.png)

**Add Grafana Repository:**

Add the repository for Grafana stable releases:

```
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

Update the package list and install Grafana:

```
sudo apt-get update
sudo apt-get -y install grafana
```

![images](images/Screenshot_69.png)

Enable and Start Grafana Service:

To automatically start Grafana after a reboot, enable the service:

```
sudo systemctl enable grafana-server
```

Then, start Grafana:

```
sudo systemctl start grafana-server
```

![images](images/Screenshot_70.png)

Open a web browser and navigate to Grafana using your server's IP address. The default port for Grafana is 3000. `http://<your-server-ip>:3000`

You'll be prompted to log in to Grafana. The default username is "admin," and the default password is also "admin."

Change the Default Password:

When you log in for the first time, Grafana will prompt you to change the default password for security reasons. Follow the prompts to set a new password.


### Add Prometheus Data Source:

![images](images/Screenshot_71.png)

To visualize metrics, you need to add a data source. Follow these steps:

Click on the gear icon (⚙️) in the left sidebar to open the "Configuration" menu.

Select "Data Sources."

Click on the "Add data source" button.

Choose "Prometheus" as the data source type.

In the "HTTP" section:

Set the "URL" to http://localhost:9090 (assuming Prometheus is running on the same server).
Click the "Save & Test" button to ensure the data source is working.



![images](images/Screenshot_72.png)

![images](images/Screenshot_73.png)

### Import a Dashboard:

To make it easier to view metrics, you can import a pre-configured dashboard. Follow these steps:

- Click on the "+" (plus) icon in the left sidebar to open the "Create" menu.

- Select "Dashboard."

- Click on the "Import" dashboard option.

- Enter the dashboard code you want to import (e.g., code 1860).

- Click the "Load" button.

![images](images/Screenshot_74.png)

- Select the data source you added (Prometheus) from the dropdown.

- Click on the "Import" button.

![images](images/Screenshot_75.png)

You should now have a Grafana dashboard set up to visualize metrics from Prometheus.

Grafana is a powerful tool for creating visualizations and dashboards, and you can further customize it to suit your specific monitoring needs.

That's it! You've successfully installed and set up Grafana to work with Prometheus for monitoring and visualization.

![images](images/Screenshot_76.png)

### Configure Prometheus Plugin Integration:

Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

![images](images/Screenshot_77.png)

To configure Prometheus to scrape metrics from Jenkins, you need to modify the prometheus.yml file. 


![images](images/Screenshot_79.png)

Check the validity of the configuration file:
```
promtool check config /etc/prometheus/prometheus.yml
```
Then Reload the Prometheus configuration without restarting:

```
curl -X POST http://localhost:9090/-/reload
```
![images](images/Screenshot_81.png)

You will now see that your Jenkins pipeline is a Target

![images](images/Screenshot_82.png)

Import  dashboard for Jenkins

![images](images/Screenshot_83.png)

You should now be able to monitor your Jenkins pipeline.

### Notification

Our program works well; however, we manually have to check Jenkins to know the status of our pipeline, which is not very ideal. To improve this process, we will now add email notifications to our project. 


![images](images/Screenshot_87.png)

![images](images/Screenshot_88.png)

![images](images/Screenshot_89.png)

![images](images/Screenshot_90.png)

![images](images/Screenshot_91.png)

![images](images/Screenshot_92.png)

![images](images/Screenshot_93.png)

![images](images/Screenshot_94.png)

![images](images/Screenshot_95.png)

![images](images/Screenshot_96.png)

![images](images/Screenshot_97.png)

![images](images/Screenshot_98.png)

![images](images/Screenshot_99.png)

## Kubernetes Cluster with Nodegroups

In this phase, you'll set up a Kubernetes cluster with node groups. This will provide a scalable environment to deploy and manage your applications.

First you will need to install awscli and eksctl

To install the AWS CLI, run the following commands.

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Install and Setup eksctl on Ubuntu Server

```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv /tmp/eksctl /usr/local/bin
```


![images](images/Screenshot_100.png)

Now that awscli is installed, you will need to create an AWS access key ID and secret access key in order to interact with AWS resources.

![images](images/Screenshot_101.png)

Now it is time to create our cluster

![images](images/Screenshot_102.png)

You can confirm that the cluster has been created by checking your AWS console.

![images](images/Screenshot_103.png)

Next, we need to install and Setup kubectl on Ubuntu Server.

```
sudo curl --silent --location -o /usr/local/bin/kubectl   https://s3.us-west-2.amazonaws.com/amazon-eks/1.22.6/2022-03-09/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl 

kubectl version --short --client
```

![images](images/Screenshot_104.png)

Create a namespace `kubectl create namespace argocd`

![images](images/Screenshot_105.png)

### Monitor Kubernetes with Prometheus

Prometheus is a powerful monitoring and alerting toolkit, and you'll use it to monitor your Kubernetes cluster. Additionally, you'll install the node exporter using Helm to collect metrics from your cluster nodes.

#### Install Node Exporter using Helm

To begin monitoring your Kubernetes cluster, you'll install the Prometheus Node Exporter. This component allows you to collect system-level metrics from your cluster nodes. Here are the steps to install the Node Exporter using Helm:

1. Add the Prometheus Community Helm repository:



```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

2. Create a Kubernetes namespace for the Node Exporter:

```
kubectl create namespace prometheus-node-exporter
```

3. Install the Node Exporter using Helm:

```
helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter
```

![images](images/Screenshot_109.png)

Now see if the node exporter namespace has been created.

![images](images/Screenshot_110.png)

Add a Job to Scrape Metrics on nodeip:9001/metrics in prometheus.yml:

Update your Prometheus configuration (prometheus.yml) to add a new job for scraping metrics from nodeip:9001/metrics. You can do this by adding the following configuration to your prometheus.yml file:

```
  - job_name: 'Netflix'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['node1Ip:9100']
```

Replace 'your-job-name' with a descriptive name for your job. The static_configs section specifies the targets to scrape metrics from, and in this case, it's set to nodeip:9001.

Don't forget to reload or restart Prometheus to apply these changes to your configuration.

To deploy an application with ArgoCD, you can follow these steps, which I'll outline in Markdown format:

### Deploy Application with ArgoCD

1. Install ArgoCD:

You can install ArgoCD on your Kubernetes cluster by following the instructions provided in the EKS Workshop documentation https://archive.eksworkshop.com/intermediate/290_argocd/install/.



![images](images/Screenshot_111.png)

Get your password for ArgoCD

![images](images/Screenshot_112.png)

### Deploy Application with ArgoCD

2. Set Your GitHub Repository as a Source:

After installing ArgoCD, you need to set up your GitHub repository as a source for your application deployment. This typically involves configuring the connection to your repository and defining the source for your ArgoCD application. The specific steps will depend on your setup and requirements.

![images](images/Screenshot_113.png)

3. Create an ArgoCD Application:

- name: Set the name for your application.
destination: Define the destination where your application should be deployed.
- project: Specify the project the application belongs to.
- source: Set the source of your application, including the GitHub repository URL, revision, and the path to the application within the repository.
- syncPolicy: Configure the sync policy, including automatic syncing, pruning, and self-healing.

![images](images/Screenshot_114.png)

![images](images/Screenshot_115.png)

![images](images/Screenshot_116.png)


![images](images/Screenshot_117.png)

![images](images/Screenshot_118.png)

![images](images/Screenshot_119.png)

![images](images/Screenshot_120.png)

Access your Application

To Access the app make sure port 30007 is open in your security group and then open a new tab paste your `NodeIP:30007`, your app should be running.


![images](images/Screenshot_121.png)

To monitor our application open port 9100


![images](images/Screenshot_122.png)

Next, configure Prometheus to scrape metrics from Kubernetes, you need to modify the prometheus.yml file. 

![images](images/Screenshot_123.png)

![images](images/Screenshot_124.png)


### Cleanup AWS EC2 Instances:
Terminate AWS EC2 instances that are no longer needed.

## THE END!!


