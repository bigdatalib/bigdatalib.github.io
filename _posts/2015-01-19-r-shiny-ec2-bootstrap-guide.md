---
layout: post
author: Chris Zhou
title:  R Shiny EC2 Bootstrap Guide
summary: Quickly spinning up a R shiny server on EC2
date:   2015-01-19 19:37:00
category: programming
tags: [github, r, shiny, aws, ec2]
---
I wrote a guide on [Github](https://github.com/chrisrzhou/RShiny-EC2Bootstrap) to quickly bootstrap an AWS EC2 instance with R Shiny.  
Posting on Jekyll (second post) seems to be as simple as copying the `README.md` file from Github over.  Pretty damn awesome!  Here's the guide!

------
# R Shiny EC2 Bootstrap
The goal of this guide is to quickly bootstrap R <a href="http://shiny.rstudio.com/" target="_blank">Shiny</a> on an <a href="http://aws.amazon.com/ec2/" target="_blank">Amazon AWS EC2</a> instance.  We will also walk through a recommended workflow using Github for rapid development on a local machine and the EC2 server instance.

![alt Amazon Machine Image](https://s3-us-west-1.amazonaws.com/chrisrzhou/github/images/rshiny-ec2bootstrap/aws_ec2_workflow.png)
    
This is an *opiniated* guide created for the following software versions:
- `Ubuntu: 14.04 LTS`
- `R: 3.1.2`
- `Shiny Server: 1.0.0`

*For other versions, please refer to the other github versions of the project repository.*

----------

## Contents
* [Launch EC2](#launch-ec2)
* [Register Key Pair](#register-key-pair)
* [Install R](#install-r)
* [Install R Packages](#install-r-packages)
* [Install Shiny Server](#install-shiny-server)
* [Install Git](#install-git)
* [Workflow Overview](#workflow-overview)
* [Problematic Packages](#problematic-packages)
* [Examples](#examples)
* [Resources](#resources)

----------

## Launch EC2
- Create an <a href="https://aws.amazon.com/" target="_blank">AWS account</a> and login to the <a href="https://console.aws.amazon.com" target="_blank">console</a>.
- Click on `Compute > EC2` (near the top left of the screen) and click the `Launch Instance` button in the `Create Instance` section of the page.
- We are now in the AMI (Amazon Machine Image) interface.  Select `Ubuntu Server 14.04 LTS`.

    ![alt Amazon Machine Image](https://s3-us-west-1.amazonaws.com/chrisrzhou/github/images/rshiny-ec2bootstrap/aws_ec2_ubuntu14.04.png)
    
- We will choose the default `t2.micro` instance.  Accept all the default settings for the next few steps:
    - Click `Next: Configure Instance Details`
    - Click `Next: Add Storage`
    - Click `Next: Tag Instance`
    - Click `Next Configure Security Group`
- When we arrive to `Configure Security Group`, we will need to add a few more security groups to open our EC2 instance to the world when we later build applications with Shiny.
- Add the following rules using the `Add Rule` buttons.  This registers and opens the ports `3838` and `80` that Shiny will use later.

    ![alt Amazon Machine Image](https://s3-us-west-1.amazonaws.com/chrisrzhou/github/images/rshiny-ec2bootstrap/aws_ec2_configure_security_groups_complete.png)
    
- We are ready to launch our EC2 instance now.
- Click on the `Review and Launch` button, and `Launch` your instance!

    ![alt Amazon Machine Image](https://s3-us-west-1.amazonaws.com/chrisrzhou/github/images/rshiny-ec2bootstrap/aws_ec2_launch.png)

[(back to contents)](#contents)

----------

## Register Key Pair
- With our EC2 instance set up, we need to register the key provided to us from Amazon with our local machine. (*Note that only one EC2 key can be registered to one machine.*)
- Amazon will prompty you to create a new key pair.  For this guide, we shall name the keypair `shinybootstrap`.

    ![alt Amazon Machine Image](https://s3-us-west-1.amazonaws.com/chrisrzhou/github/images/rshiny-ec2bootstrap/aws_ec2_create_keypair.png)
    
- Download the `shinybootstrap.pem` key pair.  We recommend that you place your `shinybootstrap.pem` key pair in a safe folder, but for purposes of this guide, we will assume that you are placing the file in a folder under the home directory `~/sshKeys`.
    
    ```sh
    # create sshKeys folder in home directory
    mkdir ~/sshKeys
    
    # navigate to your .pem directory and move the .pem file to the sshKeys folder
    mv shinybootstrap.pem ~/sshKeys
    
    # grant sudo access to .pem file
    chmod 400 ~/sshKeys/shinybootstrap.pem
    ```
    
- **Note**: If you ever run into a `Permission denied (publickey)` error message, this is because you are not able to access the `.pem` file as a superuser.  Make sure that you have granted sudo access to the file using the command `chmod 400` as outlined above.
- It is useful to create an alias to access the EC2 instance, otherwise we would be typing a long command to access our EC2 instance each time.  For this guide, we will create an alias `awslogin` and register this in the `bash_aliases` file using the `echo "blahblahablah" >> ~/.bash_aliases` command:
   
    ```sh
    echo "alias awslogin='ssh -i ~/sshKeys/shinybootstrap.pem ubuntu@public_dns_name'" >> ~/.bash_aliases
    ```
    
- where `public_dns_name` is the name of your EC2 instance created e.g. `ec2-54-183-2-10.us-west-1.compute.amazonaws.com` (see below).

    ![alt Amazon Machine Image](https://s3-us-west-1.amazonaws.com/chrisrzhou/github/images/rshiny-ec2bootstrap/aws_ec2_dns_name.png)
    
- Restart your terminal to access the changes and you should connect to your EC2 instance simply by typing `awslogin` in the terminal console.

[(back to contents)](#contents)

----------

## Install R
- Phew! The tough parts are over, we can now move onto installing `R` in our EC2 instance!
- First, create a superuser root password by entering a password in the EC2 command line.
    
    ```sh
    sudo password root
    su
    ```
    
- Update your barebones EC2 virtual machine and install R:
    
    ```sh
    sudo apt-get update
    sudo apt-get install r-base-dev
    sudo apt-get install libcurl4-openssl-dev
    ```
    
- R is now successfuly installed!

[(back to contents)](#contents)

----------

## Install R Packages
- Base R is argubly boring, so let's get started with installing a few important packages, `shiny` being an obviously shiny option.  (*while installing these packages, make sure you are in `su` mode and I highly recommend installing them from the `0-Cloud` CRAN repository*)
- From your EC2 command line:
    
    ```sh
    R       # enter the R console
    
    install.packages("shiny")
    install.packages("ggplot2")
    q()         # quit R
    ```
    
- A few other packages are highly recommended but you might encounter issues installing certain packages due to memory limitations available on the EC2 instance or missing libraries in our EC2 instance.  For more details, please refer to the **[Problematic Packages](#problematic-packages)** section for a brief outline.
- Other than that, you can come back anytime in the future to install R-packages by logging in with `su` and installing R packages via the command line.

[(back to contents)](#contents)

----------

## Install Shiny Server
- `shiny` requires a server to run.  The next step is to setup `shiny-server` in our EC2 instance.
- Run the following commands in EC2:
    
    ```sh
    sudo apt-get install gdebi-core
    wget http://download3.rstudio.org/ubuntu-12.04/x86_64/shiny-server-1.0.0.42-amd64.deb
    sudo gdebi shiny-server-1.0.0.42-amd64.deb
    
    ```
- And we're done!


[(back to contents)](#contents)

----------

## Install Git
- `git` is probably one of the greatest creations and tools that you can use in your life, so it's good to include it in our EC2 instance.
- Installing `git` is as simple as typing:
   
    ```sh
    sudo apt-get install git
    ```
    
- `git` is very critical in our Shiny EC2 workflow.  If you are not familiar with using `git`, here are some great resources to get on speed:
    - <a href="https://www.codeschool.com/courses/try-git" target="_blank">Codeschool</a>
    - <a href="http://git-scm.com/" target="_blank">Official Git site</a>

[(back to contents)](#contents)

----------

## Workflow Overview
- A big part of the EC2 R Shiny workflow is to develop with RStudio on your local machines and use Github as a repository station for transferring data between your local machine and EC2 instance.  Make sure that you are familiar with the basic `git` commands: `pull`, `push`, `clone`.  It's going to be a big part of the development cycle.
- The following diagram outlines the overall workflow of the EC2 R Shiny workflow:
    
    ![alt Amazon Machine Image](https://s3-us-west-1.amazonaws.com/chrisrzhou/github/images/rshiny-ec2bootstrap/aws_ec2_workflow.png)

- Workflow Summary:
    1. Develop on your local machine with RStudio.
    2. Whenever you are ready to show your work to the world, you would `git push` your changes to your Github repository.
    3. Initiate your EC2 instance with the same codebase on Github by using `git clone` into `/srv/shiny-server/` (this is the folder where `shiny-server` serves your application).
    4. If there are changes to be made to your app, repeat steps 1 and 2 to `git push` updates to Github from your local machine, and then perform a `git pull` in your EC2 instance to update your application in EC2.

[(back to contents)](#contents)

----------

## Problematic Packages
- Since our EC2 instance is completely barebones, installing various packages might be problematic due to a lack of libraries that EC2 comes with.
- My recommendation is to do searches on <a href="http://stackoverflow.com/" target="_blank">Stackoverflow</a> and <a href="http://www.google.com" target="_blank">Google</a> using keywords such as `EC2`, `package-name`, `installation`, `R`.
- You could run into issues with missing `Python` libraries and version requirements for `R` and various packages, but most of them can be solved by searching on Stackoverflow and Google.
- You could also encounter issues that result from the limited memory allocated in the EC2 instance, which would not complete installations of some packages (e.g. `dplyr`, `ggvis`).  If this is the case, the solution would be to install these packages on your local machine, and grab the uncompiled files in the local machine `R` libraries and copy them over to your EC2 `R` libraries.  You have a few ways to transfer files from local machine to EC2 and I will refer you to Stackoverflow and Google on how to do so.
- Stackoverflow + Goole are most definitely the best teachers and guides out there for resolving problematic packages.  My job here is to just inform on some of the issues you may encounter :)

[(back to contents)](#contents)

----------

## Examples
Here are some project examples of Shiny EC2 instances that I have built and their respective Github codebase.
- State Crime Rates (<a href="http://ec2-54-183-164-175.us-west-1.compute.amazonaws.com:3838/StateCrimeRates/" target="_blank">EC2</a> | <a href="https://github.com/chrisrzhou/RShiny-StateCrimeRates" target="_blank">Github</a>)
- Labor Force Statistics (<a href="http://ec2-54-183-164-175.us-west-1.compute.amazonaws.com:3838/LaborForceStatistics/" target="_blank">EC2</a> | <a href="https://github.com/chrisrzhou/RShiny-LaborForceStatistics" target="_blank">Github</a>)
- Box Office Mojo (<a href="http://ec2-54-183-164-175.us-west-1.compute.amazonaws.com:3838/BoxOfficeMojo/" target="_blank">EC2</a> | <a href="https://github.com/chrisrzhou/RShiny-BoxOfficeMojo" target="_blank">Github</a>)
- Power to Choose (<a href="http://ec2-54-183-164-175.us-west-1.compute.amazonaws.com:3838/PowerToChoose/" target="_blank">EC2</a>) | <a href="https://github.com/chrisrzhou/RShiny-PowerToChoose" target="_blank">Github</a>)

[(back to contents)](#contents)

----------

## Resources
This guide would not be possible without the works and guides provided by previous people.  I'm listing some additional resources that are helpful for the general audience.  Please contact me if there are any issues/corrections with this guide.  Thank you!

- Shiny AWS EC2 Guides:
    - <a href="http://tylerhunt.co/2014/03/amazon-web-services-rstudio/" target="_blank">Tyler Hunt's guide to set up Shiny AWS EC2</a>
    - <a href="http://www.r-bloggers.com/deploying-shiny-server-on-amazon-some-troubleshoots-and-solutions/" target="_blank">Custom AWS inbound rules</a>
- Other resources:
    - <a href="http://stackoverflow.com/" target="_blank">Stackoverflow</a>
    - <a href="http://cran.rstudio.com/web/packages/dplyr/vignettes/introduction.html" target="_blank">dplyr</a>

[(back to contents)](#contents)

----------
