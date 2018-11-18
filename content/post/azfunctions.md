---
date: "2017-05-13T11:10:12+08:00"
tags: ["Development", "Python", "Azure", "Functions", "Strava" ]
title: Hacking Azure Functions in Python

---

*Caveat: Support for Python on Azure Functions is still experimental so, some of the steps below might not be applicable anymore.*

# Overview
Azure Functions allow running *"serverless"* code in the cloud (and more recently [on-premises](https://azure.microsoft.com/en-us/blog/introducing-azure-functions-runtime-preview/)).

I wanted to automate the capture of my running data so, this looked like an interesting scenario to try out Azure Functions.

In a nutshell, I will be looking at:

1. Deployment of non-standard version of Python (3.5) to support my ode
2. Installing additional non-standard modules to extract my running data from STRAVA (Python stravalib)
3. Deploying the code

***

# Setting up the environment

1. Create a new Function App on Azure Portal 
2. Add a custom function to the Funtion App you just created, choose Python as the Language. *You currently only have the option to choose a Queue Trigger using Python. You can change this later by customizing the functions.json template*

![Azure Function Python Code](/blog/img/stravaazurefunction.PNG "Azure Function Python Code")

3. You will need to customize the deploymentment by accessing the Kudu environment at:

```
   http://{your function app name}.scm.azurewebsites.net/DebugConsole
```

![Kudu environment](/blog/img/kudu.PNG "Kudu environment")

# Deployment of Python 3.5
Since Azure Functions currently come with Python 2.7 and I needed 3.5, I followed the guidance [here](https://github.com/Azure/azure-webjobs-sdk-script/wiki/Using-a-custom-version-of-Python).

There was also the need to unzip the python35.zip file as pip was not able to properly install the packages I needed:

```
move python35.zip a.zip
move a.zip python35.zip
d:\7zip\7za.exe x a.zip 
```

Next came the PIP instalation (in case you don't get it with the pythin distro you need):
```
Download get-pip.py
Zip get-pip.oy
Upload get-pip.ZIP
D:\home\site\tools\python.exe get-pip.py
```

# Installing Additional Modules
I will need to execute the pip install on the Kudu Debug Console in order to get the stravalib python package.

```
D:\home\site\tools\python.exe -m pip install stravalib
```

# Deploying Code
I chose to manually deploy the following code directly on the Azure Portal Azure Function editor.

```
from stravalib.client import Client

ACCESS_TOKEN = "INSERT YOUR STRAVA ACCESS TOKEN"

# Log In
print("Logging into Strava...")
client = Client()
client.access_token = ACCESS_TOKEN

for activity in client.get_activities():
    print('Exercised on {date}'.format(date=activity.start_date_local))
```

You could alternatively look at deploying from GitHub or other deployment sources. More details [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-continuous-deployment)

You can run this code on the Azure Portal, invoque run.py on the kudu environemnt or, change the function.json file so you run this on a schedule as a TimerTrigger Function.

***

# Resources
Azure Functions - https://azure.microsoft.com/en-us/services/functions/

stravalib for Python - https://github.com/hozn/stravalib