# Triggering a Jenkins build as a task on XebiaLabs

This guide will demonstrate how to set a XebiaLabs task to trigger a Jenkins build, which will run an automation test on Kobiton cloud devices. 

## Prerequisites

* Jenkins is installed

* Have an account with XebiaLabs

* Have an account with Kobiton

## 1. XebiaLabs Configuration

### 1.1 Install the Jenkins plugin in XebiaLabs

Log in to XebiaLabs. Click on 'Plugins' in the top navigation bar. In the search bar, type in 'Jenkins' and install it. 

![jenkins-plugin](assets/jenkins-plugin.png)

### 1.2 Update Jenkins plugin settings

In the navigation bar, click on 'Settings' and then 'Shared configuration'. 

Under 'Jenkins: Server', click on 'Add Server'. 

![add-server](assets/add-server.png)

Enter in a title for the configuration, as well as the Jenkins url where the server can be reached. 

Under 'Authentication', enter in your Jenkins username and password. 

![server-settings](assets/server-settings.png)

### 1.3 Set Jenkins as a task

If you have not created a release yet, follow [this link](./planning-release.md) for guidance on how.

Once your release is created, we will add a task that will trigger a Jenkins build. 

Click on 'Add task' under the desired phase. 

![add-task](assets/add-task.png)

Set a name for the new task. Click on the dropdown menu and select 'Jenkins' and 'Build'. Then click the 'Add' button to finish creating the task. 

![build](assets/build.png)

Click on the task to open up the settings. Enter in the Server name, which was created earlier. Enter your Jenkins username and password, as well as the name of the job you want to trigger in Jenkins. 

![task-settings](assets/task-settings.png)

Exit out of the settings and start your release. You have successfully created a task that will trigger a Jenkins build!

Once you start your release, you can check the log output in your task to see if the build was successful.  

![log-output](assets/log-output.png)

## 2. Jenkins Configuration

### 2.1 Prepare Kobiton configuration for executing automation testing

* #### Username 
Go to https://portal.kobiton.com/

In the upper right hand corner, click on your name and in the drop down menu, click on Profile. 

![profile](assets/2-profile-user.png)

You should see the username. 

![username](assets/2-username.png)

* #### API key
Click on your name in the upper righthand corner again and select settings. 

You should be able to find your API key under 'API Keys'. 

![api-key](assets/2-apikey.png)

* #### Desired cap
In the navigation bar at the top of the Kobiton website, select Devices. 

Hover over any device and click on the Automation settings button (the gear symbol). 

![automation](assets/2-automation.png)

On the left hand side, you can select your preferred language, as well as any other variables you would like to adjust, such as **App Type**, **Device Group**, and **Orientation**. Adjusting the settings on the left side will affect the desiredCaps, which you can find in the right side of the window. 

![automation-settings](assets/2-automationsettings.png)

### 2.2 Create new Jenkins project to execute the automation test

#### How to setup Jenkins to run automation testing

Jenkins is an open source automation server that provides hundreds of plugins to support building, deploying, and automating any project. 

Download Jenkins from https://jenkins.io/ and make sure you have a Jenkins server available with a public domain. 

**1. Write the automation test script**

* How to write a simple test

For examples of automation tests, go to https://github.com/kobiton/samples . 

Choose a language for your test script, and decide whether you want to test on Android or iOS, and either do a web test or an app test. Make sure in the code you specify your Kobiton username, API key, and information under desiredCaps. 

**2. Create Jenkins project**

On the left side on the page, click on New Item to start a new Jenkins project. 

![new-item](assets/2-new-item.png)

Enter in a name for the project and select 'Freestyle project'. Hit 'OK'. 

![project](assets/2-project.png)

Go to Configure in your Jenkins project. Add a build step, execute shell. 

![build-execute-shell](assets/2-build-execute-shell.png)

Enter this in the command. Enter your own Kobiton  username and API key as well as the desired device and platform you wish to test with. The below example will execute the android-web-test script sample from Kobiton. 
```
#!/bin/bash
[ ! -d "samples" ] && git clone https://github.com/kobiton/samples.git || echo "Samples already exist"
cd samples/javascript
npm install
KOBITON_API_KEY=... KOBITON_USERNAME=... npm run android-web-test
```

This guide is executing the NodeJS test script from the Kobiton samples, so we will add the NodeJS Plugin on Jenkins to execute NodeJS script as a build step. To do this, go to 'Manage Jenkins' -> 'Manage Plugins' -> Available. Search for NodeJS and install it.

![manage-plugins](assets/2-manageplugins.png)

If you are using a different language, you may try to find a suitable plugin for your project if necessary.

Under build environment, check the box that says 'Provide Node & npm bin/ folder to PATH'.

![build-environment](assets/2-build-environment.png)

**3. Test**

Start your XL Release and check Kobiton cloud devices to see if a test session was created. 

**4. Get the automation session data through Kobiton REST API**

- Update test result to session (https://api.kobiton.com/docs/#update-session-information)

```
PUT /sessions/{sessionId}
``` 
To make a request:

1. Encode your credential in base64 for HTTP Basic Authentication, you may use below command and note the text result

```
echo -n <your username>:<your api-key> | base64
```
2. Use encoded base64 from above in curl commands, like below

```
curl -X GET https://api.kobiton.com/v1/sessions \
  -H 'Authorization: Basic dGVzdHVzZXI6MTIzZWQtMTIzZmFjLTkxMzdkY2E='
  -H 'Accept: application/json'
```

Below are necessary Kobiton Rest API endpoints that you may need.

- [Get session info](https://api.kobiton.com/docs/#get-a-session)
```
GET /sessions{sessionId}
```
To make a request:
```
curl -X GET https://api.kobiton.com/v1/sessions/{sessionId} \
  -H 'Authorization: Basic dGVzdHVzZXI6MTIzZWQtMTIzZmFjLTkxMzdkY2E='
  -H 'Accept: application/json'
```

With this line of code, you can print the session information to the console. 
```javascript
console.log(sessionCapabilities)
```

- [Get session commands](https://api.kobiton.com/docs/#get-session-commands)
```
GET /sessions/{sessionId}/commands
```
To make a request:

```Shell
curl -X GET https://api.kobiton.com/v1/sessions/{sessionId}/commands \
  -H 'Authorization: Basic dGVzdHVzZXI6MTIzZWQtMTIzZmFjLTkxMzdkY2E='
  -H 'Accept: application/json'

```
To get to a certain page in your commands, add the page number to the commands URL. For example:
```javascript
`https://api-test.kobiton.com/v1/sessions/${sessionId}/commands?page=2`
```

> For more details on how to retrieve information about your session, go to https://api.kobiton.com/docs/

## Final result

The test is a either a success or failure.

#### Failure case

* Error: "The environment you requested was unavailable." 
    - This means that the device you selected is already booked. Either select a different device or wait a few moments until your device becomes available
* Other 
    - Contact Kobiton for support
    - Go to portal.kobiton.com
    - In the navigation bar at the top of the page, click on 'Support'

    ![support](assets/2-support.png)

    - Fill in the information for your request and submit your ticket

    ![submit-ticket](assets/2-submit-ticket.png)
