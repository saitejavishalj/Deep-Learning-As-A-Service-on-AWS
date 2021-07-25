# Deep-Learning-As-A-Service-on-AWS

The AWS SDK Libraries were used to build this application, which was written in JavaScript with node.js and express JS.
All of the system's tiers are built from the ground up with Node.js and ExpressJs. 

### Handing asynchronous calls:
Because all AWS requests are asynchronous, they must be handled in a synchronous manner.
By resolving promises and call backs, we structured the code utilizing Async and await features. 

Crontab:
During an EC2 reboot or start-up, it is used to perform the shell command.
On EC2 app-tier starting, this crontab aids in the automatic launch of App-tier functionalities. 

Bash Scripting:
In the web-tier and app-tier, we employed bash scripting.
In the Web-tier, we used Bash scripting to configure Controller (which is always up and running), and in the App-tier, we used Bash scripting to configure Run (which starts app-tier functionalities). 

### 1. Web Tier
a. Merger.js:
This code uploads the files to an S3 input bucket, and if the upload succeeds, it adds the image ID to the input SQS queue. 

b. Create_ec2.js (LOAD BALANCER):
This module implements the auto-scaling algorithm used in the previous module.
The length of the SQS queue is determined, and the app-tier instances are auto-scaled accordingly.
This code produces EC2 instances based on the SQS queue size, which determines user demand.
This module is in charge of scaling up the application (increasing the number of EC2 instances). 

c. sqs_fetch.js:
This module receives the key (image ID) from the output sqs queue and the projected output from the output S3 bucket for that image ID. 

d. Controller.sh:
This is a shell script that runs in the Web-tier all of the time.
This script's main role is to examine the input traffic and then run merger.js, sqs fetch.js, and create ec2.js based on the input traffic. 

e. App.js:
On the web-tier, this file is needed to operate the backend node.js server. 

f. Index.html:
This is used to show the user interface for uploading photographs and displaying the output anticipated results. 

### 2. App-tier:
a. App_tier.js:
This module extracts the picture from the S3 bucket using the message ID of each incoming message from the input queue.
It then starts a python classification image process and gets the expected output, as well as putting the message ID in the SQS output queue and the predicted output (as Key-Value pairs) in the output S3 bucket, (the message ID is stored in the SQS output queue, indicating that the process is finished.)
The message is now removed from the SQS input queue.
This step is continued until there are no more message IDs in the input SQS queue, at which point the instance will self-terminate; this capability ensures our elastic application's automated scale-out feature. 

b. Run.sh:
This is used to initiate the app-tier capabilities and is run by the crontab on system startup or reboot. 

c. Package.json:
This file provides all of the requirements needed to run the node.js app on the app-tier. 

Detail on how to install your programs and how to execute them: 

App-tier:
We'll need to make an app-tier AMI.
To do so, we'll need to build a new EC2 instance with the AMI Id supplied in the project requirement and a Python image classification model.
Our code snippets app tier.js, run.sh, and package.json must now be pushed. 
To install all node.js dependencies, use the NPM install and NPM install AWS-SDK commands.
We'll also need to edit the EC2 instance's crontab to have the run.sh script execute on startup or reboot.
To retrieve the AMI id, we must first build an AMI for the app-tier setup. 

Web-tier:
Install node.js dependencies and AWS-SKD on a new EC2 instance created from a Linux AMI.
Copy all of the Web-tier code files to the EC2 instance, start the node.js server with the ‘node app.js' command, and execute the Controller with “Bash Controller.sh” in a separate terminal.
Get the web-public tier's IP address and connect to the UI using the public IP's Port:3000. 


# RESULTS
BATCH SIZE (NO OF IMAGES) | NON-ELASTIC APPLICATION | ELASTIC APPLICATION
| :--- | ---: | :---:
10  |6 minutes	|4 minutes
30	|11 minutes	|5 minutes
