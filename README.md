Build Milestone
=====================

#### Team members:
* Mark Luo (tluo@ncsu.edu)
* Bhashwanth Kadapagunta (bkadapa@ncsu.edu)


## 1. Introduction

In order to demonstrate the functioning of the build server, we are using a forked version of the project [JBake](http://jbake.org/), which is a Java-based open source blog generator for developers. JBake can create a complete site structure using contents and templates. It is an Apache Maven based project as it relies on Maven to manage dependency, build project, and perform tests. We are using [Jenkins](http://jenkins-ci.org/) to develop continuous integration pipeline for our project. It is a server-based system running in a servlet container, which can execute Apache Ant and Apache Maven based projects. This is ideal for our build server as it will automate push detection, build, test, and distribution via Jenkins.

## 2. Build

### 2.1 Target Project

The [JBake project](https://github.com/macluo/jbake) is currently forked on Github.com. A pom.xml file (Maven project object model) has been provided by the author and therefore it does not require additional testing and editing. All the Java source codes are contained inside the /src/main/ directory. Test files are also available and can be found in /src/test/ directory. When Maven is invoked it will also perform testing after the project is succesfully built. Testing can be skipped by running Maven with a '-Dmaven.test.skip=true' option.


### 2.2 Setup

#### Machine Setup:

The main build server was built on an Amazon EC2 cloud instance, a 64 bit Ubuntu Linux box where Git, Maven, Java, and Jenkins were installed. Because our Jenkins server is hosted on a public Amazon cloud, security is one of our priorities. Therefore the Global Security feature on Jenkins should be enabled. Github OAuth plugin has been installed on Jenkins in order to connect Jenkins to the user account at Github.com.

![Githut Security](/images/1.Jenkins_security.png)

#### Build Trigger Setup at Github.com

Although Jenkins can constantly check the changes in Github respository, this is not very efficient and may waste internet bandwidth. The ideal setup is to let Github send a build command to our build server on EC2 once a commit has been pushed to Github. To enable push detection at Github, a service hook must be enabled for the JBake repo. 

![Github Webhook](/images/2.webhook_on_Github.png)

Jenkins utilizes a very unique verification process for build trigger. When Jenkins receives a build command from external site, it calls back to verify the authentication, which prevents Jenkins from malicious attacks while verifying user access at the same time. To enable this feature, a new application should be registered on Github user account.

![Jenkins Webhook](/images/3.Github_application_setup.png)

A personal token for Jenkins plugin has also been created on Github, this allows Jenkins to update its build status with Github after the build is successful. This token should be entered on Jenkins's GitHub Web Hook section too.

![Jenkins Webhook](/images/4.Jenkins_webhook.png)

#### Jenkins Project Setup:

To set up a build project on Jenkins, a project "JBake-Github" has been created. Please note that we have also created additional project to test the local commit trigger on EC2 instance. Two additional Jenkins plugin were installed before proceeding to the next step and there are Git plugin and Github plugin. They can be found on the Configure Jenkins page.

![Jenkins Webhook](/images/5.Jenkins_Projects.png)

On the job configuration page, the Git repo link, Maven build info including the Root POM.xml file and Maven options should be completed. We also selected "Run only if build succeeds" option as the post-build steps and have enabled artifacts archive feature so developers can access to the zip files of the early versions.

![Jenkins Webhook](/images/6.Jenkins_Project_setup.png)

#### Jenkins Slave Setup:

We setup the slave on another instance of EC2 Ubuntu image on which Java, maven and Git were installed. We then installed the ssh server on the machine, added a slave user and generated a rsa key to enable the master to connect to the slave. On the master machine, we created a new slave node and connected it to the slave machine using the generated credentials. The connection status of the slave node can be seen in the following image.

![Jenkins slave connection](/images/SlaveConnected.png)

Now, the slave is configured to build the project based on the master machines requirement. The configuration details of the slave node are as shown below.

![Jenkins slave connection](/images/SlaveConfig.png)

We have included all the Jenkins configuration files (.xml files) inside the [/Code/Jenkins/](https://github.ncsu.edu/tluo/CSC-DevOps-Milestone1/tree/master/Code/jenkins) directory.


## 3. Results


#### 3.1 Build Trigger

The following build was triggered on the server by a git push at the local machine. Please note that the hash keys at local machine and on Github begins with 50ac9ca and bcc7efc, respectively.

![Git push](/images/Git_push.png)

As highlighted on the image below, a build was started by GitHub by a user with an assigned hash key beginning with bcc7efc.

![Build Trigger](/images/BuildTrigger.png)


#### 3.2 Dependency Management and Clean State

The dependency management of the project was carried out by Maven with dependency of the build defined in the POM.xml file. The following image shows the project fingerprints, which contains a list of libraries and tools required by the JBake project.

![JBake Dependency](/images/Project_fingerprints.png)

To restore the build server to the clean state, we have enabled "Clean Install" as the Maven build option. The result can be seen from the attached image below as well as on the image in the section 3.3.

![Clean State Restore](/images/Clean_state.png)

#### 3.3 Build Script Execution

Because Jenkins is used by our build server to carry out the Maven building task, there is no particular build script to code. We have attached the build result from the console. The complete build script output can be found in the [JBake_console_output.txt](/Console/JBake_console_output.txt) file.


![JBake Build Results](/images/JBake_console_output.png)

#### 3.4 Multiple Nodes

The build can be run on multiple nodes. We created 2 slave executers besides the master node to perform this task. Build jobs running simultaneously on the master and the slave nodes can be seen in the following image.

![Multiple Nodes](/images/MultipleNodes.png)

#### 3.5 Build Status

By default the build status for Jenkins project is available on server's Jenkins page via http request to port 8080. This page is public without any restriction. However, this isn't very convenient. Hence, we have also set up Nginx as the proxy server to redirect HTTP request at port 80 to Jenkins server. As a result, the recent build status of our project can be seen by entering the build server IP address alone. The build status of our project can be seen at this address: [http://52.0.209.194/](http://52.0.209.194/). The configuration file [proxy.conf](/Code/proxy.conf) for Nginx has also been included.


![Build Status](/images/Web_status-highlighted.png)
