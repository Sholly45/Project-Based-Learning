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
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 5BA31D57EF5975CA
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

In this part, you will learn how to configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.

1. Enable webhooks in your GitHub repository settings
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/9.PNG)

2. Go to Jenkins web console, click “New Item” and create a “Freestyle project”
![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/16.PNG)

3. Click "Configure" your job/project and add these two configurations

Configure triggering the job from GitHub webhook:

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/10.PNG)

Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".

**click on add post-build actions and click "Archive the Artifact" then save**

Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.


![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/11.PNG)


we have now successfully configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

By default, the artifacts are stored on Jenkins server locally, note the build_number is the value of the build created

**`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`**




**CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH**


**Step 3 – Configure Jenkins to copy files to NFS server via SSH**


Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to `/mnt/apps` directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called `"Publish Over SSH"`.

1. Install "Publish Over SSH" plugin.

On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

On "Available" tab search for **"Publish Over SSH"** plugin and install it

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/13.PNG)



2. Configure the job/project to copy artifacts over to NFS server.

On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:

* Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)

* Arbitrary name

* Hostname – can be `private IP address` of the NFS server

* Username – `ec2-user` (since NFS server is based on EC2 with RHEL 8)

* Remote directory – `/mnt/apps` since our Web Servers use it as a mointing point to retrieve files from the NFS server


Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/14.PNG)


Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"




Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories – so we use **.


Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.


Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:

```
SSH: Transferred 25 file(s)
Finished: SUCCESS
```

To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file

```
cat /mnt/apps/README.md
```
If you see the changes you had previously made in your GitHub – the job works as expected.


If you are having issues with the build failing or been unstable, follow the steps below to correct such.

* In your NFS server change permissions for the /mnt/apps directory

```
sudo chmod 777 /mnt/apps
sudo chown nobody:nobody /mnt/apps
sudo chmod -R 777 /mnt
sudo chown -R nobody:nobody /mnt
```

![image 1](https://github.com/Sholly45/Project-Based-Learning/blob/main/Project%209/images/18.PNG)


## THANK YOU!
   







