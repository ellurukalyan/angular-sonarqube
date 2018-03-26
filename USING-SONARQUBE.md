
# SonarQube With Docker For JavaScript or TypeScript Projects
[SonarQube](https://www.sonarqube.org/) always seemed a little daunting to me, and I never really gave that much thought to it. How wrong was I! While [Static Code Analysis](https://www.owasp.org/index.php/Static_Code_Analysis) might not find invalid business logic, it does help find security issues and it can sniff out code smells that your team has grown accustomed to. It might seem hairy to set it up for local developers, but it's actually quite easy with the help of Docker and npm!

Things you will need installed on your computer in order set this up:
- [Docker](https://www.docker.com/)
- [Node.js](https://nodejs.org/en/) and [npm](https://www.npmjs.com/get-npm)
- [Yarn](https://yarnpkg.com/en/) (For the Angular example below)

## Adding to Your Current Project
Adding SonarQube to your current project is a simple and effective way to help setup quality gates around your current, or new JavaScript or TypeScript codebase. The example below will show you what files you can add to the current [Angular GitHub repository](https://github.com/angular/angular) without affecting the existing structure of the project. You can use this boilerplate to start off any project.

### SonarQube Docker Image
The first thing we will need to do is pull down the latest Docker image of [SonarQube](https://www.sonarqube.org/) to run locally. If you currently are using Docker and Docker Compose for local development, you can just add this image to your existing docker-compose.yml file.

1. Pull down the latest SonarQube docker image:\
`docker pull sonarqube`
2. If you want to leverage Docker Compose and make the container setup easy to share, all you need to do is the following: 
    1. Create a docker-compose.yml file in the root of your project:
        ```yml
          version: '3'
          services:
            sonarqube:
              container_name: sonarqube
              image: sonarqube:latest
              ports:
                - "9000:9000"
                - "9092:9092"
        ```
    2. To start the container:\
    `docker-compose up -d`
    3. To stop the container:\
    `docker-compose stop`
    4. To remove the container:\
    `docker-compose down`
3. If you would rather create the new container by hand and not use Docker Compose:
    1. Create the instance of the container:\
      `docker create --name sonarqube -p 9000:9000 -p 9092:9092 sonarqube`
    2. Start the container:\
      `docker start sonarqube`
    3. Stop the container:\
      `docker stop sonarqube`
    4. To completely remove the container from your system:\
      `docker rm sonarqube`

After your container is up and running, within a few moments you should be able to visit [http://localhost:9000/](http://localhost:9000/) in your browser and see that SonarQube is up and running.

### SonarQube Scanner
Now that we have SonarQube setup, it's time to setup the SonarQube Scanner to run against the codebase. The easiest way to install the scanner is by using the npm module [sonarqube-scanner](https://github.com/bellingard/sonar-scanner-npm). Alternatively, you can scan your code without the usage of node or npm, but it would [require setting up SonarQube Scanner by hand](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner). However, I wouldn't recommend it considering how easy it is to setup with npm.

1. Install sonarqube-scanner as a development dependency for your current project:\
    - For npm:\
        `npm install sonarqube-scanner --save-dev`
    - For yarn:\
        `yarn add --dev sonarqube-scanner`
2. Create a sonar-project.js file in the root of your project with the following code:
    ```javascript
        const sonarqubeScanner = require('sonarqube-scanner');
        sonarqubeScanner({
          serverUrl: 'http://localhost:9000',
          options : {
          'sonar.sources': '.',
          'sonar.inclusions' : 'packages/core/src/**' // Entry point of your code
          }
        }, () => {});
    ```
    - For a list of all the different options you can pass into the scanner, visit [SonarQube's documentation](https://docs.sonarqube.org/display/SONAR/Analysis+Parameters)
    - By default, when scanning a project that has a npm package.json file, the reporting tool will use the package name and version that it finds in the JSON file.
    - You can also pass along your test coverage results into the scanner for SonarQube to parse into it's results.  More on that in a future post.
3. In your package.json file, you can update the script section to add the command to execute:
    ```javascript
      "scripts": {
        ...
        "sonar": "node sonar-project.js"
      },
    ```
4. You can now run the scanner:\
    `npm run sonar`
    - The scan will take a few minutes the first time to complete, but it will be faster for each subsequent run
    - At the end of the run, you will be prompted with the URL to the project's results
5. Be sure to add the `.scannerwork` directory to your `.gitignore` file so you don't accidentally commit it
6. Start to reap the rewards of static code analysis for your project!

### What does it tell you?
If you are not familiar with SonarQube's analysis, let's pull down a well-known repo and see what it finds. I have a fork of Angular's source on GitHub that you can use for testing:\
[https://github.com/ryandoll/angular-sonarqube](https://github.com/ryandoll/angular-sonarqube)

You can see for yourself how easy it is to get started:

1. `git clone git@github.com:ryandoll/angular-sonarqube.git`
2. `cd angular-sonarqube`
3. `docker-compose up -d`
4. `yarn install`
5. `npm run sonar`
6. Visit [http://localhost:9000/dashboard/index/angular-srcs](http://localhost:9000/dashboard/index/angular-srcs) in your browser to see the results

The SonarQube results are very interesting for the Angular project. I love the fact that they only have one hour of debt for 12K lines of code! Play around with the results and get familiar with what SonarQube finds.  I will start working on a future post about common configurations for SonarQube.  For now, the basic setup is a good litmus test for any team to start implementing right away. 