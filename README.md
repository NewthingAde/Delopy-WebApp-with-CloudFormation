# Delopy-WebApp-with-CloudFormation
Deploying a WebApp using CloudFormation

Infrastructure as Code (IaC) is the managing and provisioning of infrastructure through code instead of through manual processes. With IaC, configuration files are created that contain your infrastructure specifications, which makes it easier to edit and distribute configurations.

In this tutorial I will do a walk through on creating cloudformation template. Let get started. 

## 1. Create an AWS account
## 2. Create an IAM user account select the programmatic option
## 3. Creating a key pair
This is important when we want to ssh into our instance
## Configure the IAM user with our working environment
On the terminal use the following command to configure the environment and input your access key

                    aws configure

## 4. Create Amazon EC2 Instance
Here we will be using this code to create an instance type of our choice. in this example we will create t2.micro.
NOTE: The ImageID valid only in us-east-1 region. You can use the ImageId locator https://cloud-images.ubuntu.com/locator/ec2/
      Also, take note of the identation, it very important

                  AWSTemplateFormatVersion: 2010-09-09
                  Description: Build a webapp stack using CloudFormation
                  Resources:
                    WebAppInstance:
                      Type: AWS::EC2::Instance
                      Properties:
                        ImageId: ami-0d5eff06f840b45e9 
                        InstanceType: t2.micro

Next, wecreate the stack using this template, run the `create-stack` command-line:

              aws cloudformation create-stack --stack-name first-stack --template-body file://one.yaml  
              
## 5. Enable SSH AND HTTP/HTTPS Traffic into our Instance
Here, we want to add a security group resource that allows traffic in on port 22 for SSH and ports 80 and 443 for HTTP and HTTPS traffic. We will edit our cloud formation template and it will include the following changes 

                    AWSTemplateFormatVersion: 2010-09-09
                    Description: Build a webapp stack using CloudFormation
                    Resources:
                      WebAppInstance:
                        Type: AWS::EC2::Instance
                        Properties:
                          ImageId: ami-0d5eff06f840b45e9 
                          InstanceType: t2.micro
                          KeyName: Ade # <-- This is the keypair we created in step 3 above
                          SecurityGroupIds:
                            - !Ref WebAppSecurityGroup

                      WebAppSecurityGroup:
                        Type: AWS::EC2::SecurityGroup
                        Properties:
                          GroupName: !Join ["-", [webapp-security-group, dev]]
                          GroupDescription: "Allow HTTP/HTTPS and SSH inbound and outbound traffic"
                          SecurityGroupIngress:
                            - IpProtocol: tcp
                              FromPort: 80
                              ToPort: 80
                              CidrIp: 0.0.0.0/0
                            - IpProtocol: tcp
                              FromPort: 443
                              ToPort: 443
                              CidrIp: 0.0.0.0/0
                            - IpProtocol: tcp
                              FromPort: 22
                              ToPort: 22
                              CidrIp: 0.0.0.0/0

Next, we update the stack using the `update-stack` command:

                     aws cloudformation update-stack --stack-name first-stack --template-body file://02_ec2.yaml

Next, we will ssh into the new instance that we cretaed to see if it works

  1. cd into the directory where the keypair we downloaded above and run the following command 
  Note: Ade.pem is the name of my keypair i created, so you have to use yours
  
                              chmod 400 Ade.pem
                              
                              ssh -i "Ade.pem" ec2-user@ec2-54-90-99-154.compute-1.amazonaws.com

## 6. Create Elastic IP Address And OUTPUT the Website URL
We edit the template again and include the elastic ip and output 

                        AWSTemplateFormatVersion: 2010-09-09
                        Description: Build a webapp stack using CloudFormation
                        Resources:
                          WebAppInstance:
                            Type: AWS::EC2::Instance
                            Properties:
                              ImageId: ami-0d5eff06f840b45e9 
                              InstanceType: t2.micro
                              KeyName: Ade # <-- This is the keypair we created in step 3 above
                              SecurityGroupIds:
                                - !Ref WebAppSecurityGroup

                          WebAppSecurityGroup:
                            Type: AWS::EC2::SecurityGroup
                            Properties:
                              GroupName: !Join ["-", [webapp-security-group, dev]]
                              GroupDescription: "Allow HTTP/HTTPS and SSH inbound and outbound traffic"
                              SecurityGroupIngress:
                                - IpProtocol: tcp
                                  FromPort: 80
                                  ToPort: 80
                                  CidrIp: 0.0.0.0/0
                                - IpProtocol: tcp
                                  FromPort: 443
                                  ToPort: 443
                                  CidrIp: 0.0.0.0/0
                                - IpProtocol: tcp
                                  FromPort: 22
                                  ToPort: 22
                                  CidrIp: 0.0.0.0/0

                        # This is assigning an elastic IP address to our Instance
                          WebAppEIP:
                            Type: AWS::EC2::EIP
                            Properties:
                              Domain: vpc
                              InstanceId: !Ref WebAppInstance
                              Tags:
                                - Key: Name
                                  Value: !Join ["-", [webapp-eip, dev]]
                        Outputs:
                          WebsiteURL:
                            Value: !Sub http://${WebAppEIP}
                            Description: WebApp URL

Next, we update the stack using the `update-stack` command:

                     aws cloudformation update-stack --stack-name first-stack --template-body file://02_ec2.yaml


## 7. Delete the Stack

Next we delete the stack so as not to incur charges. You can do that with the `delete-stack` command:

                     aws cloudformation delete-stack --stack-name first-stack


