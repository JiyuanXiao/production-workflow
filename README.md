# What is this repo for?

This is a demenstration (and a memo note for myself) of a project's continuous integration and deployment setup.

In this demenstration, we used TravisCI as the continuous integration service provider, and AWS as the deployment platform.

If you are fresh on CI and deployment and you want to learn about them, hope this repo could provide valueable info to you.

# What are the files above?

The files in this repo is the final outcome of the CI and Deployment process, except the `docker-compse.yml` and `docker-compse-dev.yml`.

`docker-compse.yml` and `docker-compse-dev.yml` are only used for simplfy the process of docker images testing.

# Get Start

## Create a React Project

1. Make sure your computer has Node.js installed. If Node.js has been installed, run the following command in your terminal

```bash
node -v
```

- a version number should be printed out on following your command. Otherwise, please [install Node.js](https://nodejs.org/en/download) on your computer beforehand.

2. Create a React project:

```bash
npx create-react-app my-project
```

- after the creation is finished, open your project:

```bash
cd ./my-project
```

3. Try to start up and test your development server:

```bash
npm run start
```

- by running the command above, a development version of your website should pop up automatically

```bash
npm run test
```

- by running the command above, a default test suite should be run and the test result should be printed out on your terminal.

**Now we can automate the the start-up and test by using Docker**

## Create a Docker Image for Project Development

1. Make sure you has Docker Desktop installed in your computer. If not, please download and install [Docker Desktop](https://www.docker.com/products/docker-desktop/) beforehand.

2. To create a custom Docker image for project development, first create a Dockerfile named `Dockerfile.dev` and include the following code into the file (the file name `Dockerfile` is reserved for production build):

```dockerfile
FROM node:alpine

WORKDIR '/app'

COPY package.json .
RUN npm install

COPY . .

CMD ["npm", "run", "start"]
```

3. To build Docker image created above:

```bash
docker build -f Dockerfile.dev -t my-project .
```

4. To run the Docker Container:

```bash
docker run -p 8080:3000 my-project
```

- and open the websie http://localhost:8080, you should be able to see your webside on your browser.

## Create a Docker Image for Production Workflow

1. Create a Dockerfile named `Dockerfile` and include the following code into the file:

```dockerfile
FROM node:alpine as builder
WORKDIR '/app'
COPY ./package.json .
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
EXPOSE 80
COPY --from=builder /app/build /usr/share/nginx/html
```

2. you can test your image by running following command:

```bash
docker build -t my-production .
docker run -p 8080:80 my-production
```

- and open the websie http://localhost:8080, you should be able to see your webside on your browser.

**The purpose of this image is to make the production build on the AWS clould server at the later steps**

## Github Setup

1. Make sure you have a Github account, and you have access permission on your local computer. If not, please [create a Github Account](https://docs.github.com/en/get-started/start-your-journey/creating-an-account-on-github) and [connect your local computer to your github account with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

2. Make sure you have Git installed in your local computer. If not, please [install Git](https://git-scm.com/downloads).

3. Open your Github account and click the `+` button on the right upper corner of the webpage, and then in the dropdown click `New repository`.

4. Enter your project name under the `Repository name` blank.

5. Choose `Public`

6. Click `Create Repository

7. Inside the `Quick setup` box, click `SSH` button and copy the Github repo link following the `SSH` button. It should look like

`git@github.com:YourGithubName/RepoName.git`

8. Go back to your ternimal and make sure you are in the root directory of your project,

9. Run:

```bash
git init
git add .
git commit -m "initial commit"
```

10. Run:

```bash
git remote add origin $YOUR_GITHUB_REPO_LINK
```

- where `$YOUR_GITHUB_REPO_LINK` shoule be the link you copy from step#7

11. Run:

```bash
git  push origin master
```

12. Go back to your Github repo webpage and refresh your page, all of your project files should be shown on your repo page.

## Travis CI Setup

1. Go to [Travis-ci.com](https://www.travis-ci.com/) and Sign up with GitHub.

2. Accept the Authorization of Travis CI. You’ll be redirected to GitHub.

3. Click on your profile picture in the top right of your Travis Dashboard, click `Settings`, and then the green `Activate` button, and select the repositories you want to use with Travis CI. Or click the Activate all repositories using GitHub Apps button on the Getting Started page to activate all your repos.

4. Add a `.travis.yml` file to your repository to tell Travis CI what to do. The following example specifies a React Project which runs under a customized docker container:

```yml
sudo: required
services:
  - docker

before_install:
  - docker build -t image -f Dockerfile.dev .

script:
  - docker run -e CI=true image npm run test
```

By following instructions above, Travis CI will:

- Get the root user permission so that Docker can be run in the server
- Install docker to its server
- Build a custom Docker image using `Dokcerfile.dev`
- Run the Docker container and run our test suite on the container

## AWS Elastic Beanstalk Configuration

### Create EC2 IAM Instance Profile

1. Go to AWS Management Console

2. Search for `IAM` and click the IAM Service.

3. Click `Roles` under `Access Management` in the left sidebar.

4. Click the `Create role` button.

5. Select `AWS Service` under `Trusted entity type`. Then select `EC2` under `common use cases`.

6. Search for `AWSElasticBeanstalk` and select the `AWSElasticBeanstalkWebTier`, `AWSElasticBeanstalkWorkerTier` and `AWSElasticBeanstalkMulticontainerDocker` policies. Click the `Next` button.

7. Give the role the name of `aws-elasticbeanstalk-ec2-role`

8. Click the `Create role` button.

### Create Elastic Beanstalk Environment

1. Go to AWS Management Console

2. Search for `Elastic Beanstalk` and click the Elastic Beanstalk service.

3. If you've never used Elastic Beanstalk before you will see a splash page. Click the `Create Application` button. If you have created Elastic Beanstalk environments and applications before, you will be taken directly to the Elastic Beanstalk dashboard. In this case, click the `Create environment` button. There is now a flow of 6 steps that you will be taken through.

4. You will need to provide an Application name, which will auto-populate an Environment Name.

5. Scroll down to find the Platform section. You will need to select the Platform of `Docker`. This will auto-select several default options. Change the Platform branch to `Docker running on 64bit Amazon Linux 2`. The new 2023 branch currently has issues with single-container deployments.

6. Scroll down to the Presets section and make sure that `free tier eligible` has been selected:

7. Click the `Next` button to move to Step #2.

8. You will be taken to a Service Access configuration form.

Select `Create and use new service role` and name it `aws-elasticbeanstalk-service-role`. You will then need to set the `EC2 instance profile` to the `aws-elasticbeanstalk-ec2-role` created earlier (this will likely be auto-populated for you).

10. Click the `Skip to Review` button as Steps 3-6 are not applicable.

11. Click the `Submit` button and wait for your new Elastic Beanstalk application and environment to be created and launch.

12. Click the link below the checkmark under Domain. This should open the application in your browser and display a Congratulations message.

### Update Object Ownership of S3 Bucket

1. Go to AWS Management Console

2. Search for `S3` and click the S3 service.

3. Find and click the elasticbeanstalk bucket that was automatically created with your environment.

4. Click `Permissions` menu tab

5. Find `Object Ownership` and click `Edit`

6. Change from `ACLs disabled` to `ACLs enabled`. Change `Bucket owner Preferred` to `Object Writer`. Check the box acknowledging the warning.

7. Click `Save changes`.

### Add AWS configuration details to .travis.yml file's deploy script

1. Set the region. The region code can be found by clicking the region in the toolbar next to your username.

eg: 'us-east-1'

2. app should be set to the Application Name (Step #4 in the Initial Setup above)

eg: 'docker'

3. env should be set to the lower case of your Beanstalk Environment name with a leading capital letter.

eg: 'Docker-env'

4. Set the bucket_name. This can be found by searching for the S3 Storage service. Click the link for the elasticbeanstalk bucket that matches your region code and copy the name.

eg: 'elasticbeanstalk-us-east-1-923445599289'

5. Set the bucket_path to 'docker'

6. Set access_key_id to `$AWS_ACCESS_KEY`

7. Set secret_access_key to `"$AWS_SECRET_KEY"`. A double-quote is needed here.

8. example travisCI deploy script:

```yml
deploy:
  provider: elasticbeanstalk
  region: "us-east-1"
  app: "docker"
  env: "Docker-env"
  bucket_name: "elasticbeanstalk-us-east-1-923445599289"
  bucket_path: "docker"
  on:
    branch: master
  access_key_id: $AWS_ACCESS_KEY
  secret_access_key: "$AWS_SECRET_KEY"
```

### Create an IAM User

1. Search for the `IAM Security, Identity & Compliance Service`

2. Click `Create Individual IAM Users` and click `Manage Users`

3. Click `Add User`

4. Enter any name you’d like in the `User Name` field.

eg: docker-react-travis-ci

5. Click `Next`

6. Click `Attach Policies Directly`

7. Search for `beanstalk`

8. Tick the box next to `AdministratorAccess-AWSElasticBeanstalk`

9. Click `Next`

10. Click `Create user`

11. Select the IAM user that was just created from the list of users

12. Click `Security Credentials`

13. Scroll down to find `Access Keys`

14. Click `Create access key`

15. Select `Command Line Interface (CLI)`

16. Scroll down and tick the `I understand...` check box and click `Next`

Copy and/or download the Access Key ID and Secret Access Key to use in the Travis Variable Setup.

### Travis Variable Setup

1. Go to your Travis Dashboard and find the project repository for the application we are working on.

2. On the repository page, click `More Options` and then `Settings`

3. Create a variable named `AWS_ACCESS_KEY` and paste your IAM access key from step #13 above.

4. Create a variable named `AWS_SECRET_KEY` and paste your IAM secret key from step #13 above.

### Deploying App

1. Make a small change to your src/App.js file in the greeting text.

2. In the project root, in your terminal run:

```bash
git add.
git commit -m “testing deployment"
git push origin main
```

3. Go to your Travis Dashboard and check the status of your build.

4. The status should eventually return with a green checkmark and show `build passing`

5. Go to your AWS Elastic Beanstalk application

6. It should say `Elastic Beanstalk is updating your environment`

7. It should eventually show a green checkmark under `Health`. You will now be able to access your application at the external URL provided under the environment name.
