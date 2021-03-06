# Milestone 2

### Team Members
1. Kumar Utsav (kutsav)
2. Raman Preet Singh (rpsingh2)
3. Rohit Arora (rarora4)

### Description

We are using the following node js project as our target project: 
```
https://github.com/DevOps-HeadBangers/File-upload-in-Node.git
```
A build to this project installs all the dependencies and also runs the test cases.

### Tools Used

Jenkins Build Server

NPM Package Manager

Apache Tomcat 8 

Git

Tape Testing Framework

Esprima

ESLint

Istanbul

### Basic Setup

#### Install Jenkins 

1. Download and [install Tomcat8](http://www.liquidweb.com/kb/how-to-install-apache-tomcat-8-on-ubuntu-14-04/) on your system.

2. Download [jenkins.war](http://mirrors.jenkins-ci.org/war/latest/jenkins.war) file.

3. Place the war file in the /opt/tomcat/webapps directory.

4. Run tomcat server using bin/startup.sh file.

5. Open browser and go to [http://localhost:8080/jenkins](http://localhost:8080/jenkins) to run jenkins. 

#### Configure Jenkins

1. Go to Jenkins on the top left corner and choose Manage Jenkins -> Manage Plugins.

2. Install GitHub Plugin, TAP Plugin, CheckStyle Plugin, Clover Coverage Plugin and NodeJS Plugin from there.

3. Now, go to Manage Jenkins -> Configure System -> NodeJS. Click Add NodeJS and click install automatically. This will give you a list of node versions. Choose according to the version installed on your system. Save.

4. When you are done, save.

#### Create Jobs

1. Create a new job by clicking on Create New Job.

2. Provide the job name, choose freestyle project and click OK.

3. Now configure your job to add the target git project to the job by choosing Git from Source Code Management         Options and enter the git repository.

4. Under Build Environment, check the 'Provide Node & npm bin/ folder to PATH' option and choose the NodeJS            installation from the previous step.


### Test Section

##### The ability to run unit tests, measure coverage, and report the results

1. Go to the Configure page of your job.

2. Under Build section, choose 'Execute Shell' option for Add Build Step option.

3. Add the following code to the 'Execute Shell Command'. This script will do a clean npm install, run unit tests,     measure coverage and report the results.  
    ```
    rm -rf node_modules
    npm install
    npm run test
    ```
    
4. Under 'Post Build Sections', choose 'Publish TAP Results' for 'Add Post Build Action' option. And, for 'Test        Results' textbox, write 'test.tap'. 'test.tap' file will contain the result of ```istanbul cover test.js```.

5. Again, under 'Post Build Sections', choose 'Publish Clover Coverage Report' for 'Add Post Build Action' option.     And, for 'Clover Report Directory' textbox, write 'coverage'. 'coverage' is the folder that will contain the        istanbul coverage reports. Click 'Apply' and 'Save' when done. Run the build.

Following is the screencast for the capability:

![Capability 1](https://github.com/DevOps-HeadBangers/Milestone2/blob/master/GIFs/Cap1.gif) 

##### The ability to improve testing coverage using one of the techniques covered in class: constraint-based test generation, fuzzing, etc. You can use an existing tool or implement your own approach.

To the 'Execute Shell Command' of the previous capability, add ```node testgen.js```. Adding this command will generate new test cases and improve the test coverage. The tests are generated using constraint based test        generation. Click 'Apply' and 'Save' when done. Run the build.

Following is the screencast for the capability:

![Capability 2](https://github.com/DevOps-HeadBangers/Milestone2/blob/master/GIFs/Cap2.gif) 

### Analysis Section

##### The ability to run an existing static analysis tool on the source code (e.g. FindBugs, PMD, CheckStyle, NCover, Lint, etc.), process its results, and report its findings.

For this capability, to the 'Execute Shell Command' of the previous capability, add ```npm run lint || :```.  Under 'Post Build Sections', choose 'Publish CheckStyle Analysis Results' for 'Add Post Build Action' option. Click 'Apply' and 'Save' when done. Run the build. During the build, eslint will check the ```server.js``` file for style errors and report the errors as warnings in checkstyle-result.xml. This xml file will be used to generate the final result on Jenkins.

Following is the screencast for the capability:

![Capability 3](https://github.com/DevOps-HeadBangers/Milestone2/blob/master/GIFs/Cap3.gif) 

##### The ability to extend an existing analysis tool with a custom analysis, or implement a new analysis from scratch. For example, you could write a static analysis that checks for the ratio of comments to code, or finds parse errors in SQL string statements. You could introduce security checks, a dynamic analysis, a data-flow analysis or a data-flow based test coverage.

For this capability, add ```node analysis.js``` to the 'Execute Shell Command' of the previous capability. Click 'Apply' and 'Save' when done. Run the build. During the build, the 'analysis' build will run and output the comment to code line ratio of all the js files in the project on the console output. 

Following is the screencast for the capability:

![Capability 4](https://github.com/DevOps-HeadBangers/Milestone2/blob/master/GIFs/Cap4.gif) 

##### Using hooks or post-build scripts, have the ability to reject a commit if it fails a minimum testing criteria (e.g. failed test case, or less than 50% statement coverage) and analysis criteria (e.g. cannot commits that generate a particular FindBugs rule, such as "Method concatenates strings using + in a loop").

###### Part 1 : Failing minimum testing criteria

Go to the build section in the job configuration and add a new 'Execute Shell' box. Add the following code to it:
```
node extendTest.js
if [ $? -eq "1" ];
then
  exit 1
fi
```
The above shell script will run the script ```extendTest.js``` which analyzes the test results and looks for failed test cases. If there is any failed test case, the script will exit with exit code 1 which will fail the build. 

Click 'Apply' and 'Save' when done. Run the build.

Following is the screencast for the capability:

![Capability 5_1](https://github.com/DevOps-HeadBangers/Milestone2/blob/master/GIFs/Cap5_1.gif) 

###### Part 2 : Failing minimum analysis criteria

Go to the build section in the job configuration and add a new 'Execute Shell' box. Add the following code to it:
```
node extendAnalysis.js
if [ $? -eq "1" ];
then
  exit 1
fi
```
The above shell script will run the node script ```extendAnalysis.js``` which sets a threshold on the number of errors given by lint analysis on the server code. It compares the threshold value with the actual number of lint errors and fails the build if the actual number exceeds the threshold.

Click 'Apply' and 'Save' when done. Run the build.

Following is the screencast for the capability:

![Capability 5_2](https://github.com/DevOps-HeadBangers/Milestone2/blob/master/GIFs/Cap5_2.gif) 

##### The ability to parse a code files and json files in order to detect the presence of AWS/digital ocean security tokens. The ability to check commited files that are private ssh keys. Using hooks, reject the commit if any violation occurs.

###### Part 1 : Testing for presence of keys in one of the files.

We are testing for AWS and Digital Ocean keys. Through, our observation of various keys we are assuming that Digital Ocean keys are alphanumeric and of 64 length. And, AWS keys are also alphanumeric starting with "AKI" and its length varies between 20 and 21. We are also assuming that the key will always be in quotes. 

When the commit is made, the hook will run the ```checkForKey.js``` node script, which will parse all the files and look for the keys.

If any such key is found in the commit, the commit is rejected and an appropriate message is prompted on the terminal.

We added a key to the ```server.js``` file and then tried to commit. The pre-commit hook ran the script, identified that there is a key in the commit and rejected the commit.

![Capability 6_1](https://github.com/DevOps-HeadBangers/Milestone2/blob/master/GIFs/Cap6_1.gif) 

###### Part 2 : Testing for PEM file which has private SSH key.

We are assuming that files containing private SSH key have .pem extension.

When the commit is made, the hook will run the ```checkForKey.js``` node script, which will look for .pem files.

If any such file is found, the commit is rejected and an appropriate message is prompted on the terminal.

We created a file with .pem extension to test whether this capability is able to find that file and reject the commit.

![Capability 6_2](https://github.com/DevOps-HeadBangers/Milestone2/blob/master/GIFs/Cap6_2.gif) 
