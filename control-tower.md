# Architecture for Multi-tenant application using AWS Cloud Tower and Cloud Formation

Let's quickly walk-through the basic terminologies involved:

- AWS Control Tower: ability to manage and govern mutliple accounts (a dedicated account for each client) in AWS root account.
- Landing zone: a configuration space to set home region, configure Organization Units and accounts.
- AWS Account: The dedicated account for each client.
- Organization Unit: Logical grouping of accounts to apply policies, govern compliance.
- Shared Account: These are common accounts used by all the accounts (clients) we will be creating in future and managed by the super admin. By default, we will have two accounts:
  - Log archive account (logs of API activities and resource configurations across all accounts (clients))
  - Audit account (manage security and compliances across all accounts (clients))
- Amazon Single Sign On: Enables Super admin to login to any accounts and manage and track resources (added by default for root account)
- AWS Config: A service to track and record all the configurations of resources across all accounts.
- Account Factory: This is used to provision resources such as EC2, VPC in a specific account.
- AWS CloudFormation: Resource configuration templates to easily deploy services across multiple accounts.

I know this is a lot and quite tough to understand but as you go through the step by step process below, we will grasp an idea of how it works.

## Architecture Explanation

Let's assume we are creating an AWS account from scratch, in order to better understand the step by step process:

1. We will use an e-mail ID and set password to register an account in AWS console. This will be our root account (hydren-root).
2. Inside root account, we will use AWS Control Tower Landing Zone service to manage multiple accounts.
3. In order for Super Admin (Hydren team) to access all the accounts (client's isolated account inside root AWS account), we will need a shared account to manage all the client's accounts.
4. For that, we will create a Organization Unit in the name Hydren-Admin. Under that OU, we will create two accounts which will be used by Hydren admins to manage Log archives,  security and governance. 
5. Note that each accounts mentioned will require a valid e-mail account to confirm subscription. We will login to those accounts using that email IDs.
6. After Launching a Landing zone, it will take a maximum of 60 minutes to allocate one OUs and two accounts.
7. Once done, the root account will receive mail within which there will be a link to set up password and a link (a centralized place which will have all the accounts of clients))
8. Once password is set, we will click on that centralized link, which will redirect to page showing list of all the account we have created so far( Log archive account and Audit account)
9. Now we have set up a full fledged Control tower which has the capability to provision create accounts only for each clients who will be onboarded.

Wait, we are not done yet. We have only created accounts Control tower to provision accounts for each clients and configured security and goverance. We still need to configure resources as per our architecture (VPC, Fargate, Load Balancers, etc..) for each clients to run the applications.

1. We don't need to manually provision resources every time when client is onboarded. 
2. We can use AWS CloudFormation or Terraform to configure a template of all the resource we will need, inorder to run the application.
3. That template setup is one time thing which can be re-used during each client onboarding.
4. This template will be linked to a Account Factory. The account factory will privision the resources in the targeted account.

Quick Summary:

1. We will create a root account (Hydren-root).
2. We will create a Organization Unit named (Hydren-admin) and created two accounts under that OU. Account names will be (Logs, Audits).
3. We will setup AWS SSO using root (Hydren-root) account and get the authoization to login into any future accounts (client).
4. We will enable AWS Config in root (Hydren-root) account to track and record configurations across future accounts (client).
5. We will develop a Cloud Formation template that will contain all the resources configuration to run our Hydren application.
6. We will attach that template into Account Factory which is linked to Control tower.
7. Now any client onboarding will be easier. We just have to login to a link (shared via email while setting up password) and create accounts and group accounts into different OUs.
8. The Control tower will manage security and governance. Account Factory with the help of Cloud Formation will deploy infrastructure on each client's account.

CI/CD and Future Considerations:

- We can integrate Jenkins with AWS (free) or use CI/CD services of AWS (Expensive) and deploy the latest changes to each client. We can design a pipeline as requested by Developers. 100% flexibility.
- Using Control Tower can lead to way too much of billing. For bootstrapping and to eliminate the risk of security and governance, we can consider AWS Control Tower and all the approach described above.
- As we stabilize ourselves through this architecture, later some period, we will gain confidence about how everything works, how control tower manages the accounts.
- We can then continously improve the infrastructure provisioning and application deployments and if possible, we can look for ways to save lot of money from billing using the automation, free of open source secure tools.
