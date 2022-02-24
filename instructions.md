# Jenkins/Spring Boot Tutorial

This tutorial steps you through the creation of a Jenkins pipeline for an existing Spring Boot web service project.

You will need:

* Internet connection
* Browser
* Terminal/command prompt
* Heroku account
* Locally installed Heroku CLI tools
* ClearDB MySQL Database add-on installed in Heroku account
* MySQL Server
* MySql Workbench
* Java 8 (or greater)
* IntelliJ IDEA Community Edition
* A private GitHub repository for this project
* Jenkins Server (set up as part of this tutorial)

## Background

Jenkins is an open source server that is used to automate continuous integration and continuous delivery tasks in the software development process.

The Pipeline (with a captital P) is the abstraction that Jenkins uses to define and automate continuous integration and continuous delivery processes. A Pipeline consists of various software plugins and a **Jenkinsfile** that defines the Pipeline and specifies its behavior.

### Pipeline Design

We will create and configure a Pipeline for an existing RSVP web service consisting of the following stages:

1. Build - compiles the project
2. Test - runs the unit and integration tests
3. Deliver - packages the project

## Creating the Pipeline

We will exercise the Jenkins pipeline functionality with the following steps:

1. Jenkins installation
2. Set up an application on Heroku
3. Pipeline creation
4. Pipeline execution

---

## Part 1: Jenkins Installation

Our first step is to download, install, and configure Jenkins.

### 1.1: Download Jenkins

Go to https://jenkins.io/download/ and select Generic Java package (.war) under the Long-term Support (LTS) column as shown here. *Do not attempt to install a different installation*:

![](images/jenkins-100-lts-war-2-303-1.png)


Download this file to your computer.

Create a folder on your computer named `jenkins` to serve as your Jenkins root folder.

Move the `jenkins.war` file from your Downloads folder into the `jenkins` root folder.

### 1.2: Install Jenkins

Now open a terminal or command prompt and navigate to your new `jenkins` directory and enter the following command:

```bash
java -jar jenkins.war --httpPort=8085
```

This will start Jenkins on port 8085 and kick off the installation process (any available port can be used, it does not have to be port 8085). Leave this command window open for now. It needs to be open in order for jenkins to continue running. You will need the generated password for the Admin user to proceed. Scan your terminal/command prompt output for the following message and copy the generated password before navigating to http://localhost:8085 (note that your password will be different than the password in the image below):

![](images/jenkins-110-generated-password.png)

Type this password in as the Administrator password when prompted on the Unlock Jenkins screen and click Continue:

![](images/jenkins-120-unlock-jenkins.png)

Now click on Install suggested plugins:

![](images/jenkins-130-install-suggested-plugins.png)

Select Retry for any plugins that fail to install initially. 

The installer will now prompt you to create an Admin User. Supply the following values and click Save and Continue:

Username: root

Password: root

Full Name: root

E-mail Address: root@root.root

![](images/jenkins-140-admin-user-creation.png)

Leave the default value for Jenkins URL on the next screen and click Save and Finish:

![](images/jenkins-150-instance-configuration.png)

Your installation is complete! Click on the Start using Jenkins button:

![](images/jenkins-160-installation-complete.png)

Go back to your terminal or command prompt where you started jenkins, and shut down Jenkins by entering Ctrl-C. You may need to enter Ctrl-C twice. Restart Jenkins by issuing the following command:

```bash
java -jar jenkins.war --httpPort=8085
```

Now visit http://localhost:8085. Login using the admin user credentials (username: root, password:root) that you setup earlier.

---

## Part 2: The application: RSVP Service

### 2.1 Run the application locally

#### 2.1.1 Database setup

This application uses a MySQL database. We need to create the schema and table for the application. We will also need to create a test version of the schema and table.

1. Open MySQL Workbench and connect to localhost.

    ![](images/mysql-10-localhost.png)

2. In the query pane, paste the following code to create the main schema and table, and the test schema and table.

    ```
        create schema if not exists rsvp;
        use rsvp;
                
        create table if not exists rsvp (
        rsvp_id int not null auto_increment primary key,
        guest_name varchar(50) not null,
        total_attending int not null
        );

        create schema if not exists rsvp_test;
        use rsvp_test;
                
        create table if not exists rsvp (
        rsvp_id int not null auto_increment primary key,
        guest_name varchar(50) not null,
        total_attending int not null
        );
    ```

3. Run the database code by clicking the lightning bolt.

    ![](images/mysql-20-run-code.png)

4. Confirm the schema was created.
    
    A. If the Schemas view is not displayed, click the Schemas tab to display it.

    ![](images/mysql-30-schemas-tab.png)

    B. Click the refresh button on the Schemas pane.

    ![](images/mysql-40-refresh-schemas.png)

    C. Observe that the rsvp and rsvp_test schemas are present.

    ![](images/mysql-50-schemas-present.png)

#### 2.1.2 Java code setup

1. Create a new directory *that is outside of all git repositories* called `rsvp-service` Copy the contents of the rsvp-service directory that is a part of this activity into the new directory. It is important that this directory is not within another repository because we will later create a *new repository* for it that jenkins will monitor.

2. Open the project in IntelliJ

3. In Intellij, open the file `rsvp-service/src/main/resources/application.properties`.

    ![](images/intellij-10-application-properties-configuration.png)

4. Confirm that the password for the root user is correct. The password is the value associated with the property `spring.datasource.password`. If your root user's database password is not `password`, change the value in this file to match your root user's database password.

5. Open the file `rsvp-service/src/test/resources/application.properties`. 

6. Again, confirm that the value associated with `spring.datasource.password` is correct.

    **Both files must be correct before proceeding.**

7. Open the main application class and run the project.

8. Use Postman to confirm that the RSVP API is running correctly as follows:

    * Send a POST request to `http://localhost:8080/rsvps` with the JSON body

        ```javascript
        {
            "guestName":"Snow White",
            "totalAttending":8
        }
        ```

    * Send a GET request to `http://localhost:8080/rsvps` to observe that the Snow White RSVP is present. The result should be

        ```javascript
        [
            {
                "rsvpId": 1,
                "guestName": "Snow White",
                "totalAttending": 8
            }
        ]
        
        ```
### 2.2 Create New GitHub Repo

1. On the GitHub UI create a repo named `rsvp-service`.
    
    * Leave default values for everything: Repo should be Public, and should not have a README or a .gitignore or a license. 
    
2. From terminal navigate to the project folder in your workspace.

3. To initialize the folder as a git repository on your local computer, run the command

    ```git init .``` 
    
    Note: *Make sure to include the (.) at the end of this command.*
4. Run `git status` to show the uncommitted files. Check the first line of the output to confirm the name of your current branch.

    ![](images/terminal-10-branch.png)

    If your output says "On branch main", proceed to the next step.

    If your output says "On branch master", run `git branch -M main` to change the branch from master to main. Main has replaced master as the default branch in GitHub.
5. Run `git add .` to stage the files.
6. Run `git commit -m "First Commit" ` to commit the files.
7. Run `git remote add origin [gitHubRepositoryUrl]` to establish the connection to the repository
    * Where `[gitHubRepositoryUrl]` is the URL to your repository. For example: `https://github.com/rwidtmann/rsvp-service.git`
8. Run `git push -u origin main` to push the code and setup tracking on the main branch.
9. Run `git status` to confirm that the folder is up to date and in sync with the main branch.
 
### 2.3 Create the project on the Heroku Cloud Platform

1. In a browser, navigate to the Heroku Dashboard page at: https://dashboard.heroku.com/

2. In the upper right corner, click the `New` button and select `Create New App`.

    * Name the app `rsvp-service-firstname-lastname` and click the `Create app` button. firstname-lastname is your name. For example, Olivia Rodrigo's app would be called `rsvp-service-olivia-rodrigo`. App names are required to by unique on heroku.

#### 2.3.1: Add a free MySql database instance to the project on Heroku

1. From the Heroku Dashboard, click the Resources menu link.

   ![image](images/heroku-10-resources-menu.png)

2. Under Add-ons type `clearDb` in the text field and you will see an option to select the free version of ClearDB MySql.

    * Click that option and follow the prompts to add that resource to your rsvp-service-first-last instance on Heroku.
    
    > Note: Before you can add resources you will be required to provide a credit card number. As long as you only utilize free add-on resources and do not use more than 999 hours of service processing you will not be charged. We will not ask you to use any paid resources for any of our lesson activities.
    
     ![image](images/heroku-20-cleardb-resource.png)


#### 2.3.2: Configure MySql on Heroku 

1. On the Heroku Dashboard, click the Settings menu at the top of the page.

2. In the middle of the page find the Config Vars section and click the `Reveal Config Vars button`.

   ![image](images/heroku-30-reveal-vars-button.png)

3. After clicking the button, the Configuration Variables are displayed. Take note that there is one Config Var named `CLEARDB_DATABASE_URL`.  

    * Add a new Config Var named `DB_URL` by typing that name in the `Key` text field. 

    * Copy the value in the `CLEARDB_DATABASE_URL` variable and paste that into the value field for the new `DB_URL` that you are creating.

    * Be sure to click the `Add` button to complete and save the new Config Var.
    
     ![image](images/heroku-40-config-var-creation.png)


#### 2.3.3: Connect Heroku to the GitHub Repository

Later, we will be connecting this `rsvp-service` to a Jenkins CI/CD pipeline. We will need this `rsvp-service-first-last` Heroku service to be connected to the GitHub repository. This will create a "web hook" on the GitHub repository that will serve as the connection mechanism between the Heroku service and the GitHub repository. Follow the steps below to connect this Heroku service to GitHub:

1. Click the Deploy tab in Heroku

   ![image](images/heroku-50-deploy-tab.png)

2. In the `Deployment method` section of the page click the `GitHub Connect to GitHub` option.

   ![image](images/heroku-60-deployment-method.png)
      
3. In the `Connect to GitHub` section select your repository from the `Search for a repository to connect to` drop-down option list and enter the specific repository to connect to (in this example that would be the `rsvp-service` repository).
    
    * If the repository is successfully located a `Connect` button will appear. Be sure to click that `Connect` button to
    save the connection.

    ![image](images/heroku-70-connect-to-github.png)

4. After clicking the `Connect` button you should see confirmation that the Heroku service is successfully connected to the GitHub repository. You will see a message that begins "Connected to" as in the image below.

   ![image](images/heroku-80-connect-to-github-success.png)


#### 2.3.4: Connect MySql Workbench to the Heroku MySql database

Just as in our local environment with our local instance of MySql, we need to create the database schema tables that will be utilized by our Java API in the MySql database on Heroku. To do so we will connect MySql Workbench to our MySql Heroku database in the following steps.

1. Open MySql Workbench

    * On the MySql Workbench Dashboard page locate the `MySql Connections` and click the '+' (plus) symbol to add a new connection.
    
    ![image](images/mysql-60-workbench-dashboard.png) 

    * The Setup New Connection dialog box will open
    
    ![image](images/mysql-70-setup-new-connection.png)

2. We will need to provide the required values in the Setup New Connection dialog box. We will obtain those values from the `DB_URL` Config Var that we created in the previous section. Follow these steps:

    * Return to the Heroku Dashboard and locate the Config Vars by following the menu path. In Settings, click the `Reveal Config Vars` button to display the DB_URL variable.

    * Copy the value of the `DB_URL` variable. That value should look similar to this:
    `mysql://b9bece03b87326:007ff0f7@us-cdbr-east-02.cleardb.com/heroku_e5ea0661158612d?reconnect=true`

    * You will need to dissect this environment variable to find the values needed to populate the fields in the Setup New Connection dialog box. Follow this breakdown to obtain the values needed for `Hostname`, `Username`, `Password` and `default schema`. 
    
     ![image](images/mysql-80-workbench-config-breakdown.png)

    * Obtain the values and populate the Setup New Connection dialog box with the `Hostname`, `Username`, `Password` and `default schema` values.
    
    * Name the connection by adding a name in the `Connection Name` text field. 
    
     ![image](images/mysql-90-new-connection-filled.png)
    
    * Click the `Test Connection` button to verify the connection
        
       * You may receive a warning about features on MySql Workbench that might be unavailable. Click `Continue` to move forward with the connection test. 
    
    * Once the values are populated and the connection if verified click the `OK` button to create the connection.
    
    * After the connection is created you will see that new connection as an option on the MySql Workbench Dashboard.
    
    ![image](images/mysql-95-workbench-connection-created.png)



#### 2.3.5: Create the Schema and table on Heroku

1. Now that you have MySql Workbench connected to the Heroku MySql database, click that connection on the MySql Workbench Dashboard to open the connection.
    
    * On the Schemas menu on the left side of the MySql Workbench screen notice that the Schema for our new connection is the `default_schema` value that we entered in the New Connection dialog box. This is because unlike our local MySql database where we create multiple schemas in our local database instance, on Heroku we will only have one schema per database instance.
    
     ![image](images/mysql-955-workbench-schema-display.png)

    * Now we will create the rsvp table on our Heroku database.
        
        * The schema is already established in our connection, so we simply need to run the following SQL script to create the table on the Heroku database where `[default_schema_name]` is the default schema obtained from the Heroku database Config Var.
       
          ```
           use [default_schema_name];
                     
           create table if not exists rsvp (
           rsvp_id int not null auto_increment primary key,
           guest_name varchar(50) not null,
           total_attending int not null
           );
          ```

    * After running the SQL script, you should be able to see that the table is created and you are now able to run SQL queries against the database
    
    ![image](images/mysql-960-workbench-after-scripts-run.png)


> Note: You will NOT need to create the rsvp_test instance on the Heroku database instance.

#### 2.3.6: Deploy the rsvp-service to Heroku

Connect Heroku to the Java API GitHub Repository and Deploy the API - from the command line

1. Switch to the terminal and navigate to the `rsvp-service` root folder

2. Run this command: 
    ```
    heroku git:remote -a [HEROKU-appName]
    ```
    * Replace `[HEROKU-appName]` with the name of the app on Heroku (In this example that name should be `rsvp-service-first-last`)
    * You should see a confirmation that git remote heroku is now set
    
    ![image](images/terminal-20-sync-heroku-to-github-confirmation.png)
    
3. Run this command:
    ```
    git push heroku main
    ```
    
    to push the code and the app out to the Heroku Cloud Platform.

    * You should see a long tailing log that represents the deployment to Heroku. With a final deployment confirmation similar to:
       
     ![image](images/terminal-30-heroku-deployment-verification.png)

4. Following deployment return to the Heroku dashboard page at https://dashboard.heroku.com/
    
    * Click on your `rsvp-service-first-last` app
    
    * In the upper right corner click the `more` button and select `View logs`
        * You should see logging that confirms that the app was deployed, initialized and started
    
     ![image](images/heroku-90-deploy-logs.png)

5. Click on the `Open app` button and a browser window will open. This app does not have a front end so the app will not display. And you will likely see a Whitelabel error indicating a 404 Not Found status in the browser. This is expected.
    
    * Copy the URL for the app from the browser window
    
6. Open Postman and create a POST transaction to create a new rsvp.
    
     * Paste the Heroku service URL that you copied in the previous step and add the `/rsvps` URI so that the request will look like:  `https://rsvp-service-first-last.herokuapp.com/rsvps`
    
     * Add a proper body to save an rsvp in Postman.
    
     * Run the POST transaction to create a new rsvp and confirm a successful response
    
     ![image](images/postman-10-rsvp-post.png)
    
7. In Postman, make a GET request to the same path as the POST request. Again, it will be something like `https://rsvp-service-first-last.herokuapp.com/rsvps` The result should include the RSVP you just sent in the POST request.
  
    ![image](images/postman-20-rsvp-get.png)    

   
Congratulations! You have successfully deployed a Java API to the Heroku Cloud Platform with a working MySql database that is connected and running to support the API.


Now we can exercise the RSVP service. The RSVP service has the following API:

```
Create RSVP
===========
URL: /rsvps
HTTP Method: POST
RequestBody: RSVP data
ResponseBody: RSVP data + ID

Get RSVP
========
URL: /rsvps/{id}
HTTP Method: GET
RequestBody: None
ResponseBody: RSVP data

Get All RSVPs
=============
URL: /rsvps
HTTP Method: GET
RequestBody: None
ResponseBody: Array of RSVP data

Update RSVP
===========
URL: /rsvps/{id}
HTTP Method: PUT
RequestBody: RSVP data
ResponseBody: None

Delete RSVP
===========
URL: /rsvps/{id}
HTTP Method: DELETE
RequestBody: None
ResponseBody: None
```

---

## Part 3: Jenkins Pipeline

There are many different options available in Jenkins to customize a pipeline for most any situation. And the options and tools available open up extensive opportunities to be as creative and complex with those pipeline creations as desired. We encourage you to continue to research and explore the many options available with Jenkins. 

To incorporate the CI/CD process for the rsvp-service project, we'll need to ensure the following target goals:

1. Ability to trigger the pipeline process manually.

2. Automatically trigger the pipeline process each time there is a code push to the project GitHub repository.

3. The pipeline process should consist of the following stages.

    * Build/Compile
    * Test
    * Deploy

If any of the three stages fails, then the pipeline status will indicate failure and the rsvp-service will NOT be updated on the Heroku cloud. 
   
### 3.1: Initialize the Pipeline

Access the Jenkins Dashboard page by navigating to: `http://localhost:8085/`. It should look something like this:


 ![](images/jenkins-300-dashboard.png)

   
#### 3.1.1: Create Global Credential in Jenkins for GitHub

Before we continue with the pipeline creation, we will take a moment to store our GitHub credentials in Jenkins so we can use those in the pipeline creation.

1. From the main Jenkins dashboard page select, the `Manage Jenkins` option from the left-side menu. Then choose the `Manage Credentials` tile in the Security section of the main content area

      ![image](images/jenkins-310-manage-credentials-menu.png)  
      
2. From the Credentials page, hover your mouse pointer over the word "(global)" in the **Stores scoped to Jenkins** section, and click the drop-down arrow that appears. Select `Add Credentials`

      ![image](images/jenkins-320-add-credentials-menu.png)
      
3. To setup the Global Credentials entry for GitHub, enter the following options and click OK:

    * Kind: leave the default setting as `Username with password`
    * Scope: leave the default setting as `Global (Jenkins, nodes, items, all child items, etc)`
    * Enter your GitHub username
    * Leave the **Treat username as secret** box unchecked
    * Enter your GitHub password (the one you need to log into the website)
    * ID can be left empty
    * For Description, enter My GitHub Credentials
    * Click the `OK` button to save these settings
    
       ![image](images/jenkins-330-credentials-form-complete.png)

4. After completing the Global Credentials GitHub creation you should now see your credentials ready to be utilized for
this pipeline and any additional pipelines you create in the future

   ![image](images/jenkins-340-credentials-final.png)

---


#### 3.1.2: Configure a new pipeline

1. To create a new pipeline select `New Item` from the left-side menu on the Jenkins dashboard page

   ![](images/jenkins-400-new-item.png)

2. After selecting `New Item`, you should see the pipeline creation page. On this page perform the following:
    * Give the pipeline a name. Here we will enter: `rsvp-jenkins-pipeline`. Or any name you prefer for the pipeline.
    * From the list of pipeline template options choose `Pipeline`.
    * Be sure to click the `OK` button at the bottom of the page to save your details.
    
   ![](images/jenkins-410-create-pipeline-project.png)

3. On the next page perform the following:
    * Enter 'Pipeline for rsvp-service project' in the Description text area.
    * Under the `Build Triggers` section check the `GitHub hook trigger for GITScm polling` checkbox.
        * Checking this option will automatically trigger a pipeline build process every time new code is pushed to the `rsvp-service` GitHub repository.

   ![](images/jenkins-420-desc-build-trigger.png)
   
4. In the pipeline section from the `Definition` drop-down list select `Pipeline script from SCM` (SCM stands for Source Code Management. In this case, that is git.)
    
   ![](images/jenkins-440-pipeline-selections.png)
  
5. You will see a drop-down for `SCM` and that the `Script Path` is set to `Jenkinsfile`.

    > **Important Note:** Be sure to *uncheck* the `Lightweight Checkout` checkbox.

   ![](images/jenkins-450-pipeline-setup.png)

6. Next for `SCM` select `Git` from the drop-down menu and additional selections will appear.

   ![](images/jenkins-460-scm-details.png)
   
7. Enter the GitHub repository URL for the rsvp-service GitHub repository in    the `Repository URL` field.

8. Select `My GitHub Credentials` from the `Credentials` drop-down menu.

9. Change the `Branch Specifier` field from its default value: `*/master` to `*/main`. Recently GitHub has changed the HEAD branch defined for new repositories to `main` instead of `master`.

10. Leave the rest of the fields as is.

11. Click `Apply` then `Save` to store the values
    
   ![](images/jenkins-470-scm-settings.png)

---


#### 3.1.3: Create the Jenkinsfile

The Jenkinsfile defines the behavior of the Pipeline. A Jenkinsfile can be quite complex and can be written in Declarative or Scripted syntax. We will create a fairly simple Declarative Jenkinsfile. Declarative files are straightforward and suffice for most simple projects.

A Declarative Jenkinsfile requires at minimum five directives:

1. `pipeline` - This is the required container directive for the Jenkins file.
2. `agent` - This directive tells Jenkins to create a workspace and allocate an executor to run the Pipeline. Without an agent, the Pipeline cannot run. The agent is also responsible for checking out the source code for the project from the repository. The details of the repository access are configured through the Jenkins UI. We will do that in the next step.
3. `stages` - This is the container directive for all of the stages of the Pipeline.
4. `stage`- This directive holds the definition for a stage of the Pipeline. For example `build` or `test`.
5. `steps` - This directive contains the specific commands to be run for a particular stage. In our example these are all shell (Mac) or batch (Windows) commands.

From IntelliJ create a new file called `Jenkinsfile` (note the capital J) in the root folder of your `rsvp-service` project.

   ![](images/intellij-200-new-Jenkinsfile.png)

Copy the following content (use the appropriate file for your operating system). 

> Note: Do not add an extension suffix to the file. From IntelliJ simply create a new file named `Jenkinsfile`.

##### Mac

```groovy
pipeline {
    agent any

    stages {

        stage('build') {
            steps {
              sh '''
                 ./mvnw -DskipTests clean compile
              '''
            }
        }

        stage('test') {
            steps {
              sh '''
                 ./mvnw test
              '''
            }
        }

        stage('deliver') {
            steps {
              echo 'Deploying...'
              sh '''
                 git push https://git.heroku.com/rsvp-service-1.git HEAD:main -f
              '''
            }
        }

    }
}
```



##### Windows

```groovy
pipeline {
    agent any

    stages {

        stage('build') {
            steps {
              bat '''
                 ./mvnw -DskipTests clean compile
              '''
            }
        }

        stage('test') {
            steps {
              bat '''
                 ./mvnw test
              '''
            }
        }

        stage('deliver') {
            steps {
              echo 'Deploying...'
              bat '''
                 git push https://git.heroku.com/rsvp-service-1.git HEAD:main -f
              '''
            }
        }

    }
}
```

Some items to note about the Jenkinsfile:

* We don't have any special requirements for running this Pipeline so we specify `any` for the agent, which allows Jenkins to use its default agent.
* The file contains three stages: `build`, `test`, and `deliver`.
* Each `stage` has one `steps` directive which consists of shell or batch commands. The use of `sh` (Mac) or `bat` (Windows) allows us to run shell or batch commands. Wrapping  the shell or batch commands in `'''` allows us to run more than one command in sequence. This is crucial in our case because we need to change directories and then execute another command. 
* Each `stage` of our Pipeline consists of running Maven commands using the Maven instance included in our project by the Spring Initializr (mvnw).

* **Important Note:** in the final stage `deliver`, you need to change the content. In the current example, the push command `git push https://git.heroku.com/rsvp-service-1.git HEAD:main -f` has `rsvp-service-1.git` which represents the
name of the service on Heroku. Change it to your heroku app name (for example, `https://git.heroku.com/rsvp-service-olivia-rodrigo.git`).

**Commit your changes and push them to your GitHub repository (add, commit, push to origin main).**

---

### 3.2: Create GitHub webhook and a localhost connection tunnel to expose Jenkins

One of the goals specified in this tutorial is to have an automatic Jenkins build/test/deploy process triggered each time code is pushed to the GitHub project repository. In our Jenkins pipeline configuration under the section `Build Triggers` we selected `GitHub hook trigger for GITScm polling`. We said that this was the setting that would trigger the pipeline process each time code is pushed to the GitHub repository. 

Because the Jenkins server is running locally on your computer, it is not exposed to the external Internet.

In order to configure the connection between the GitHub 
repository (which lives on the internet), and our local instance of Jenkins, we can configure an "HTTP tunnel." An HTTP tunnel connects a publicly accessible IP address and port to an IP address and port that is not publicly accessible, like localhost:8085. Then, a computer on the public internet can connect to the new publicly accessible IP address and port, and that connection will be forwarded to (or *tunneled* to) Jenkins running on your computer. In this way, the GitHub trigger event can be received by your local Jenkins instance.

To set up this tunnel, and expose port 8085 to the Internet, we will utilize ngrok.


#### 3.2.1: ngrok Download and Configuration

1. Navigate to the ngrok website at: https://dashboard.ngrok.com/get-started/setup.
2. Download the ngrok zip file.
3. On your computer create a folder named `ngrok` to serve as a root folder.
4. Copy the ngrok zip file to the ngrok root folder and unzip the file.
5. Open a terminal window and navigate to the ngrok root folder
6. Run the command to create the HTTP tunnel for port 8085: `./ngrok http 8085`.
    * Copy the URL generated by ngrok and save that to your clipboard. We will use that to create the webhook on GitHub.
    For example shown below: `http://2f39e3a09363.ngrok.io`
    
    ![](images/ngrok-100-started-screen.png)
    
*Note that the URL generated by ngrok is mapped to http://localhost:8085*
 
 
#### 3.2.2: Update Repository Configuration on GitHub

1. From a browser navigate to the rsvp-service GitHub repository.
2. Select the `Settings` menu.

   ![](images/github-100-home-screen.png)

3. From the left-side menu select `Webhooks` and then click the `Add webhook` button.

   ![](images/github-110-add-webhook.png)

4. In the `Payload URL` field, paste the ngrok URL that you copied to your clipboard and append the following to the end of that URL `/github-webhook/`. Be sure to add the `/` (forward-slash) at the end of the `/github-webhook/` URI.
   * For example the final webhook URL should look similar to: `http://2f39e3a09363.ngrok.io/github-webhook/`
5. Leave everything else as is, and click the `Add webhook` button to save the new webhook
    
   ![](images/github-120-webhook-config.png)

6. If the webhook was successfully saved and ready for use you will see a green checkmark in front of the webhook. If the newly created Webhook does not show the active check mark, you may need to refresh the GitHub dashboard screen.

   ![](images/github-130-successful-webhook.png)

**Note:** The Webhook created will not be restored after shutting down your computer. Upon restart, you will need to create a new Webhook to connect from the GitHub repository to your computer port where the Jenkins server is running.

---

## Part 4: Run and Test the Pipeline

Congratulations on creating the Jenkins Pipeline. It is now time to trigger our first manual build/test/deploy process
and then test the automatic deployment process.

### 4.1: Manual Pipeline Trigger

* Open the Jenkins Dashboard in a browser window.

* Open the rsvp-service-first-last Heroku dashboard page in another browser window. Open the application log by clicking the `More` button in the upper-right side of the screen and selecting `View logs`.

1. From the Jenkins Dashboard, click on the rsvp-jenkins-pipeline Select `Build Now` from the left-side menu.

   ![](images/jenkins-500-manual-build-now.png)

2. Clicking the `Build Now ` button triggers the pipeline. Notice in the lower-left side of the dashboard screen the 
Build History section. This section lists the past and current builds that have been triggered. If you click the drop-down
arrow associated with the build you can choose the `Console Output` option to see the Jenkins build log file.

   ![](images/jenkins-510-build-and-console-output.png)

3. You can monitor the progress of the 3 build stages that we specified in the Jenkinsfile (build/test/deliver). And if 
the build is successful you will see indication of that similar to here:

   ![](images/jenkins-520-console-output-log.png)
   
4. In addition to the console log the progress of a build can also be seen in the `Stage View` shown in the main section
of the dashboard screen. 
    
   * Notice that the `Stage View` shows four stages of the build. The three stages we defined in the 
Jenkinsfile (build/test/deliver) and an initial first stage which represents the pulling of the latest code from the 
GitHub repository. 

   * Also in this example we can see that both builds #18 and #19 were successful, as indicated by the green shading color of each build step. If there was a failure, the stage that failed would be shown in a pink color.
   
   ![](images/jenkins-530-stage-section.png)
 

### 4.2: Automatic GitHub Push Trigger

As we stated earlier in this tutorial, our other trigger goal is to have a code push to the GitHub repository trigger the Jenkins pipeline. 

* Make a code change to the rsvp-service. You can add a log message, or a comment, or update the functionality. Push that change to the GitHub repository.
    
* As soon as the push is made you should see a pending pipeline build indicated in the `Build History` section of the
    Jenkins dashboard. That build will pend briefly and then turn into a full build. All triggered by the code push to
    the rsvp-service repository.
    
    ![](images/jenkins-530-pending-build.png)
  
  
Congratulations! You have successfully built a functioning Jenkins pipeline that will now allow you to perform Continuous Integration and Continuous Delivery (CI/CD) for your rsvp-service!

In fact, this pipeline Coninuously Deploys to heroku. So, you no longer have to push code changes to heroku. Instead, every time you push to the main branch in GitHub, the change is detected by jenkins, built, tested, and then *deployed to heroku*. And, a benefit of the pipeline is that if the build fails (due to syntax error, or for any reason), or if the tests fail, the pipeline will stop execution, and prevent that code from being automatically pushed to heroku.

--- 

## Part 5: Cleanup

Finally, it is always a good idea to stop any services that you are not using to ensure that no service charges will be incurred on heroku.

When you're satisfied that you've seen everything you want and need to see, you can decommission the ClearDB MySQL database on Heroku by going to the app view for rsvp-service-first-last.

![](images/heroku-95-choose-app.png)

In the Resources tab, click the menu icon in the ClearDB MySQL section. Click Delete Add-on, and confirm that you actually want to delete the database.

![](images/heroku-98-delete-cleardb.png)

If you want to, you can also delete the app from heroku.

---

&copy; 2021 Trilogy Education Services, a 2U Brand