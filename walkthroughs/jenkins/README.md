---
title: "Walkthrough: Jenkins"
---

Follow along with the instructor as we configure Jenkins.

## Our Continous Integration Goals
After a branch/story is merged into our `master` branch. We want...
1. Make sure the code compiles
2. Makes sure the tests pass
3. Deliver a `.jar` file that is ready to be deployed

## Install and Configure Jenkins
- Download jenkins war from [jenkins home page](https://jenkins.io/download/) by clicking *Generic Java War*  (note: scroll to the bottom of the page)
![Download Jenkins - Generic War](../../materials/week05/download-jenkins.png)
- Copy the `jenkins.war` file the folder to `~/jenkins`
- Now start jenkins via terminal
```
$ java -jar ~/jenkins/jenkins.war --httpPort=9090
```
<aside class="aside-note" markdown="1">
Normally you would not install jenkins on your dev machine. You would isntall it on a server that would be continsouly running so that it could react to a commit at anytime.
</aside>

#### Configure Jenkins
- Go to [http://localhost:9090](http://localhost:9090)
- You should see screen asking for an admin key
- Access the file listed in the terminal and copy the key inside it
- Paste that key into the input box

#### Install Plugins
- Click install suggested plugins (this will take a couple minutes)

#### Create a Jenkins User
- This will be how you login to jenkins going forward
- Be sure to remember the username and password

## Create Project to Compile Airwaze
- Click *New Item*
- Enter name `Airwaze Compile`
- Click *Freestyle Project*
- Click *Ok* at bottom

#### Configure the Compile Project
- In *Source Code Management* click *Git*
- Post your SSH gitlab repo url into *Repository URL*. Example: git@gitlab.com:welzie/zika-cdc-dashboard.git
- Make sure you the branch you want to compile is in the *Branch Specifier* field 
- Go to the *Build Triggers* section
- Select *Poll SCM* and enter `H/5 * * * *` into the *Schedule* input 
- Go to the *Build* section
- Click *Add build step*
- Click *Invoke Gradle script*
- Enter `clean compileJava` into the *Tasks* input

Now test out the build by clicking *Build Now*

#### We Need to Install a Plugin
- Go to your jenkins home page `http://localhost:9090`
- Click *Manage Jenkins* on the left
- Click *Manage Plugins* on the right
- Click *Available*
- Enter *Parameterized Trigger* in search box
- Install *Parameterized Trigger plugin* without restarting

## Create Test, CreateJar, and DeployToS3 Projects
- Create three more *Freestyle* projects
- `Airwaze Test`
- `Airwaze CreateJar` 
- `Airwaze Deliver`
- Don't do anything but give these a name. We will configure them next.

### Edit the Compile Project
We need the *Compile Project* to kick off the *Test Project* when it's done. We also want the two projects to share the same work space, so that the repo doesn't have to be checked out again.
- Navigate to project `http://localhost:9090/job/Airwaze%20Compile/`
- Click *Configure*
- Go to *Post Build Actions*
- Select *Trigger parameterized build on other projects* from the select box
- Enter `Airwaze Test` as the project to build
- Click *Add Parameters* and select *Build on the same node*
- Click *Add Parameters* again and select *Predefined parameters*
- Enter this `AIRWAZE_WORKSPACE=${WORKSPACE}` into input
- Click save

#### Configure Test Project
- Navigate to project `http://localhost:9090/job/Airwaze%20Test/`
- In *General* select *This project is parameterized*
![String Parameter](../../materials/week05/parameter-project-1.png)
- Paste this `AIRWAZE_WORKSPACE` into *name* input
![Enter parameter name](../../materials/week05/parameter-project-2.png)
- Click *Advanced* button and select *Custom Workspace*
- Enter `${AIRWAZE_WORKSPACE}` in the input
![Custom Workspace Direstory](../../materials/week05/parameter-project-3.png)
- Go to the *Build* section
- Click *Add build step*
- Click *Invode Gradle script*
- Enter `clean test` into the *Tasks* input
Now we need to kick off the *CreateJar Project*
- Go to *Post Build Actions*
- Enter `Airwaze CreateJar` as the project to build
- Click *Add Parameters* and select *Build on the same node*
- Click *Add Parameters* again and select *Predefined parameters*
- Enter this `AIRWAZE_WORKSPACE=${WORKSPACE}` into input
- Click save

####Run the Compile Project, which runs the Test Project
- Run the Compile Project
- After both the Compile Project and Test Project have finished
- You can view the tests by finding the test results in the project work space
- Naviage to project works space by clicking *Work Space* in the left menu of a project. Example: [http://localhost:9090/job/Airwaze%20Test/ws/](http://localhost:9090/job/Airwaze%20Test/ws/)
- Once on the *Work Space* page click on the folder names and navigate to `/build/reports/tests/test/index.html`
- Clicking on `index.html` should open up the junit test results. Example: [http://localhost:9090/job/Airwaze%20Test/ws/build/reports/tests/test/index.html])(http://localhost:9090/job/Airwaze%20Test/ws/build/reports/tests/test/index.html)

#### Configure the Tests Results to be Published Automatically
- We can configure the tests results to be pushlised on the project results after every run
- Go to the *Post build actions* for the *Test Project*
- Select *Publish JUnit test result report* and input this `build/test-results/test/*.xml` into input
- Run the project again and you will see a link named *Latest Test Results* on the project page
- You can also click on a specific build and see a link named *Test Results*
- NOTE: a graph will appear on the project page that shows a history of test results

#### Configure CreateJar Project
- Same configuration as the *Test Project*, with these exceptions
- In the *Build* section run this gradle command `bootRepackage`
- In *Post Build Actions* project to build enter `Airwaze Deliver`
- Do not have the test results copied as we did with the *Test Project*

#### Setup S3 Bucket (Needed so we can configure the next project)
 - If you haven't already, you need to install `awscli`. Instructions can be found in the [AWS3 Studio](https://education.launchcode.org/gis-devops/studios/AWS3/)
 - Create a new S3 bucket that will used for the `.jar` files your jenkins builds produce
 - Be sure to create the new bucket with **VERSIONING** enabled
 
 Make sure your s3 bucket shows up when you run this command in terminal
 ```
 $ aws s3 ls
 ```

#### Configure Deliver Project
- Same configuration as *CreateJar Project*, with these two exceptions
- In the *Build* section select *Execute shell*
- Enter this into input `aws s3 cp build/libs/app-0.0.1-SNAPSHOT.jar s3://YOUR-S3-BUCKET/`
- There are NO *Post Build Actions*

##That's It!
Now run the *Airwaze Compile* project now and watch it kick off the other projects automatically!
