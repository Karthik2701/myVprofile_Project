## Prerequisites
- JDK 1.8 or later
- Maven 3 or later
- MySQL 5.6 or later
######
## Technologies 
- Spring MVC
- Spring Security
- Spring Data JPA
- Maven
- JSP
- MySQL
## Database
Here,we used Mysql DB 
MSQL DB Installation Steps for Linux ubuntu 14.04:
- $ sudo apt-get update
- $ sudo apt-get install mysql-server

Then look for the file :
- /src/main/resources/accountsdb
- accountsdb.sql file is a mysql dump file.we have to import this dump to mysql db server
- > mysql -u <user_name> -p accounts < accountsdb.sql

//KOPS SETUP
1. EC2 20.0 ubuntu
2. S3 Bucket to store state 
3. Configure AWS CLI 
# Create Cluster
    eksctl create cluster --name=democluster \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup 

# Get List of clusters
     eksctl get clusters       

# Create & Associate IAM OIDC Provider for our EKS Cluster

    eksctl utils associate-iam-oidc-provider \
      --region us-east-1 \
      --cluster democluster \
      --approve

 # Create Public Node Group   
 
      eksctl create nodegroup --cluster=democluster \
                        --region=us-east-1 \
                        --name=demo-cluster-public1 \
                        --node-type=t3.medium \
                        --nodes=1 \
                        --nodes-min=1 \
                        --nodes-max=2 \
                        --node-volume-size=20 \
                        --ssh-access \
                        --ssh-public-key=jumpbox \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access    

# Delete nodeGroup

   eksctl delete nodegroup \
     --cluster eksdemo1 \
     --region us-east-1 \
     --name eksdemo1-public1


# myVprofile_Project

   ![image](https://user-images.githubusercontent.com/35370115/180217060-bd8af0cc-9121-4e8f-8bfb-016c995b4bce.png)

