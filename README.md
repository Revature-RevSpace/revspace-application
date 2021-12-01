# RevSpace 
## Project Description
RevSpace is a full-stack web application that allows users to register for an Account to make posts on the Social Media Application. Posts can be liked and commented on by other various users. Users are also able to modify their own user profiles.

### Environment / Technologies
Spring MVC, Git, Java, Angular 4, AWS S3, Spring Test, JUnit4, Selenium, Spring Boot, Spring Data, Jenkins, Cucumber, Postman, Spring Test, Spring AOC, Log4J, Maven

### Contributions By:
Michael Than, Jaskaran Singh, Parita Shah, Theodore Fanflor, Ahmed Al-sahli, Dylan Rigney, Devin Louis Reichle, Gabriel Gastelum-Valenzuela, Keven Mitchell, Huangyingrui Wang, Thomas Latham, Joseph Bettendorff, Victor Gee, Shamim Rahman, Ryan Whitehill, Tutan Negash Abegaz, Bannon Smith, Jada Forde, Matthew Ramirez, Melissa Martinez

## Getting Started

### Backend Server

The following prerequisites are required to run or develop the backend server:

* Git
* Java 8+
* Maven
* A PostgreSQL relational database
* (for development) A Java IDE with support for maven projects (Eclipse or Intellij IDEA recommended)

To run the backend server:
1. Clone the backend repository
  * `git clone https://github.com/Revature-RevSpace/revspace-backend.git`
  * This will create a revspace-backend git directory; keep in mind that the maven project itself is a subdirectory at `/.revspace-backend/RevSpaceWebService`
2. To deploy the backend server on an EC2 instance, these shell commands can be run:
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

The backend repository also includes a SQL script to initialize tables in your database and populate them with sample data.

### Frontend Web App

The following software is required to run or develop the frontend web app:
* Git
* npm
* (optional, for development) An IDE that supports TypeScript (such as VS Code)

To run or build the web app:

1. Clone the frontend git repository
  * `git clone https://github.com/Revature-RevSpace/revspace-frontend.git`
  * The newly cloned directory has two subdirectories in it, `./revspace-frontend/RevSpaceWebApp` is the Angular project for the frontend web app
2. From the `./revspace-frontend/RevSpaceWebApp` directory, run `npm install` to download dependencies
3. Run `ng serve` to run the web app in development mode, or run `ng build` to build static web files.
  * Either of these commands can be run with `--configuration=development` or `configuration=production`
  * `ng serve` defaults to development mode, `ng build` defaults to production mode
  * In development mode, the web app will look for the backend server at `http://localhost:8080`
  * In production mode, the web app will look for the backend server at the url specified in the `backendURL` field of `src/environments/environment.prod.ts` (this is currently hardcoded and will need to be changed or be refactored to make more configurable if you are deploying your own backend server)

### Frontend Testing

The following software is required to run or develop the frontend web app testing app:
* Git
* Java 8+
* Maven
* (for development) A Java IDE with support for maven projects (Eclipse or Intellij IDEA recommended)
* Chrome OR Firefox web browser
* A selenium webdriver for chrome or firefox

The testing app is part of the same subproject as the frontend web app. After cloning the frontend web app as described above, the unit tests are found in a maven project at `./revspace-frontend/frontend-tests`.

The unit testing maven project for the frontend web app use these environment variables:

* `WEB_DRIVER_PATH`
  * The path to the selenium web driver executable, e.g. `C:/Program Files/Selenium/geckodriver.exe`
* `WEB_DRIVER_TYPE`
  * Which browser the webdriver will run. Currently allowed values are `FIREFOX` and `CHROME`.
* `WEB_APP_URL`
  * The URL to have the web browser being run by Selenium look for the web app at, e.g. `http://localhost:4200/`
