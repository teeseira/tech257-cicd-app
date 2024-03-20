# CI/DI with Jenkins

### Add SSH Public Key to GitHub Repo
Deploy SSH key to local repo  not entire Github Account
Allow write access

- Jenkins > `New item` > Give a desired name > choose `Freestyle project` > `OK`.
- Configure the following:
  ![alt text](image-5.png)
- Tick the box for `GitHub project`, then enter the **Project url** (this is the GitHub Repo URL - HTTPS one).
  ![alt text](image-8.png)
- Select `Restrict where this project can be run` and provide the `Label expression` (name of the agent node).
  ![alt text](image-6.png)
- For Source Code Managment: select `Git` and provide the **Repository URL** (this time the SSH one because we would like it to be secure).
- Jenkins will try to sent a request to GitHub, so you must now provide `Credentials`.
    - Click `Add` > `Jenkins`
    - Kind: `SSH Username with private key` > Give a `Username` e.g. *my_key* > click `Add` to store value for private key > then `Add`.
        > Note for private key include the start and end e.g. from where it says `-----BEGIN OPENSSH PRIVATE KEY-----`
        <br> Results:<br>
        ![alt text](image-10.png)


