---
title: "Accelerating Your Application Deployment Process with Jenkins, GitHub Webhook, and Maven for Smooth Software Delivery"
seoTitle: "Jenkins, Github-webhook, and Maven"
datePublished: Fri May 05 2023 08:31:57 GMT+0000 (Coordinated Universal Time)
cuid: clhaas5uq000509jo2sroa1e9
slug: accelerating-your-application-deployment-process-with-jenkins-github-webhook-and-maven-for-smooth-software-delivery
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1683275012546/37c6b364-7c31-49a6-b3c8-645117474cdc.jpeg
tags: cloud, automation, jenkins, ci-cd

---

Jenkins is a popular open-source automation tool that can be used to automate the entire software delivery process. In this documentation, we will be discussing how to set up a Jenkins server and an application server running a Maven application using Jenkinsfiles. Additionally, we will cover how to create a GitHub webhook to trigger automatic builds.

Prerequisites:

* A server to install Jenkins and the application server.
    
* Java JDK is installed on the server.
    
* Maven is installed on the server.
    
* A GitHub account with a repository to build.
    

**Step 1: Install Jenkins on the Server**

* Ubuntu:
    
    ```plaintext
    sudo apt update
    sudo apt install openjdk-8-jdk -y
    wget -q -O - <https://pkg.jenkins.io/debian-stable/jenkins.io.key> | sudo apt-key add -
    sudo sh -c 'echo deb <https://pkg.jenkins.io/debian-stable> binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt update
    sudo apt install jenkins -y
    
    ```
    
* Start Jenkins service: `sudo systemctl start jenkins`
    

**Step 2: Install Required Plugins**

Plugins are software components that provide additional functionality to the core system. Jenkins plugins can be installed and configured through the Jenkins web interface, and they can add new features or modify existing ones, such as build triggers, source code management, reporting, and notifications. Plugins in Jenkins are essentially Java Archive (JAR) files that contain code that extends Jenkins. They are designed to be modular, so you can choose which plugins to install based on your specific needs.

Go to Jenkins -&gt; Manage Jenkins -&gt; Manage Plugins -&gt; Available.

* Maven Integration plugin:
    
* Git plugin
    
* Pipeline plugin
    
* GitHub plugin
    

**Step 3: Set up the Application Server**

Tomcat is like a chef who specializes in serving up Java web applications to hungry internet users. It's a powerful and flexible tool that can take the raw ingredients of your Java application and turn them into a delicious, fully-cooked website.

Just like a chef needs a kitchen to work in, Tomcat needs a server to run on. Once it's fired up, it can take in your Java web application and start serving it up to users who request it. It's a bit like a waiter, who takes orders from customers and brings them their food - except in this case, the customers are internet browsers, and the food is your website.

But Tomcat isn't just a simple waiter. It's more like a gourmet chef who can customize each order to suit the customer's tastes. It can handle all sorts of requests, from basic HTML pages to complex Java applications, and it can even handle multiple requests at once, just like a busy restaurant.

So, if you want to serve up a delicious Java web application to the world, Tomcat is the chef you need in your kitchen. It's fast, flexible, and always ready to cook up something amazing.

* Install the application server, such as Tomcat, on the server where Jenkins is installed. Here we will be using Tomcat.
    
* Install Tomcat:
    
    ```plaintext
    sudo apt-get update
    sudo apt-get install tomcat8 -y
    
    ```
    
* Configure the Tomcat user by setting up a new user with the appropriate permissions to access the server. Edit the Tomcat configuration file with the following:  
    Add the following lines before the line:
    
    ```plaintext
    sudo nano /etc/tomcat8/tomcat-users.xml
    
    ```
    

````plaintext
```
<role rolename="manager-gui"/>
<user username="tomcat" password="password" roles="manager-gui"/>

```
````

* Restart Tomcat: `sudo systemctl restart tomcat8`
    

**Step 6: Create a Jenkinsfile**

Jenkinsfile is a powerful tool used in the Jenkins automation server to define the entire build process of a project. It provides a declarative and programmatic way of defining the build pipeline, enabling the entire process to be versioned, reviewed, and tested just like any other codebase.

Jenkinsfile allows for the automation of the entire build process, from code compilation to the delivery of the final product. With Jenkinsfile, developers can define every stage of the pipeline, including the build, test, and deployment phases, as well as any custom stages that are specific to their project.

This level of automation can significantly reduce the time and effort required to build and deploy complex applications, as well as ensure consistency across all deployments. Jenkinsfile is also designed to be highly customizable, with a wide range of plugins and integrations available to extend its functionality and tailor it to specific project requirements. To create a Jenkinsfile, follow these steps:

1. In the root directory of your GitHub repository, create a new file named "Jenkinsfile".
    
2. Open the file in a text editor and add the following script:
    

```plaintext
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // Checkout the code from the GitHub repository
                git url: '<https://github.com/yourusername/yourrepository.git>'

                // Build the Maven project
                sh 'mvn clean package'

                // Archive the artifact
                archiveArtifacts artifacts: 'target/*.war'
            }
        }
        stage('Deploy') {
            steps {
                // Copy the artifact to the application server
                sshPublisher(
                    continueOnError: false,
                    failOnError: true,
                    publishers: [
                        sshPublisherDesc(
                            configName: 'yourserver',
                            transfers: [
                               sshTransfer(
                                    cleanRemote: false,
                                    excludes: '',
                                    execCommand: '',
                                    execTimeout: 120000,
                                    flatten: false,
                                    makeEmptyDirs: false,
                                    noDefaultExcludes: false,
                                    patternSeparator: '[, ]+',
                                    remoteDirectory: '/usr/share/tomcat8/webapps/',
                                    remoteDirectorySDF: false,
                                    removePrefix: '',
                                    sourceFiles: 'target/*.war'
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }
}

```

The script defines a two-stage pipeline: Build and Deploy. In the Build stage, the code is checked out from the GitHub repository, the Maven project is built, and the artifact is archived. In the Deploy stage, the artifact is copied to the application server using the SSH plugin.

Note: Make sure to replace &lt;your username&gt; and &lt;your repository&gt; with your GitHub username and repository name, respectively. Also, replace '/usr/share/tomcat8/webapps/' with the path to the webapps directory on your application server.

**Step 7: Set up GitHub Webhook**

GitHub webhook is a feature of the GitHub platform that allows for the automatic triggering of events when certain actions are performed on a GitHub repository. It enables developers to automate their development workflow, increasing productivity and efficiency.

GitHub webhook works by sending HTTP requests to a URL configured to receive these requests whenever an event occurs in a GitHub repository. These events can include push notifications, pull requests, and issue comments, among others. When the webhook receives an event, it triggers a set of actions, such as running tests, deploying code, or sending notifications.

GitHub webhook can be integrated with a wide range of third-party tools and services, allowing developers to automate almost any aspect of their development workflow. For example, it can be used to trigger a build in Jenkins or another continuous integration tool or to automatically deploy code to a production server. To trigger the Jenkins job automatically when changes are pushed to the GitHub repository, set up a GitHub webhook. To do this, follow these steps:

1. Go to the repository page on GitHub and select "Settings" -&gt; "Webhooks" -&gt; "Add webhook".
    
2. Enter the Payload URL of your Jenkins server followed by "/github-webhook/" (e.g., [http://yourjenkinsserver.com/github-webhook/](http://yourjenkinsserver.com/github-webhook/)).
    
3. Select "application/json" as the content type and choose "Let me select individual events".
    
4. Check the "Push" and "Pull Request" events.
    
5. Click "Add webhook" to save the configuration.
    

That's it! Now, every time changes are pushed to the GitHub repository, the Jenkins job will be triggered automatically, and the application will be built and deployed to the application server.

Thank You!