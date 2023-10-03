## TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION.
-----
Here is how the updated architecture will look like upon competition of this project:

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/1.png)

**Step 1 – Install Jenkins server**

1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
2. Install JDK (since Jenkins is a Java-based application)

```
sudo apt update
sudo apt install default-jdk-headless
```

3. **Install Jenkins**

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/2.PNG)
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/3.PNG)

Make sure Jenkins is up and running:

**`sudo systemctl status jenkins`**

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/4.PNG)

4. By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
5. Perform initial Jenkins setup.

From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/5.PNG)
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/6.PNG)
You will be prompted to provide a default admin password



Retrieve it from your server:

**`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`**

Then you will be asked which plugins to install – choose suggested plugins.

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/7.PNG)

Once plugins installation is done – create an admin user and you will get your Jenkins server address.

The installation is completed!
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/8.PNG)

**Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks**

In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings
[image 6)

2. Go to Jenkins web console, click “New Item” and create a “Freestyle project”
3. 
   







