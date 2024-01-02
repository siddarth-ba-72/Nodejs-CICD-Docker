# CI/CD of Nodejs Using GitHub Actions, EC2 and Docker containers

## Development
- Develop the Application
- Push on GitHub

## EC2 Instance
- Create an AWS EC2 instance
- Add an actions runner in GitHub to connect to aws-ec2 instance
- Go to `Settings` -> `Actions` Tab -> `Runner` -> `New self-hosted runner` -> Select `Linux`
- In Your EC2 dashboard go to the instance -> Click on `Instance id` -> `Connect` -> `Connect`
- After connecting you can access the ec2 linux terminal
- Follow the instructions given below:-
```
mkdir actions-runner && cd actions-runner
```
```
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz
```
- Install `libicu` and `.NET Core 6.O` dependencies on ec2 terminal. Remember the Amazon-Linux has the Fedora dsitribution
```
sudo dnf install libicu
```
```
sudo rpm -Uvh https://packages.microsoft.com/config/rhel/8/packages-microsoft-prod.rpm
```
- For further requirements we need `docker`
```
sudo yum update -y
sudo yum -y install docker
```
- Start Docker
```
sudo service docker start
```
- Now we need to provide access to docker by going to root user. You can create the duplicate of this browser tab and run these
```
sudo su
chmod 666 /var/run/docker.sock
```
- Back to ec2 instance Extract the installer
```
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz
```
- Create Runner and Start
```
./config.sh --url https://github.com/Budget-Sales-Analysis/Budget_Sales_Analysis_Ineuron --token AUEFA43YF3GF2PHNHKMJ5OLFSQGJ6
```
- Give the name as `aws-ec2`
```
./run.sh
```

## Dockerization
- Create a repository on [Docker Hub](https://hub.docker.com)
- In your project terminal
```
docker build -t <image_name> .
```
```
docker tag <image_name>:latest <repository_name>
```
```
docker push <repository_name>
```
- This will push your remote image on Docker Hub

## CI/CD Pipeline
- Go tp your project root directory and create `.github/workflows/cicd_workflow.yml` file
- Go to `Settings` -> `Security tab` -> `Secrets and Variables` -> `Actions` -> `New repository secret`
- Create two new secret variables
  - `DOCKER_USERNAME`: Your Docker Hub username
  - `DOCKER_PASSWORD`: Your Docker Hub Personal Access Token
- Copy the below content to this file
```
name: CICD

on:
  push:
    branches: [master]

jobs:
  build:
    runs-on: [ubuntu-latest]
    steps:
      - name: Checkout source
        uses: actions/checkout@v3
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      - name: Build docker image
        run: docker build -t <repository_name> .
      - name: Publish image to docker hub
        run: docker push <repository_name>

  deploy:
    needs: build
    runs-on: [aws-ec2]
    steps:
      - name: Pull image from docker hub
        run: docker pull <repository_name>
      - name: Delete old container
        run: docker rm -f nodejs-app-container
      - name: Run docker container
        run: docker run -d -p 5000:5000 --name nodejs-app-container <repository_name>

```
- Push the code to GitHub






