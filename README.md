# Deploying a Book Management App to AWS

In this guide, we will walk you through the process of deploying a Book Management app, which is a REST API built using express.js, to Amazon Web Services (AWS). The app will be deployed in an Auto-Scaling Group behind an Application Load Balancer (ALB), and it will be managed by systemd. Additionally, we'll show you how to add an endpoint that returns the EC2 instance's IP address.

## Prerequisites

Before you begin, make sure you have the following:

- An AWS account
- Basic knowledge of AWS services
- Git installed on your local machine

## Step 1: GitHub Release

1. Create a public GitHub repository for your project if you haven't already.
2. Clone the repository to your local machine using `git clone`.
3. Navigate to the app's directory and build the app by running:
   
   ```bash
   npm install
   npm run dev
   ```
4. Package the app using the following command:
``` 
tar czvf app.tar.gz package.json package-lock.json dist 
```

5. Go to your repository on GitHub and click on the "Releases" tab.

6. Click on the "Draft a new release" button.

7. Provide a version tag and release title, then upload the app.tar.gz file you created earlier.

8. Publish the release.   

## Step 2 : Create Security Group
1. Go to the AWS Management Console.
2. Navigate to the EC2 Dashboard.
3. In the left sidebar, under "Security Groups," click on "Create Security Group."
4. Configure inbound rules to allow HTTP/HTTPS traffic (port 80 and 443)
5. configure outbound rules as needed.
6. Create the security group. 

## Step 3: Create User Data Bash Script

1. Create a bash script named user_data.sh.

2. Add the following content to the script:
```
#!/bin/bash

sudo apt update 

curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash - &&\

sudo apt-get install -y nodejs

npm install -g npm

cd /home/ubuntu
sudo git clone https://github.com/MostafaTahboub/Book-Management-App.git  Book-Management-App

cd Book-Management-App
npm install express

echo "[Unit]
Description=Book Management App
After=network.target

[Service]
ExecStart=/usr/bin/node ./dist/index.js
WorkingDirectory=/home/ubuntu/Book-Management-App
Restart=always
User=ubuntu

[Install]
WantedBy=multi-user.target" | sudo tee /etc/systemd/system/book-management-app.service

sudo systemctl enable book-management-app
sudo systemctl start book-management-app
```
3. please make sure to replace a paths with your needs .

## Step 4: Launch Template

1. Navigate to the EC2 Dashboard.
2. Under "Instances," click on "Launch Templates."
3. Click on the "Create launch template" button.
4. Configure the template, including instance type, key pair, and user data.
5. In the "Advanced Details" section, add a user data script (the user_data.sh script you created earlier).
6. Create the launch template.

## Step 5: Auto Scaling and Load Balancing
1. Navigate to the EC2 Dashboard.
2. Under "Auto Scaling Groups," click on "Create Auto Scaling group."
3. Select the launch template you created.
4. Configure the Auto Scaling group settings, including desired capacity and scaling policies.
5. Add tags as needed.
6. Review and create the Auto Scaling group.

## Step 6: IP Address Endpoint
1. In your app code, create a new route that returns the instance's private IP address.

2. For example, using express.js:
```
app.get('/get-ip', (req, res) => {
  const ip = req.socket.remoteAddress;
  res.json({ ip });
});
```

## Steps by screenshots 

[screenshots for steps](screenshots/images)

## Conclusion

Congratulations! You have successfully deployed your Book Management app to AWS using Auto Scaling and Load Balancing. Your app is now accessible through the ALB and can handle varying levels of traffic while ensuring high availability.

Feel free to customize and expand this deployment process according to your project's needs. If you encounter any issues, refer to the AWS documentation for further assistance.
