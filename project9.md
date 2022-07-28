Step 1 – Jenkins server's Installation

Created an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

![Screenshot 2022-07-28 at 11 22 26](https://user-images.githubusercontent.com/79456052/181483344-11f6f6ae-6c4f-4777-975b-105092c6b44e.png)

Installed JDK using the command *sudo apt update & sudo apt install default-jdk-headless* and confirmed that jenkins was running using the command *sudo systemctl status jenkins*

![1](https://user-images.githubusercontent.com/79456052/181481293-909c3a1c-a0b7-49a1-aa06-1f83d0cf1001.png)

![2](https://user-images.githubusercontent.com/79456052/181481465-d98c6a84-ac81-4501-9db8-146a6d7ed3a1.png)

![3](https://user-images.githubusercontent.com/79456052/181481483-02966dae-204e-4bcf-b96a-04a59c56ac25.png)

Opened up TCP port 8080 on the Jenkins server by creating a new Inbound Rule in the EC2's Security Group

![Screenshot 2022-07-28 at 11 25 09](https://user-images.githubusercontent.com/79456052/181483968-fdb3b3cf-65a8-4a51-b903-a20e8ea4f871.png)

Performed the initial Jenkins setup, from the  browser access i.e http://44.201.149.20:8080, I provided a default admin password using the *sudo cat/var/lib/jenkins/secrets/initialAdminPassword* command

![4](https://user-images.githubusercontent.com/79456052/181485287-aae80492-1b20-4ca1-9dfc-a151fe1b7eaf.png)

Installed the plugins, – created an admin user and obtained a Jenkins server address completeing the  jenkins installation. 

![5](https://user-images.githubusercontent.com/79456052/181484438-3ebe4cd3-5e17-462b-847c-2bf701406c79.png)

Step 2 – Jenkins Configuration -Retrieval of source codes from GitHub using Webhooks

Enabled webhooks in my  GitHub repository settings, clicked on "New Item" and created a "Freestyle project" under the jenkins web console.

![8](https://user-images.githubusercontent.com/79456052/181490277-1bcf060c-16ff-4bcb-872b-cf39083d1ffe.png)

Connected to the GitHub repository, by providing its URL from the repository itself.

![Screenshot 2022-07-28 at 12 05 28](https://user-images.githubusercontent.com/79456052/181491009-4bf98e44-2047-4648-9343-41574a89b549.png)

In configuration of the Jenkins freestyle project under source code management, choose Git repository, provide there the link to the Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

![10](https://user-images.githubusercontent.com/79456052/181491649-8f9ff196-8d6d-49ad-bdbf-ef0b91b24d39.png)


Saved the configuration and  the build. Clicked on the "Build Now" button. Opened the build and checked in "Console Output" if it had run successfully.

![Screenshot 2022-07-28 at 12 34 44](https://user-images.githubusercontent.com/79456052/181497567-46a84342-7e59-468d-bd65-3f56c91d9245.png)

Clicked "Configure" project9 and added the configurations triggering the job from GitHub webhook:
and "Post-build Actions" to archive all the files.


Made changes in the README.MD file in my GitHub repository and pushed the changes to the master branch.
A new build was launched automatically (by webhook) and the results – artifacts, saved on Jenkins server.

![Screenshot 2022-07-28 at 12 53 34](https://user-images.githubusercontent.com/79456052/181498764-698be02e-1ab3-4fd4-9a73-549896eed8cd.png)

#Step 3 – Configure Jenkins to copy files to NFS server via SSH#

With the artifacts saved locally on Jenkins server, copied the artifacts to the NFS server to /mnt/apps directory.

Installed "Publish Over SSH" plugin, on the main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item. On "Available" tab searched for "Publish Over SSH" plugin and installed it

![11](https://user-images.githubusercontent.com/79456052/181505212-09d90fc1-4fd0-462e-bd01-7b36af7f96f0.png)

![12](https://user-images.githubusercontent.com/79456052/181505352-91615033-97fc-4cbc-b1c9-fae575646cf1.png)

On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

Scrolled down to Publish over SSH plugin configuration section and configured it to be able to connect to my NFS server:

Provided a private key (content of opeyemi.pem file used to connect to NFS server via SSH)
Arbitrary name-NFS
Hostname – private IP address of the NFS server i.e 172.31.83.108
Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
Remote directory – /mnt/apps since our Web Servers uses it as a mointing point to retrieve files from the NFS server


Tested the configuration and made sure the connection returned Success. confirmed, that TCP port 22 on NFS server was open to receive SSH connections. 


![13](https://user-images.githubusercontent.com/79456052/181505552-9e39645f-9313-4462-a906-7a079a6392fb.png)

![14](https://user-images.githubusercontent.com/79456052/181506808-68b9f1ce-484e-4032-9a0a-4973976ce547.png)


Saved the configuration, opened Jenkins job/project9 configuration page and added another one "Post-build Action" Configured it to send all files produced by the build into our previous define remote directory. In this case we want to copy all files and directories – so we use **.

![15](https://user-images.githubusercontent.com/79456052/181506433-572e6d39-f415-4f75-a09c-bcbde5701918.png)

Saved the configuration and added a phrase "CHANGES MADE TO THE README FILE" README.MD file in the GitHub Tooling repository,connected via SSH to the NFS server and checked the README.MD file using the command *cat /mnt/apps/README.md* to confirm the changes made in my github.

![Screenshot 2022-07-28 at 15 27 43](https://user-images.githubusercontent.com/79456052/181546641-704a5bd6-4fcd-4e80-9c84-9a177cc60d70.png)








