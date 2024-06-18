---

title: Managing Identity Center Users with Terraform

slug: idc-users-with-tf

tags: terraform, aws

domain: blog.ysac.the0x.dev

subtitle: A Gude

cover: 

ignorePost: false

publishAs: 

canonical: 

hideFromHashnodeCommunity: 

seoTitle: 

seoDescription: 

disableComments: true

seriesSlug:

enableToc: true

saveAsDraft:  false

---

Hey Everyone, in this post I am going to run your through how easy it is to provision users and manage AWS managed users leveraging the AWS identity center and organizations service using terraform.

## What is AWS IAM Identity Center

AWS IAM Identity Center, previously known as AWS Single Sign-On, is a service that allows centralized management of users in organizations using a multi-account strategy for developing and deploying business apps. It simplifies access with SSO authentication and enables admins to assign detailed permissions across AWS resources.

### What is AWS Organizations?

AWS Organizations is a service that allows teams to centrally manage multiple AWS accounts in a hierarchical grouping model called organizational units. This service gives businesses significant control over billing, tagging, and service control policies, which can also be managed using Terraform code. In this article, we will use the Organizations service to pull a list of account IDs so we can attach permission sets to specific users and accounts. This provides a level of flexibility that is usually quite cumbersome for a typical IAM administrator to handle.

### Getting Started

#### Permissions

To get started we have a bit of a chicken and the egg problem. We need to have a credentialed access pair that has the permissions in order to deploy our terraform code in order to manage our aws identity center instance. You can create a role and assign the policies AWSSSOMasterAccountAdministrator and AWSSSOMemberAccountAdministrator to the role. Now from here you can create a user and assign this role that you created to it. Now that you have a user with the proper permissions to manage identity center, you are going to need to have a credential pair for you to run your terraform code. You can do this by going to IAM > Users > <your user> > security credentials > access keys > create access key > local code > create access key

#### Creating A Organization

Now that we can the proper credential pairs and permissions, we need to enable the AWS organizations service. We can do this by searching the organizations keyword in the search box at the top of our UI and enabling the service. Once the service is enabled, you will be greeted with a structure similar to the following attached.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716997935588/93a6cb04-e49b-4d4e-b992-83c46450f742.jpeg align="center")

#### Enabling AWS Identity Center

To use AWS Identity Center, we need to enable it as a trusted service within the organization through the trusted access portal in the organizations service. In the organizations service, go to the Services portal, search for AWS IAM Identity Center (AWS Single Sign-On), and enable it. Simple.

Next, go to the IAM Identity Center service and enable the service as a organization service. Once that is completed, you should be greeted with a dashboard similar to the following.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1716998199127/d0359450-4921-4726-bd7b-39ff8f72a966.jpeg align="center")

#### The Code (Slide Breakdown)

%[https://snappify.com/view/8be0b2ae-2756-435d-8b60-6dedb2b96168] 

Alright, now that we have completed the preparation steps, let's look at a quick Terraform example. In this example, we will create a user in our identity center, give them ViewOnly access to our account, and provide them with sign-in credentials.

#### Slide 1 (The Terraform Backend)

In the first slide, we are setting up a Terraform state to reside in an S3 bucket. This bucket needs to be created before running the terraform init command. I prefer using S3 remote states because of the state locking mechanisms (not shown) and the added security of not keeping the state file locally on your machine. Additionally, you can integrate your lock file with Lambdas for other uses when the infrastructure gets updated, which is quite convenient.

#### Slide 2 (Data Sources)

In the second slide we are just pulling data from our identity center instance and our AWS organization service. This will allow us to dynamically configure account ids without the need to hardcode them. This is useful as many organization have more than just a single account and we can perform pattern matching between account names and account ids, which will be shown in a later post.

#### Slide 3 (Creating A Directory User)

In this slide we are creating a directory user named the0x we are setting up the user with a email that is being set with a variable to avoid having hardcoded values in the code base. The identity store id is set to a list, but will be evaluated to a single value since we are only running a single IDC within the us-east-1 region.

#### Slide 4 (Create the Permission Set)

In this slide we are creating a permission set, this is only creating the resource and does not actually have any policies/permissions attached to it at this moment. We are calling this permission set TFMANAGEDREAD_ONLY

#### Slide 5 (Attaching a AWS Managed Policy to the Permission Set)

Now here we are attaching a typical aws managed policy to the permission set. In this case we are pairing the ViewOnlyAccess Policy to the newly created permission set named TFMANAGEDREAD_ONLY

#### Slide 6 (Putting it all together)

Lastly, we are performing a account assignment. This brings the permission set which has the attached aws managed view only policy on it and attaches it to a USER which in this case is our the0x user identity and pairs this with the target\_id which is our AWS account ID for the current account.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717001243413/531768eb-727e-4922-be2d-01d8b8d861c9.jpeg align="center")

When the user logs into the AWS access portal, they will be presented with a list of AWS accounts that have the managed permission set assigned by the identity center administrator. This permission set determines the level of access and permissions the user has within each account. Once logged in, the user can navigate through the various AWS services and resources, but their actions will be limited to the permissions defined in the permission set. Additionally, all sessions can be monitored and tracked for activity, providing administrators with detailed logs of user actions. This tracking capability is crucial for auditing and ensuring compliance with security policies.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717001460826/1a7ea597-6fe6-4ece-83a3-eb1f227288d8.jpeg align="center")

#### Wrap Up

Overall, being able to manage permission sets through an Infrastructure as Code (IaC) toolset like Terraform is an excellent method for IAM administrators to efficiently iterate and scale permissions. By using Terraform, administrators can automate the creation, modification, and deletion of permission sets, ensuring consistency and reducing the potential for human error. This approach not only streamlines the process of managing permissions but also allows for version control and easier auditing. As organizations grow and their infrastructure becomes more complex, leveraging tools like Terraform becomes increasingly valuable, enabling IAM administrators to maintain robust security practices while adapting to changing needs and scaling operations seamlessly.
