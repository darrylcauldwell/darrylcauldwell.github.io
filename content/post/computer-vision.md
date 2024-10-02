+++
title = "Building a Portable Real-Time Face Detection Web App with OpenCV, Flask, and Docker"
date = "2024-10-01"
description = "Learn how to create a face recognition web application using Python Flask, OpenCV, and package it as a portal Docker image."
tags = [
    "python",
    "docker"
    ]
categories = [
    "pipeline",
    "ai"
    ]
thumbnail = "clarity-icons/code-144.svg"
+++

Face detection is a powerful computer vision capability with wide-ranging applications across industries. In this post, I'll describe how I built a demo application which performs real-time face detection on a video stream. This application is a flexible platform for demonstrating AI inference capabilities across diverse deployment scenarios, from edge devices to cloud environments. To ensure this broad compatibility and ease of deployment, I packaged the application into a portable Docker image that runs on ARM64 and AMD64 architectures.

## Project Overview

The key components of this application are:

* OpenCV for face detection using Haar cascades
* Python Flask web framework to serve the application
* RTSP video stream as the input source
* Docker for containerization and cross-platform compatibility

## Implementing the Application

The web application is built using Flask, a lightweight Python web framework. I selected Flask due to its simplicity and the widespread familiarity of Python, making it accessible to a broader audience. My goal is to ensure that the application is easy to understand and engage with for users of varying technical backgrounds.

The latest version of the application code can be found [here app.py](https://github.com/darrylcauldwell/keswick/blob/main/face-detector/app.py).

The first function detect_faces does the following:

* Takes input of a frame of video
* Uses OpenCV to load the face detection classifier
* Uses OpenCV to convert the frame from colour to grayscale for face detection
* Detects faces in the grayscale frame
* Draw a green rectangle around the detected face

The second function generate_frames does the following:

* Gets the URL of the RTSP stream from environment variable
* For each frame call the detect_faces function

The Flask application uses a template to format the webpage which can be found [here index.html](https://github.com/darrylcauldwell/keswick/blob/main/face-detector/templates/index.html).

To demonstrate a simple change to the application it is each to change either the:

* background-colour attribute of the index.html template to another colour
* cv2.data.haarcascades attribute of the app.py application to another pre-build haar cascades model

## Building for Multiple Architectures

I chose Ubuntu to develop the Docker container images due to its official support for both amd64 and arm64. It has broad software compatibility, ensuring that most tools and libraries should be available and should work. It also has a vast user base and extensive documentation, which can be helpful when troubleshooting.

As this is a demo application I wanted any changes I make to the application to be applied quickly. To do this I created a base ubuntu image with OpenCV, Python3 and other tools I depend upon later.

The latest versions of Dockerfile and Python requirements.txt can be found [here](https://github.com/darrylcauldwell/keswick/tree/main/opencv).  I used docker buildx to create arm64 and amd64 images, the latest version can be found [here](ghcr.io/darrylcauldwell/opencv:latest).

With the base opencv container image in place I then have a second Dockerfile which layers on the Flask application. The latest versions of Dockerfile can be found [here](https://github.com/darrylcauldwell/keswick/blob/main/face-detector/Dockerfile) and latest images can be found [here](https://github.com/users/darrylcauldwell/packages/container/keswick/282184304?tag=latest).

## Conclusion

This project demonstrates how to combine OpenCV's powerful computer vision capabilities with Flask's web serving to create a real-time face detection application. By packaging it as a multi-architecture Docker image, we've made it highly portable and easy to deploy on a wide range of systems. The use of Docker not only simplifies deployment but also makes the application more accessible to users and developers working with different hardware architectures. This approach showcases how modern development practices can be applied to computer vision projects to increase their reach and usability.
