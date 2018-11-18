+++
date = "2018-11-18T21:35:08+08:00"
tags = ["Deep Learning", "Object detection", "Object Classification"]
title = "Building a Planespotting AI demo"
+++

# Introduction
This post is aimed at providing some additional background into the Plane Spotting Object Detection/Classification demo that I recently built and uploaded to GitHub.

Latest code is available [here](https://github.com/joaobi/planespotter).  
There is a web demo available [here](http://planespotter-demo.azurewebsites.net/).

The demo is composed of a website that allows you to upload the image of a plane and receive the same images with bounding boxes on the planes it detects and the airline they belong to.

![Plane detection and Airline Classification](/blog/img/planes.jpg "Plane detection and Airline Classification")

It currently detects planes from: Emirates, Singapore Airlines, ANA, Asiana, Korean Airlines and Qantas  

## Screenshots of Demo

The demo is [here](http://planespotter-demo.azurewebsites.net/).   
![Plane detection and Airline Classification](/blog/img/homepage.jpg "Plane detection and Airline Classification")  
![Plane detection and Airline Classification](/blog/img/gallery.jpg "Plane detection and Airline Classification")

# Additional Technical Details
The projects is built using:

+ [Tensorflow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection) - The object (plane) detection API used. Used the standard MSCOCO image library
+ Custom airline classifier model built on Keras using the Tensorflow backend - the model is included in GitHub repo
+ All code (machine learning and web) is on Python (plus some minimal html, JavaScript)   

# Deployment
 Code was tested on docker running locally on windows/MacOs, on Azure Container Instances, Azure App Service, Azure Kubernetes Services (AKS) and manually (git pull + manual run of the bash script).

