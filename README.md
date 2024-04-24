# DOCKER IMAGE

## Description
This is a simple repo that contains all of my docker images that I have created for pipelines that need specific runtimes.
Read the README.md file in each folder to know more about the docker image.

## Docker Images
- ### [Docker Image 1](docker-java) (Java) (Deprecated, prefer using docker-maven instead)
  - _**Description**_: This is a docker image that contains the java runtime environment installed with terraform, helm and the aws cli.
- ### [Docker Image 2](docker-maven) (Java)
  - **_Description_**: This is a docker image superior to the first one, it contains the java runtime environment installed with terraform, helm, the aws cli and is pulled from maven images.
- ### [Docker Image 3](docker-nodeJS) (NodeJS)
  - **_Description_**: This is a docker image that contains the node runtime environment installed with terraform, helm, kubectl and the aws cli.

## How to use
- ### Docker Image 1 (Deprecated, prefer using maven-terraform instead)
  - **_Pull the image_**: `docker pull shad0wz7/java-aws-terraform:tagname`
  - **_Run the image_**: `Use it in your gitlab-ci.yml file or any other pipeline file`
- ### Docker Image 2
  - **_Pull the image_**: `docker pull shad0wz7/maven-terraform:tagname`
  - **_Run the image_**: `Use it in your gitlab-ci.yml file or any other pipeline file`
- ### Docker Image 3
  - **_Pull the image_**: `docker pull shad0wz7/node-aws-terraform:node-20.10`
  - **_Run the image_**: `Use it in your gitlab-ci.yml file or any other pipeline file`

### Note
If you like this repo, please don't forget to give it a star. Thank you!
If you have any questions or need help, please open an issue.
If you have any suggestions, please reach out to me on [LinkedIn](https://www.linkedin.com/in/emmanuel-boisset-b90970195/).

My gitlab: https://gitlab.com/ShadowZ11  
My github: https://github.com/ShadowZ11