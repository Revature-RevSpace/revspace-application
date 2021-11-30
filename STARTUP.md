# Startup

This document details how to get started on a new iteration of RevSpace development.

## Repository Structure

The main repository is located at `https://github.com/Revature-RevSpace/revspace-application`.

This has two sub-repositories, which can be cloned individually, or cloned together with `git clone --recurse-submodules https://github.com/Revature-RevSpace/revspace-application.git`.

* The backend repository can be cloned individually with `https://github.com/Revature-RevSpace/revspace-backend.git`. This contains:
  1. A `RevSpaceWebService` directory containing a Maven project for the backend server
  2. A `RevSpaceDB.sql` SQL script, which can be run on a PostgreSQL relational database to create the schema tables and populate them with some sample data

* The frontend repository can be cloned individually with `git clone https://github.com/Revature-RevSpace/revspace-frontend.git`. This contains:
  1. A `RevSpaceWebApp` directory containing an Angular project for the frontend web app
  2. A `frontend-tests` directory containing a Maven project with Selenium tests for the frontend web app

## Using the Frontend Web App

If the previous development iteration's cloud resources are being re-used, there should be a web app deployed on the S3 bucket that can be reached publicly and is able to connect to the backend server. Otherwise, read further for instructions on deploying the backend server and the frontend web app.

## Running the Backend Server (development environment)
You'll need:
* Git
* Java JDK 8+
* A Java IDE with Maven support (e.g. Eclipse or Intellij IDEA)
* A PostgreSQL relational database
* A means of sending queries to that database (DBeaver recommended)

After cloning the backend repository, open `./revspace-backend/RevSpaceDBScript.sql` in DBeaver, connect to your database, create a new schema, and run the script. This will create the necessary tables and populate them with some sample data.

`./revspace-backend/RevSpaceWebService` is a Maven project that can be built with Maven or opened as a Maven project in most Java IDEs. The pom.xml file specifies all third-party dependencies, which Maven will attempt to automatically download when the project is built (if the project was opened with an IDE, it may be necessary to refresh Maven dependencies via your IDE before the project can be run).

`com.revature.revspace.app.RevSpaceWebService#main` will deploy the server locally on port 8080.

The following environment variables are needed:

* DB_URL
  * The URL of the relational database to connect to. Currently only PostgreSQL is supported.
* DB_USERNAME
  * The username for the database
* DB_PASSWORD
  * The password for the database
* SERVER_PORT
  * The port to run the server on. Defaults to 8080 if not specified.
  * Production servers being deployed from Jenkins will generally to use a different port, such as 8081

When running from an IDE, these can be set in your IDE's run configuration.

When running the built project from a CLI, these can be set by using `-D` flags (see the shell script below for examples)

## Deploying the Backend Server (production environment)

You'll need
* An Amazon EC2 instance with
  * Git
  * Jenkins
  * Maven
  * Java 8+
  * A publicly accessible port (the below example uses port 8081)

Clone the project as described above, or set up Jenkins to automatically pull from the backend repository when changes are pushed.

If using the same EC2 instance as the previous development iteration, you should already have a `deploy-backend` Jenkins job linked to the repository in this manner. Shell scripts will be run from inside the cloned repository folder.

If configuring Jenkins on a new EC2 instance:
* Find your Jenkins Payload URL under Configuration -> Configure System -> Jenkins Location
  * It should look similar to `http://ec2-some-ip-address...amazonaws.com:8080/jenkins/`
* Add the Jenkins Payload URL to a new webhook in the `revspace-backend` GitHub repository
  * Settings -> Webhooks -> Add webhook
* Create a new Freestyle Project in Jenkins
* Under this new project's Configuration -> Source Code Managment, configure it to listen to GitHub webhooks
  * Set Source COde Management type to Git
  * Set Repository URL to `https://github.com/Revature-RevSpace/revspace-backend`
  * Set branch specifier to `*/main` to have Jenkins listen for pushes to the main branch
* In the new project's Configuration -> Build Triggers, enable "GitHub hook trigger for GITScm polling"
* In the project's Configuration -> Build, the following shell script will compile, build, and deploy the backend server when pushes to the GitHub repository are detected:
```sh
# Environment Variables:
DB_URL=url/to/your/database
DB_USERNAME=yourDatabaseUsername
DB_PASSWORD=yourDatabasePassword
SERVER_PORT=8081 # The port to run the server on

# Compile the project, run unit tests, build a jar if the tests succeed
mvn -DDB_URL=${DB_URL} -DDB_USERNAME=${DB_USERNAME} -DDB_PASSWORD=${DB_PASSWORD} clean package -f RevSpaceWebService/pom.xml

# Stop the currently-running server if a previously deployed server is currently running
kill `lsof -t -i:8081` -9 >/dev/null 2>&1 || : # if server isn't running, ignore errors and continue

# Run built jar on a separate thread
java -jar -DDB_URL=${DB_URL} -DDB_USERNAME=${DB_USERNAME} -DDB_PASSWORD=${DB_PASSWORD} -DSERVER_PORT=${SERVER_PORT} ./RevSpaceWebService/target/revspace*.war &
```
* Your EC2 instance will need the server's port (8081 in the above example) to be publicly accessible in order for the frontend web app to be able to use its HTTP API.

## Running the Frontend Web App (development environment)

You'll need
* npm
* (optional, for development) Visual Studio Code

After cloning the frontend web app as described earlier,
* Open a terminal in `./revspace-frontend/RevSpaceWebApp`
* Run `npm install` to generate or download dependency modules
* Run `ng serve` to deploy the web app locally in development mode
  * This will make the web app reachable at `http://localhost:4200`
  * The web app will look for the backend API at `http://localhost:8080` while in development mode
  * `ng serve --configuration=production` will look for the backend API at the URL of the previous development iteration's production server (this is currently hardcoded)

## Deploying the Frontend Web App

You'll need
* Jenkins with NodeJS 17+ installed 

To deploy the frontend web app automatically after a build:

* Create a new Freestyle Project in Jenkins
* Configure this Jenkins project to listen to the revspace-frontend repository in the same manner as described above for the backend
* Under the project's Configure -> Build Environment, enable "Provide Node & npm bin/ folder to PATH" and select your NodeJS installation
* Add the following shell script as a build step:
```sh
# navigate to the correct directory 
cd RevSpaceWebApp

#install angular
npm install -g @angular/cli

#install all the modules
npm install

#build the angular project
ng build

#clean the s3 bucket by deleting all the files within
aws s3 rm s3://revspace --recursive
```

* Add a "Publish artifacts to S3 Bucket" Post-build Action
  * Set source to `**RevSpaceWebApp/dist/RevSpaceWebApp/`

## Running the Frontend Web App Tests

You will need:

* Maven
* A Selenium webdriver executable for the Firefox or Chrome web browser

The revspace-frontend repository contains a maven project at `revspace-frontend/frontend-tests` that uses Selenium and Cucumber to test the frontend web app. It uses the following environment variables:

* `WEB_DRIVER_PATH`
  * The path to the selenium web driver executable, e.g. `C:/Program Files/Selenium/geckodriver.exe`
* `WEB_DRIVER_TYPE`
  * Which browser the webdriver will run. Currently allowed values are `FIREFOX` and `CHROME`.
* `WEB_APP_URL`
  * The URL to have the web browser being run by Selenium look for the web app at, e.g. `http://localhost:4200/`
