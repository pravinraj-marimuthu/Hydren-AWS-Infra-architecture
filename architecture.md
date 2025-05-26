# Hydren's Infrastructure Architecture on AWS

Here's a quick overview on the AWS service we are going to use.

1. VPC - Isolates our resources inside AWS
2. CloudFront - Distributes our web contents fastly
3. Route 53 - Resolves DNS to our resources
4. WAF - Prevents the site from security threats
5. ALB - Distributes user traffic evenly to the servers
6. RDS PostgreSQL - Database instance with automated backups enabled to persist application data
7. Docker - Containerize the frontend (React) and backend (Flask) as Docker images
8. DockerHub - Stores the images inside a private repository
9. ECS Fargate - Manages the containers using Fargate as a compute resource, enabling auto scaling to improve performance based on the traffic and performance
10. Cloud Watch Logs - Application, Database logs for issue detection and resolution
11. Dashboard/Alarms - Alerts on threshold breach such as high error rate, CPU and Memory spikes, etc..
12. VPC Flow Logs - to analyse traffic.

## Application architecture:

Before understanding the architecture, let's quicky walk-through the terminologies we will be using.

Containers:
  It is a light-weight executable which will package all the dependencies into a single bundle. Easy to deploy and scale. 

Why Containers are better than traditional virtualization?
  - In Virtulization, we can run only one application per server which will lead to wastage of resources.
  - Containers does not need an entire server to run the application. It is isolated from the kernel level, using resources virtually, providing us the ability to run multiple application in a single resource.
  - For example, let's say we are using a t2.micro EC2 instance to deploy an application, we need to install nginx, deploy the code in the server and run the application. If you look at this, this instance eventually will have lot of resources being wasted and it would lead to unnecessary cost.
  - When it comes to Docker which is a containerization platform, we install Docker as a tool in the instance, we will be able to run around 5-10 small application making the resource usable in an efficient ways.
Our desired architecture will be as follows:

- We will have two components:
    - Frontend - React 
    - Backend - Flask
 
- Both these services will be built as a seperate images and pushed to private repositories.
- We will then be deploying those images in ECS fargate (a serverless computing to run containers)
- Fargate will generate cost only for the resource we use. We can integrate Auto scaling with backend and frontend to increase the performance based the traffic (concurrent users, bulk uploads)
- In our case,
    - For frontend (React), we will only need a very little resource **0.25-1 vCPU** and **1 GB memory**.
    - For backend (Flask), we can set a moderate value **0.5 to 2 vCPU and 2 GB memory**.
    - These are sample performance values considering a small scale applications. After testing, we can decide and set a default value for both frontend and backend.
    - The good thing with Fargate is, we can enable auto-scaling meaning we can actually increase the infrastructure compute based on the load it receives
    - An example is as follows:
        - our frontend does not need auto scaling as it only serves static contents
        - our backed will only do all the heavy works such as Bulk upload. We can set a thresold value such as if vCPU or memory becomes >60%, we can automatically add (scale up) one more service to manage the load and it can keep increasing. Likewise, when the load drops, we can automatically reduce (scale down) the service count.
        - The good thing here is we don't have to worry about managing the server management because AWS will take care of this for us. We only pay for the resources we use.
        - In our case, only our backend will have heavy load and we have enabled auto scaling to manage the changing demand.
        - So, When the load is high, the resources will scale up automatically and so will the cost.
        - When the load drops, the resouces will scale down and so will the cost.
     
## Conclusion

Using EC2, atleast one server should be always be running, it leads to 
  - Wastage of resources during less traffic
  - High cost consumption as atleast has to be running to run the application 24x7
Using ECS Fargate, we develop tasks (container), it will help us in many ways as below:
  - Only 1 task of frontend and 1 backend will run which will use very less resource during idle time (no bulk upload, less users).
  - If load increases, only then resource usage will increase automatically leading to high cost consumption.
  - Using this setup, we can surely optimize our infrastructure to achieve highly available application, reducing the cost drastically.

Thank you,
Pravinraj Marimuthu
