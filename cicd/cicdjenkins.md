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
  
## Integrate Continuous Delivery

### Update the payload URL

Update the payload URL, and push a code change to ensure CI is still working.

<img src="../assets/image-18.png" >

### Create a "dev" branch, update the Jenkins job, and test
