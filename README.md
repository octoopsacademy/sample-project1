# sample-project1
Steps involved in deploying the app to the EKS cluster:

1. Pre requisites AWS Infra( Can be created manually or can be created using IaC scripts using Terraform):
       1. Create an EC2 instance to Host Jenkins Server ( attache a IAM role with relevant permissions)
       2. Create a EKS cluster
       3. Create A RDS Database
2. Installation and configurations of Jenkins are provided in the Readme.md file inside jenkins directory

3. Application Repo Tree:
## Repo tree

```
octoopsacademy/
├─ frontend/
│  └─ index.html
├─ backend/
│  ├─ package.json
│  └─ server.js
├─ jenkins/
│  ├─ Dockerfile
│  └─ README.md
├─ Dockerfile
├─ Jenkinsfile
├─ k8s/
│  ├─ deployment.yaml
│  └─ service.yaml
├─ sql/
│  └─ init.sql
└─ README.md

4. Create an mysql rds in aws and setup admin credentials which can be used as DB_PASS and DB_USER
5. Install any mysql client and connect to your rds db and create a database and table using below commands

Create Database & Table (Only Once)

Run this SQL in RDS (via MySQL Workbench / AWS Query Editor / CLI):

CREATE DATABASE IF NOT EXISTS octoopsdb;
USE octoopsdb;

CREATE TABLE IF NOT EXISTS users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

5. customise the k8s/deployment.yaml with the necessary DB credentials and ECR repo url
6. Then Run the jeniks pipline or use webhook setup to auto trigger the pipline
   
