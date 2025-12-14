**S3 Static hosting: S3 Static Hosting using Terraform tutorial**(https://youtu.be/bK6RimAv2nQ?si=riG29ZYMsUqgHJjD)

**Pre-requisite:**
⦁	You are a DevOps Engineer with only QA env access, such as GitHub-TEST-ENV-Repo, Terraform Enterprise TEST workspace, AWS-TEST-Account with limited IAM permissions, and Jenkins-TEST-ENV page.
 	NOTE: Every application on GitHub, Terraform Enterprise, Jenkins, AppDynamics, ArgoCD, SonarQube, Trivy, DockerHub, and AWS Account is secured with AWS 	Cognito.
⦁	AWS Cognito is used to log in to GitHub, Terraform Enterprise(SAML), & Jenkins
⦁	How to bind Infra GitHub TEST Repo, AWS TEST Account to Terraform Enterprise, which also manages state files, modules, & does sentinel policy checks and applies?
⦁	How to create a module for S3 hosting Infra and store it in GitHub Repo as source?
⦁	How to bind Developer's GitHub TEST Repo, Jenkins-TEST-ENV, AWS TEST Account?

**Deployment Flow:**

0. Whenever the DevOps Engineer updates the modules with latest security best practices and cost optimization enhancements, Modules are verified by platform  team before creating the service catalogues and SCPs at respective AWS Organizations level accounts. Modules are created with assistance of Security team with respect to latest security customizations, OAC & ACL rules, appending malicious IPs, bot control.

1.	Deploy the S3 Static hosting Infra using Terraform Enterprise.
 	-> Clone the GitHub Test Repo, use TFE modules to write the code, push the code to GitHub Test Repo, TFE enterprise triggers the infra after 	 	   successful sentinel run in AWS TEST Account.

 	Note: State Locking Link, But in case of TFE Enterprise version it stores the Terraform state in its own managed backend. You do not need (and 	should not have) a local or S3 backend block in your root module for those runs — TFE manages state, locking, versions, and encryption for you.

2. S3 Static Hosting Infrastructure is ready and is in working condition.

3. Developer with the access to GitHub Repo creates a PR to merge dev to test branch(This Repo contains the Jenkinsfile too).-- GitLab Flow Branching strategy is used.


**GitLab Flow Branching Strategy:**
Step-by-step:
1.	You take a task from GitLab Issue Tracker
2.	Create a branch named after the issue:
 		-> issue-143-add-otp-service
3.	Develop locally
4.	Open Merge Request
5.	Pipelines run:
 	o	Unit tests
 	o	Code quality (SonarQube)
 	o	SAST
6.	If everything passes → merge into dev
7.	GitLab CI automatically deploys to Dev environment
8.	QA tests → merge into test
9.	After approval → merge into main
10.	Production deployment happens



4. PR triggers the Jenkins TEST Env CI pipeline that checks-out the code from GitHub TEST branch, builds it and uploads the files on to AWS TEST Env S3 bucket and successfully hosting the static react web app.

5. After all the functional testing, OAT testing in TEST ENV. The code is merged to the GitHub PROD branch and deployment is taken care by Senior DevOps Engineer in Production Env.

**Optional but important resources to integrate the service with:**
1.	AWS Config to assess, audit & evaluate the configurations of AWS resources
2.	AWS Trusted Advisor that identifies opportunities that can save money for users by providing recommendations on cost optimization and detect underutilized resources.
3.	AWS Shield - Protection against DDoS
4.	AWS GuardDuty - Intelligent threat protection & continuous monitoring across all AWS accounts
5.	AWS X-Ray - View end to end performance metrics & troubleshoot distributed applications, aiding in identifying performance bottlenecks.
6.	AWS Security Hub - Aggregates findings from tools like GuradDuty, Inspector, etc into a single dashboard.

**IAC & CI Design:**

1.	Modules for S3 hosting with below AWS resources:
-> S3 bucket with versioning, custom S3 bucket policies to allow access to cloudfront & GetObject to CloudFront ARN, enable static hosting with HTTPS, Block public access & apply tags. (Optional): S3 Event Notification,S3 Encryption using AWS KMS/Customer provided keys(SSE-C), S3 Access Logs.

-> IAM roles for intercommunication of AWS resources.

-> Route 53 with record name A, AAAA, CNAME, Failover routing policy, routing traffic to cloudfront & tags.

-> CloudFront with S3 website endpoint as Origin Domain, enable realtime logs, takes in AWS WAF, Alternative Domain Name(CNAME) = domain name, SSL certificate & tags. (Optional): CloudFront Origin Access Control(OAC), CloudFront Invalidation.

-> AWS ACM with public certificate, domain name, Validation method, append CNAME record output from ACM to Records in Route 53 & tags.

-> AWS WAF: Layer 7(HTTPS) protection with WAF ACL(Access control Lists) & tags.(Optional): AWS Shield Advanced for DDoS attack.

2.	Jenkinsfile which checkout the code from repo branch, builds the source code, and upload the files in S3 (access to AWS S3 Put object), curl the URL and get the exit status code.


**AWS Hosting Theory(Route53+ CloudFront+ AWS WAF + S3-Private):

Non-CloudFront hosting (Basic Hosting):**

⦁	Upload the react file to the S3 bucket
⦁	Mention the permission of get objects & custom URL in the bucket policy section of the S3. 
⦁	To Access the URL just go to index.html and click on Object URL (might not work)
⦁	Enable the static website hosting option.
⦁	Domain name is registered with Instra for Westpac.
⦁	Remember which routing policy to use for Westpac(Failover).
⦁	In Route 53, record name with type A and recommended routing policy should Route traffic to S3 website endpoint.
⦁	Open the URL and it works!!

**With CloudFront(Secure Hosting):
**
⦁	CloudFront enables secure access of page in HTTPS and uses caching for latency.

⦁	Before that we have to create a certificate from AWS ACM, with type request a public certificate > Add domain name > Select validation method (DNS Validation) > Create the tag names > Add the CNAME record obtained to Route 53 for validation.

⦁	Open Cloudfront > Create Distribution > input origin Domain name value as S3 static bucket website endpoint > Enables realtime logs > Select the AWS WAF > Input the original DNS name as your Alternative Domain Name(CNAME) > Select Custom SSL certificate > Enable Distribution State > Create Distribution.

⦁	Make sure you have static website hosting enabled to HTTPS & Block all Public Access in S3 and edit the Route 53 routing policy towards Cloudfront instead of S3.

⦁	Also to have a good latency of the webpage with caching, you need to enable cloudfront invalidation: Open Cloudfront > Distribution Settings > Invalidations > Create Invalidation for all S3 path.
