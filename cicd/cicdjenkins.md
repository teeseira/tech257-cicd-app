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
  <img src="../assets/image-18.png" >
- Commit a change locally and push it to GitHub.
- Check that Jenkins receives the webhook payload and triggers the CI job. <!-- You can check the Jenkins job's build history or console output to confirm. -->

### Create a "dev" branch, update the Jenkins job, and test

   - Create a new branch named "dev" on local machine: `git checkout -b dev`.
   - Push the "dev" branch to your GitHub repo: `git push origin dev`.
   - In your Jenkins job for the CI, update the branch to build from `*/dev`.
      <br><img src="../assets/image-19.png" >
   - Commit a change on the "dev" branch and push it to GitHub: `git push origin dev`.
   - Check that Jenkins triggers the job.
   <br><img src="../assets/image-20.png">
  
### Create a new Jenkins job for merge from dev to main branch

- Jenkins dashboard > "New Item" e.g. `My-CI-Merge` > Freestyle project > OK.
- For `General`, give Description: e.g. _merge from dev to main after successful tests_ > tick `Discard old build` > Max #: of builds to keep: `3` > tick `GitHub project` > provide **Project url** (HTTPS one).
- For `Office 365 Connector`, tick `Restrict where this project can be run` and provide the Label expression (agent node).
- For `Source Code Managment`, select `Git` > provide **Repository URL** (SSH one) > Branch specifier: `*/main`.
- For `Build Triggers`, tick `Build after other projects are built` and provide the Projects to watch e.g. `My-CI`.
   <br><img src="../assets/image-21.png">
- For `Build Environment` tick `Provide Node & npm bin/ folder to PATH`.
- For `Post-build Actions` choose `Git Publisher`, then the following:
  <br><img src="../assets/image-22.png">
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
  <br><img src="../assets/image-23.png">
- Instance type: t2.micro > choose Key pair > choose existing security group (which allows for ports 22, 80 and 3000).
- In Advanced details, enter the user data:

  ```
  sudo apt update -y
  sudo apt upgrade -y
  ```
- `Launch instance`.

### Create new job on Jenkins

- Create a new Job called `My-CD`, for example.
- For `General`, give Description: e.g. _merge CD with AWS_ > tick `Discard old build` > Max #: of builds to keep: `3` > tick `GitHub project` > provide **Project url** (HTTPS one).
- For `Source Code Managment`, select `Git` > provide **Repository URL** (SSH one) > Branch specifier: `*/main`.
<!-- For `Build Triggers`, tick `Build after other projects are built` and provide the Projects to watch e.g. `My-CD-Merge`. we will build manually first-->
- For `Build Environment` tick `Provide Node & npm bin/ folder to PATH` and `SSH agent`
  - In the SSH agent add credentials > Kind: SSH Username with private key > Username: pemkey > Add private key (this is the value of the AWS **.pem** file) > Add.
- For `Build` choose `Execute shell`, then enter:

  ```
  # ensure the aws security group allows ssh to jenkins ip
  # ensure file.pem provided to Jenkins
  # ensure ec2 is running
  ssh -o "StrictHostKeyChecking=no" ubuntu@3.250.0.106 <<EOF
      sudo apt update -y
      sudo apt upgrade -y
      sudo apt install nginx -y
  EOF
  ```

Then switch to main brancg and do git pull

```
git checkout main
git pull origin main
```
