# Packer and AWS Inspector Integration using Jenkins to create AMI

## Overview
The idea behind this solution is to create the Jenkins Pipeline to create AWS AMI using Packer and also run the AWS Inspector to find any vulnerabilities. The newly created AMI image names will be added/updated to System Manager Parameter Store for future reference. Currently using this pipeline you are able to create the plain Ubuntu, Java 1.8, ActiveMQ and ElasticSearch images. You can keep adding more software installation packages and update the configuration accordingly.

## Installation and Usage instructions

**Important Note: **
Please download jdk-8u251-linux-x64.tar.gz file and add into current repository root folder or udpate the download location path in the script before running the script using Jenkins

1. Create AWS Assessment target and assessment template for tag as "Name=Packer Builder". Please note that this is the default name of Packer Build EC2 Instance which get created during image creation process.
![jenkinsconfig](/images/image8.png)

2. Create Jenkins Pipeline Job by pointing to Jenkinsfile groovy file. Please make sure *aws_access_key* and *aws_secret_key* are configured into Jenkins credentails section.

3. Execute Jenkins job with "Build with Parameter" option
   ![jenkinsconfig](/images/image1.png)
   You need to execute this job by selecting above build parameter : 
   * **os_name :** Select which OS you want to target to create AMI. Currently only two OS are supported i.e. Ubuntu and Amazon Linux
   * **install_package :** Select software package which you want to install on AMI. This is only for Ubuntu OS currently
   * **aws_source_ami :** Select which base image you want to use to create new AMI. There are two currently i.e. for Ubuntu and Amazon Linux

4. Once Jenkins job started executing, it will display following steps 
   Jenkins Declarive Job View
   ![jenkinsconfig](/images/imag2.png)
   
   * **Packer Validate :** Validate the packer project to identify any potential errors 
   * **Packer Build Approval :** Manual approval in the pipeline. You have to approve it manually by reviewing the Packer Validate step output
   * **Packer Build & Execute Inspector :** This step create the temporary EC2 instance, install the the required software packages, additionally installed the Inspector agent and run the python script run the assessment. This step take almost 30 minutes to get the result out. JUnit result will be generated by Python script.
   * **Update Parameter Store for AMI :** This steps update the System Manager - Parameter store with updated AMI IDs for future use
   * **Generated Inspected Report :** This steps use the JUnit result script generated by packer build and display into Jenkins Unit Test Plugin
   
 5. Once Jenkins job is executed successfully, you will see above JUnit Test Result screen where all the Vulnerabilities will be displayed after scanning them from Inspector. The Python script execute the inspector assessment and generate the Vulnerability report.
 ![jenkinsconfig](/images/image3.png)
 
 6. If you want to see detailed Vulnerability details, click on respective test result and it shows further details of respective Vulnerability found on EC2 instnace
 ![jenkinsconfig](/images/image4.png)
 
 7. You can also find the same Vulnerability report under AWS Inspector run section
 ![jenkinsconfig](/images/image5.png)
 
 8. At the end of the execution, the newly created AMI ID get added/updated to System Manger Parameter store section. You can always take the latest AMI ID from below location whenever you want to create EC2 instance from golden AMIs
 ![jenkinsconfig](/images/image6.png)
 
