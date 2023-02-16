# AWS SAP-CO2 Study Guide
The following guide is my attempt at helping myself and possibly others to pass the AWS Certified Solutions Architect Professional Exam.  I'd recommend taking Stephane Maarek's course [Ultimate AWS Certified Solutions Architect Professional 2023](https://www.udemy.com/course/aws-solutions-architect-professional/) and the a few practice exams from [AWS Certified Solutions Architect Professional Practice Exam](https://www.udemy.com/course/aws-certified-solutions-architect-professional-aws-practice-exams/) before taking the exam.  

Note: The author makes no promises or guarantees on this guide as this is as stated, a guide used by myself, nothing more.  This is a work in progress.  

## Table of Contents
1. <a href="#introduction">Introduction</a>
2. <a href="#identity-and-access-management-iam">Identity and Access Management (IAM)</a>
3. <a href="#ec2">EC2</a>
4. <a href="#containers">Containers</a>
5. <a href="#cloudwatch">CloudWatch</a>
6. <a href="#database">Database</a>
7. <a href="#vpc">VPC</a>
8. <a href="#acronyms">Acronyms</a>

## Introduction
<a href="https://d1.awsstatic.com/training-and-certification/docs-sa-pro/AWS-Certified-Solutions-Architect-Professional_Exam-Guide.pdf">AWS Certified Solutions Architect - Professional (SAP-C02) Exam Guide</a>
### Exam Content Breakdown:

| Domain  | % of Exam |
| ------------- | ------------- |
| Domain 1: Design Solutions for Organizational Complexity  | 26%  |
| Domain 2: Design for New Solutions  | 29%  |
| Domain 3: Continuous Improvement for Existing Solutions | 25% |
| Domain 4: Accelerate Workload Migration and Modernization | 20% |
| **Total** | **100%** |

## Identity and Access Management (IAM)

### Identity and Access Management Policy Evaulation Logic:
![IAM Policy Evaulation Logic](https://docs.aws.amazon.com/images/IAM/latest/UserGuide/images/PolicyEvaluationHorizontal111621.png)

Allow vs Deny: If any denial in policy is present, the resource is denied regardless of allow statement(s).  The default behavior is to deny resource(s) and resource(s) need allow statements to be allowed.  

LDAP: software protocol for enabling the location of data about organizations, individuals and other resources in a network.  

Identity federation: a system of trust between two parties for the purpose of authenticating users and conveying information needed to authorize their access to resources.

###  Identity and Access Management Access Analyzer:

## EC2

## Containers

## CloudWatch

## Database

## VPC

## Acronyms

| Acronym  | Definition |
| ------------- | ------------- |
| ADFS | Active Directory Federation Services |
| ARN | Amazon Resource Name |
| IAM |  Identity and Access Management |
| IdP | Identity Provider |
| LDAP | Lightweight Directory Access Protocol |
| OU | Organizational Unit |
| SAML | Security Assertion Markup Language |
| STS | Security Token Service |
| SCP | Service Control Policies  |
