## Understanding Architecture

1. When we click on "Pay now", the request goes to the HDFC bank server.

2. The bank server responds back with a token to the backend.

3. The Backend provides a url for the frontend to redirect to the bank's payment page.

4. The bank's payment page is shown to the user.

5. After user pays the amount, the bank redirects the user back to the frontend with a response.

---

## Understanding Authentication

1. Go to the `apps/user-app/api/auth/[...nextauth]/route.ts` file.

2. Any request to the `/api/auth` endpoint is handled by the `next-auth` package.

3. The `next-auth` package is used to authenticate the user. Let's go deep inside the `authOptions` being imported inside this file.

4. The `authOptions` is our main configuration file for authenticating the user. It is being exported from `app/lib/auth.ts`

   - `providers: [..]` : This is where we define the providers for authentication. We are using phone no. and password for authentication.

   - `name: 'Credentials` : This is the name of the provider. Will be shown in the login page button.

   - `credentials: {..}` : This is where we define the fields required for authentication.

   - `authorize: async (credentials) => {..}` : This is where we define the main logic for authenticating the user.

     - `return {...}` : We need to return an object with the user data if the user is authenticated else return null.

   - `callbacks: {..}` : These are the callbacks that are called after the user is authenticated.
     - `async session(session, user) {..}` : After the user is authenticated, We can add the user data to the session here.
     - `session.user.id = token.sub` : This is how we add the user id to the session.
     - `return session`: We need to return the session object. It will be available in the `req` object in the API routes.

---

## Understanding Password Hashing

1. When a user signs up, we can see the password being hashed in the database.

2. This is done using a package called [`bcrypt`](https://www.npmjs.com/package/bcrypt).

3. If we continue looking at the code inside `async authorize(credentials)` , we can see the password being hashed using the `bcrypt` package.

---

## Understanding the Next Steps

1. We have created the basic user and merchant ui with authentications.

2. Today's goal is to create

   - a dummy bank server.
   - a pure backend server which will work as a webhook handler that will handle the bank's response and payment requests.

3. Go to `bank_webhook` folder to see next steps.

---

## What is Continous Integration?

Does the following work on each commit:

- Building the project
- Running tests
- Checking for code quality

## What is Continous Deployment?

Continous Deployment is the practice of automatically deploying new code to production.
This is done by having a pipeline that automatically deploys the code to production after it has been built, tested and checked for code quality.

### Flow of CD

1. **Source Control**: The source code is stored in a source control system like Git.
2. **Build**: The code is built using a build tool like Maven or Gradle.
3. **Test**: The code is tested using a testing framework like JUnit.
4. **Quality Check**: The code is checked for quality using a code quality tool like SonarQube.
5. **Deploy**: The code is deployed to production using a deployment tool like Jenkins.
6. **Monitor**: The code is monitored using a monitoring tool like Nagios.
7. **Feedback**: The results of the build, test, quality check and deployment are sent back to the developer.

---

## Setting up CI Pipeline in Github

1. **Create a new repository**: Create a new repository in Github or fork the monorepo [paytm-end-to-end](https://github.com/its-id/paytm-end-to-end.git).
<br/>

2. Create a .yaml file in the root of the repository (`.github/workflows/build.yml`). This file will contain the configuration for the CI pipeline (basically, the steps that need to be executed when a commit is made to the repository).
   <br/>

3. Add the following code to the `build.yml` file:

   ```yaml
   name: Build on PR

   on:
     pull_request:
       branches:
         - master

   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         - name: Use Node.js
           uses: actions/setup-node@v3
           with:
             node-version: '20'

         - name: Install Dependencies
           run: npm install

         - name: Run Build
           run: npm run build
   ```

   <details>
   <summary>Explaining above file</summary>

   - `name`: The name of the workflow.
   - `on`: The event that triggers the workflow. In this case, the workflow is triggered when a pull request is made to the master branch.
   - `jobs`: The jobs that need to be executed when the workflow is triggered.
   - `build`: The name of the job.
   - `runs-on`: The operating system on which the job needs to be executed.
   - `steps`: The steps that need to be executed in the job.
   - `uses`: The action that needs to be used in the step.
   - `actions/checkout@v3`: The action that checks out the code from the repository. (open-source)
   - `actions/setup-node@v3`: The action that sets up Node.js. (open-source)
   - `node-version`: The version of Node.js that needs to be set up.
   - `run`: The command that needs to be executed in the step.
   </details>

4. Push the changes to the repository and see the CI pipeline in action in every commit.

---

## Dockerizing the application

Dockerize the application by creating a `Dockerfile` for each service inside `docker` folder in root of the repository. This file will contain the configuration for the Docker image that needs to be created.

1. **Creating Dockerfile for `user-app`**:

   ```Dockerfile
   FROM node:20.12.0-alpine3.19

   WORKDIR /usr/src/app

   COPY package.json package-lock.json turbo.json tsconfig.json ./

   COPY apps ./apps
   COPY packages ./packages

   # Install dependencies
   RUN npm install

   # Global package.json script to generate prisma client
   RUN npm run db:generate

   # Can you filter the build down to just one app?
   RUN npm run build

   CMD ["npm", "run", "start-user-app"]
   ```

2. Build the Docker image by running the following command:

   ```sh
   docker build -t user-app -f docker/Dockerfile.user .
   ```

3. Run the Docker image by running the following command:
   ```sh
   docker run -p 3001:3001 user-app
   ```

Similarly, create Dockerfile for `merchant-app` and `bank-webhook` services.

---

## Setting up CD Pipeline in Github

1. Create another .yaml file in the root of the repository (`.github/workflows/deploy.yml`).

   <br>

   > This file will contain the configuration for CD Pipeline of pushing the image to `dockerhub` (basically, the steps that need to be executed when a commit is made to the repository).

2. Create a new docker repository in [dockerhub](https://hub.docker.com/repository).

3. Go to Github repository settings and add the following secrets:

   - `DOCKER_USERNAME`: Your dockerhub username.
   - `DOCKER_PASSWORD`: Your dockerhub password.
    <br>

   ![alt text](/assets/github-secrets-page.png)

---

## Deploying to EC2 machine

1. Create an EC2 instance in AWS.
   <br/>

2. SSH into the EC2 instance.
   <br/>

3. Install Docker on the EC2 instance.
   <br/>

4. Add the `SSH_HOST`, `SSH_KEY` and `SSH_KEY` to the Github repository secrets
   <br/>

5. Create another .yaml file in the root of the repository (`.github/workflows/deploy-ec2.yml`) with following content:

   ```yaml
   name: Build and Deploy to Docker Hub

   on:
     push:
       branches:
         - master # Adjusted to trigger on pushes to master

   jobs:
     build-and-push:
       runs-on: ubuntu-latest
       steps:
         - name: Check Out Repo
           uses: actions/checkout@v2

         - name: Prepare Dockerfile
           run: cp ./docker/Dockerfile.user ./Dockerfile

         - name: Log in to Docker Hub
           uses: docker/login-action@v1
           with:
             username: ${{ secrets.DOCKER_USERNAME }}
             password: ${{ secrets.DOCKER_SECRET }}

         - name: Build and Push Docker image
           uses: docker/build-push-action@v2
           with:
             context: .
             file: ./docker/Dockerfile.user
             push: true
             tags: ikdan/paytm-end-to-end:latest

         - name: Verify Pushed Image
           run: docker pull ikdan/paytm-end-to-end:latest

         - name: Deploy to EC2
           uses: appleboy/ssh-action@master
           with:
             host: ${{ secrets.SSH_HOST }}
             username: ${{ secrets.SSH_USERNAME }}
             key: ${{ secrets.SSH_KEY }}
             script: |
               sudo docker pull ikdan/paytm-end-to-end:latest
               sudo docker stop web-app || true
               sudo docker rm web-app || true
               sudo docker run -d --name web-app -p 3005:3000 ikdan/paytm-end-to-end:latest
   ```

6. Push the changes to the repository and see the CD pipeline in action.

### Deploying with Auto Scaling

**AWS Elastic Beanstalk**: AWS Elastic Beanstalk is a service for deploying and scaling web applications and services.

To deploy the application to Elastic Beanstalk, create an Elastic Beanstalk environment and add the following workflow action to the `deploy-ec2.yml` file:

```yaml
- name: Deploy to Elastic beanstalk
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets. SSH_HOST }}
    username: ${{ secrets. SSH_USERNAME }}
    key: ${{ secrets. SSH_KEY }}
    script: sudo docker pull 100devs/week-18-class:latest sudo docker stop web-app | | true
    sudo docker rm web-app || true
  sudo docker run -e DATABASE_URL ${{ SECRETS.DB_URL }} --restart-always -d --name web-app -p 3005:3000 100devs/ week-18-class:latest
```

- Replace the `SECRETS.DB_URL` with the database URL.

## Reverse Proxying with Nginx

1. Install Nginx on the EC2 instance.
2. Create a new file in `/etc/nginx/nginx.conf` with the following content:

   ```nginx
   server {
       listen 80;
       server_name localhost;

       location / {
           proxy_pass http://localhost:3005;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

3. Restart Nginx.
