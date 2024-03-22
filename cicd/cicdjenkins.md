# CI/DI with Jenkins

## Architectural Design

<br><img src="../assets/image0.png" width=600px>

## Integrate Continuous Integration

### Add SSH Public Key to GitHub Repo
- Create new SSH key on local machine
- Copy the output of the public key, then deploy the key to your desired GitHub repository (not entire Github Account).
  > Note: Make sure to `Allow write access`

### Setup Jenkins
<!-- - Install Jenkins on your server. You can download it from the official website and follow installation instructions. -->
   - Access Jenkins through your browser by navigating to `http://<Jenkins-Server-IP>:8080/`. <!--http://35.176.97.54:8080/ -->

- Jenkins > `New Item` > Give a desired name > choose `Freestyle project` > `OK`.

- For `General`:
  - Provide a `Description`, tick box for `Discard old builds` and have a Max # of build to keep as `3`.
  <br><img src="../assets/image-5.png" width=600px>
  - Tick the box for `GitHub project`, then enter the **Project url** (this is the GitHub Repo URL - HTTPS one).
  <br><img src="../assets/image-8.png" width=400px>
  

- For `Office 365 Connector`:
  - Select `Restrict where this project can be run` and provide the `Label expression` (name of the agent node).
  <br><img src="../assets/image-6.png" width=550px>

- For `Source Code Managment`:
  - Select `Git` and provide the **Repository URL** (this time the SSH one because we would like it to be secure).
  - Jenkins will try to sent a request to GitHub, so you must now provide `Credentials`.
    - Click `Add` > `Jenkins`
    - Kind: `SSH Username with private key` > Give a `Username` e.g. *my_key* > click `Add` to store value for private key > then `Add`.
        > Note for private key include the start and end e.g. from where it says `-----BEGIN OPENSSH PRIVATE KEY-----`
        <br> Results:
        <br><img src="../assets/image-10.png" width=550px>
  - Branch specifier: `*/main`

- For `Build environment`:
  <br><img src="../assets/image-9.png" width=500px>
- For `Build`:
  <br><img src="../assets/image-12.png" width=300>

- Click `Save`

### Trigger the Job

- Click `Build Now`.
  <br><img src="../assets/image-13.png" width=600px>

- If unsuccessful review `Build History` > and `Console Output`.

## Web Hook

Create a webhook on GitHub, set up continuous integration using Jenkins, and ensure that changes to your repository trigger the CI pipeline automatically.

### Create Webhook on GitHub
- Go to your GitHub repository.
- Click on "Settings" tab.
- Choose "Webhooks" from the sidebar.
- Click on "Add webhook".
- Enter the payload URL as `http://35.176.97.54:8080/github-webhook/`.
- Set the Content type to `application/json`.
- Choose the events that should trigger the webhook: `Just the push event`.
- Click on "Add webhook" to save the settings.

<br><img src="../assets/image-14.png" width=500px>

### Test the CI

- Configure the Jenkins jobs [like this](#integrate-continuous-integration).
  - But for Build Triggers, select `GitHub hook triger fr GITScm polling`.
    <br><img src="../assets/image-15.png" width=600px>
- Now test the CI:
   - Make a change to your README file in the GitHub repository.
   - Commit and push the change to GitHub.
   - GitHub will trigger the webhook, which in turn will trigger the Jenkins job.
   - Monitor Jenkins to ensure that the job executes successfully.
    <br><img src="../assets/image-16.png" width=200px>
    <br><img src="../assets/image-17.png" width=600px>
  
## Integrate Merge

### Update the payload URL

- In your GitHub repo settings, <!--find the webhook you've set up for Jenkins, and --> update the Payload URL<!-- to the appropriate Jenkins webhook URL. This URL should be the endpoint where Jenkins listens for webhook events-->.
  <br><img src="../assets/image-18.png" width=500>
- Commit a change locally and push it to GitHub.
- Check that Jenkins receives the webhook payload and triggers the CI job. <!-- You can check the Jenkins job's build history or console output to confirm. -->

### Create a "dev" branch, update the Jenkins job, and test

   - Create a new branch named "dev" on local machine: `git checkout -b dev`.
   - Push the "dev" branch to your GitHub repo: `git push origin dev`.
   - In your Jenkins job for the CI, update the branch to build from `*/dev`.
      <br><img src="../assets/image-19.png" width=500>
   - Commit a change on the "dev" branch and push it to GitHub: `git push origin dev`.
   - Check that Jenkins triggers the job.
   <br><img src="../assets/image-20.png" width=500>
  
### Create a new Jenkins job for merge from dev to main branch

- Jenkins dashboard > "New Item" e.g. `My-CI-Merge` > Freestyle project > OK.
- For `General`, give Description: e.g. _merge from dev to main after successful tests_ > tick `Discard old build` > Max #: of builds to keep: `3` > tick `GitHub project` > provide **Project url** (HTTPS one).
- For `Office 365 Connector`, tick `Restrict where this project can be run` and provide the Label expression (agent node).
- For `Source Code Managment`, select `Git` > provide **Repository URL** (SSH one) > Branch specifier: `*/main`.
- For `Build Triggers`, tick `Build after other projects are built` and provide the Projects to watch e.g. `My-CI`.
   <br><img src="../assets/image-21.png">
- For `Build Triggers`, tick `Build after other projects are built` and provide the Projects to watch e.g. `My-CI`.
   <br><img src="../assets/image-21.png">
- For `Post-build Actions` choose `Git Publisher`, then the following:
  <br><img src="../assets/image-22.png" width=500>
  >This will execute a Git merge from "dev" to "main".
- `Save` the job config.

#### Test the setup on Jenkins

- Click `Build Now` on the previous Jenkins job "My-CI".
- Ensure that the "My-CI-Merge" job is triggered by the successful completion of your previous CI job.

#### Test the setup on GitHub

- Push a change to the "dev" branch (`git push origin dev`) and verify that Jenkins triggers the "My-CI-Merge" job to merge the changes into the main branch.

## Integrate Continuous Delivery (with AWS)

### Create an Amazon EC2 Instance

- AWS Management Console > EC2 > Launch instance > Name e.g. `my-tech257-cd-app` > choose the Community AMI provided:
  <br><img src="../assets/image-23.png" width=900>
- Instance type: t2.micro > choose Key pair > choose existing security group (which allows for ports 22, 80 and 3000).
- In Advanced details, enter the user data:

  ```
  sudo apt update -y
  sudo apt upgrade -y
  ```
- `Launch instance`.
  <br><img src="../assets/image-24.png" width=550>
  <br><img src="../assets/image-24.png" width=550>

### Create new job on Jenkins

- Create a new Job called `My-CD`, for example.
- For `General`, give Description: e.g. _merge CD with AWS_ > tick `Discard old build` > Max #: of builds to keep: `3` > tick `GitHub project` > provide **Project url** (HTTPS one).
- For `Source Code Managment`, select `Git` > provide **Repository URL** (SSH one) > Branch specifier: `*/main`.
For `Build Triggers`, tick `Build after other projects are built` and provide the Projects to watch e.g. `My-CD-Merge`.
For `Build Triggers`, tick `Build after other projects are built` and provide the Projects to watch e.g. `My-CD-Merge`.
- For `Build Environment` tick `Provide Node & npm bin/ folder to PATH` and `SSH agent`
  - In the SSH agent add credentials > Kind: SSH Username with private key > Username: pemkey > Add private key (this is the value of the AWS **.pem** file) > Add.
- For `Build` choose `Execute shell`, then enter:
  ```bash
  ```bash
  ssh -o "StrictHostKeyChecking=no" ubuntu@3.250.0.106 <<EOF
      sudo apt update -y
      sudo apt upgrade -y
      sudo apt install nginx -y
  EOF
  ```
  > This will connect to the specified server, execute the listed commands (update, upgrade, and install nginx), and then close the SSH connection.

### Verify deployment on AWS

- Check the Public IP of the app's EC2 instance:
  <br><img src="../assets/image-25.png" width=550>

## Normal Proxy
You want to be able to access the application on its port number.

### Automated method

- Go to CD Jenkins Job.
- For `Build` > `Execute shell`, update the script to:
  ```bash
  # Get app and env folders from git root directory to home directory of VM
  rsync -avz -e "ssh -o StrictHostKeyChecking=no" app ubuntu@<PublicIP>:/home/ubuntu/
  rsync -avz -e "ssh -o StrictHostKeyChecking=no" environment ubuntu@<PublicIP>:/home/ubuntu/

  ssh -o "StrictHostKeyChecking=no" ubuntu@<PublicIP> <<EOF
      sudo apt-get update -y
      sudo apt-get upgrade -y
      sudo apt-get install nginx -y
      sudo systemctl restart nginx 
      
      # Find and execute the script
      cd environment/app/
      sudo -n chmod +x provision.sh
      ./provision.sh
      
      # Install dependencies and start the app
      cd ..
      cd ..
      cd app
      sudo apt-get install npm -y
      npm install
  ```
  >Note: Ensure the aws security group allows ssh to jenkins ip. Ensure file.pem provided to Jenkins. Ensure ec2 is running.  > This will connect to the specified server, execute the listed commands (update, upgrade, and install nginx), and then close the SSH connection.

### Verify deployment on AWS

- Check the Public IP of the app's EC2 instance:
  <br><img src="../assets/image-25.png" width=550>

## Normal Proxy
You want to be able to access the application on its port number.

### Automated method

- Go to CD Jenkins Job.
- For `Build` > `Execute shell`, update the script to:
  ```
  # Get app and environment folders on home directory of VM
  rsync -avz -e "ssh -o StrictHostKeyChecking=no" app ubuntu@<PublicIP>:/home/ubuntu/
  rsync -avz -e "ssh -o StrictHostKeyChecking=no" environment ubuntu@<PublicIP>:/home/ubuntu/

  ssh -o "StrictHostKeyChecking=no" ubuntu@<PublicIP> <<EOF
      sudo apt-get update -y
      sudo apt-get upgrade -y
      sudo apt-get install nginx -y
      sudo systemctl restart nginx 
      
      # Find and execute the script
      cd environment/app/
      sudo -n chmod +x provision.sh
      ./provision.sh
      
      # Install dependencies and start the app
      cd ..
      cd ..
      cd app
      sudo apt-get install npm -y
      npm install
  ```
  >Note: Ensure the aws security group allows ssh to jenkins ip. Ensure file.pem provided to Jenkins. Ensure ec2 is running.
- `Save`.
- Click `Build Now` on the first Jenkins CI job.

#### Check on AWS
- Connect and SSH into the application's EC2 instance.
- Run `ls` to see the **app** and **environment** folders.
- Navigate into the app folder and run `npm start` to start application.
  <br><img src="../assets/image-26.png" width=400>
- Check the Public IP of the app's EC2 instance without and with its port number:
  <br><img src="../assets/image-25.png" width=550>
  <br><img src="../assets/image-27.png" width=550>

### Key Takeaways

- Best practice is to not make changes in the main branch.
- Only do changes/testing in the `dev` branch.
- Command to checkout the dev branch:
  ```
  git checkout dev
  ```
- Git Workflow:
  ```
  git add .
  git commit -m "[your-message]"
  git push origin dev   
  ```
  > Then merge changes into main branch by creating a pull request or by using Git Bash.

<!--
#### Git Troubleshooting:

- If you're currently on the dev branch and you want to directly merge changes from dev branch into the main branch:
  - Switch to main branch: `git checkout main`
  - Merge chnages from dev into main `git merge dev`
  - Fix conflicts and commit the merge: `git commit -m "Merge changes from dev into main"`
  - Push changes to remote: `git push origin main`
- Click `Build Now` on the first Jenkins CI job.

#### Check on AWS
- Connect and SSH into the application's EC2 instance.
- Run `ls` to see the **app** and **environment** folders.
- Navigate into the app folder and run `npm start` to start application.
  <br><img src="../assets/image-26.png" width=400>
- Check the Public IP of the app's EC2 instance without and with its port number:
  <br><img src="../assets/image-25.png" width=550>
  <br><img src="../assets/image-27.png" width=550>

## Key Takeaways

- Best practice is to not make changes in the main branch.
- Only do changes/testing in the `dev` branch.
- Command to checkout the dev branch:
  ```
  git checkout dev
  ```
- Git Workflow:
  ```
  git add .
  git commit -m "[your-message]"
  git push origin dev   
  ```
  > Then merge changes into main branch by creating a pull request or by using Git Bash.

<!--
#### Git Troubleshooting:

- If you're currently on the dev branch and you want to directly merge changes from dev branch into the main branch:
  - Switch to main branch: `git checkout main`
  - Merge chnages from dev into main `git merge dev`
  - Fix conflicts and commit the merge: `git commit -m "Merge changes from dev into main"`
  - Push changes to remote: `git push origin main`

  >  >Key takeway: with these steps you can avoid manually creating a pull request on GitHub.

- If you are a couple commits **behind** main, you can merge changes from main into dev:
  - Go to dev branch: `git checkout dev`
  - Fetch latest changes from the remote repo: `git fetch origin`
  - Merge chnages from origin/main into dev branch: `git merge origin/main`
  - Resolve conflicts
  - Complete merge: `git commit -m "Merge origin/main into dev"`
  - Push merged changes to remote dev branch: `git psuh origin dev`

- If you are a couple commits **ahead of** main you can merge changes from dev into  main:
  - Go to main branch: `git checkout main`
  - Merge dev into: `git merge dev`
  - Resolve conflicts
  - Push changes to remote main branch: `git push origin main`
-->

## Deploy Application Changes on AWS

### Architecture diagram
<img src="../assets/jenkins.png" width=500>

### Review the new IP address
On AWS, every time you stop and start an instance the Public IP address changes.

- On AWS Management console > EC2 > Start instance for application VM.
- Review new IP.
  <br><img src="../assets/image-28.png" width=500>

### Update Jenkins job

- Update the Jenkins CD job:
  ```bash
  # Replace with new app folder
  rsync -avz -e "ssh -o StrictHostKeyChecking=no" app ubuntu@3.253.33.155:/home/ubuntu/

  # SSH into the VM and execute commands
  ssh -o "StrictHostKeyChecking=no" ubuntu@3.253.33.155 <<EOF
      # Navigate to the app directory
      cd ~/app

      # Stop existing Node.js processes
      sudo pkill -f node

      # Install dependencies
      npm install
      
      # Launch the app
      nohup node app.js > /dev/null 2>&1 &
  EOF
  ```
  <br><img src="../assets/image-29.png" width=550>

- `Build now` only on CD stage.
  <br><img src="../assets/image-30.png" width=400>

  > You can also `Build Now` from the CI stage to ensure the Pipeline works:
  <br><img src="../assets/image-31.png" width=500>
- Check deployment on AWS.
  <br><img src="../assets/image-32.png" width=400>

### Apply code changes
- Check you're on testing branch (dev).
- Change the heading of app and save:
  <br><img src="../assets/image-33.png" width=400>
- Push changes to dev branch.
  ```
  git add .
  git commit -m "Code change"
  git push origin dev
  ```
- Merge with main branch.
  ```
  git checkout main
  git merge dev
  git commit -m "Merge changes from dev branch into main"
  git push origin main
  ```

### Verify deployment on AWS

- Yan see the code change of adding "2024":
  <br><img src="../assets/image-34.png" width=400>