# Disaster Recovery



## Introduction

DXP Cloud provides two ways for customers to take advantage of the Disaster Recovery (DR) procedure in the case of major incidents.

- **Automatic Disaster Recovery:** It's important to understand that <u>DXP Cloud already performs automatic disaster recovery by replicating services between three Availability Zones in different geographic locations within the same Region</u>. In case any of the Availability Zones become unavailable, the Load Balancer automatically routes to the remaining Availability Zone without requiring DNS changes on the customer side. In this situation, no action is required from the customer during an incident.
- **Cross-Region Disaster Recovery:** The Disaster Recovery procedure explained in this document is only necessary when there's a compromise in all three Availability Zones in the same region at the same time, which is extremely unlikely to occur. In this situation, customers will use the DR environment and follow the instructions explained in this document.



## Cross-Region Disaster Recovery Strategies

There are multiple DR strategies in the IT industry and they are considered to be cold, warm, or hot. These strategies indicate how prepared the system is to recover when something goes wrong.
An analogy would be how you would respond if you were driving and had a flat tire. How you deal with this situation depends on how equipped you are:

|||
| -------- | ------------------------------------------------------------ |
| **Cold** | You have no spare tire, so you must call someone to come to you with a new tire and replace it. Your trip stops until help arrives to make the repair. |
| **Warm** | You have a spare tire and a replacement kit, so you can get back on the road using what you have in your car. However, you must stop your journey to repair the problem. |
| **Hot**  | You have special run-flat tires. You might need to slow down a little, but there is no immediate impact on your journey. Your tires run well enough that you can continue (although you must eventually address the issue). |

In DXP Cloud, we classify our DR strategy as *Warm*.

It's not *Cold* since you don't have to contact our support in order to solve that problem for you. It's not *Hot* because you need human intervention to handle it. We consider it to be *Warm* since we provide you with all the tools to realize the procedure.



## Use Cases
We’ll use two examples to show how a DR environment setup can be helpful and step-by-step instructions for getting it to work as expected. Example 1 deals with configuring the DR environment once disaster has struck. Example 2 deals with how to restore the Production environment once the disaster has passed.

### **1. Once the event of a disaster happens**
![Picture 1: Restoring from Production to Disaster Recovery Environment](../../images/dr-01.png)

|              |                                                              |
| ------------ | ------------------------------------------------------------ |
| **12:30 pm** | At this time everything is normal with our Production environment, which is located in *eu-west2* region and users are able to use our system with no problems. |
| **1:00 pm**  | Our automatic backup is triggered, which will generate a copy of our Database and Document Library. |
| **2:00 pm**  | Our automatic backup is triggered once again since we configured it to run every hour. |
| **2:34 pm**  | We are notified that the entire *eu-west2* region has been compromised. This means we'll need to enable our DR environment, which is located in the *us-west1* region, in order to minimize downtime.<br/>From **2:00 pm** until **2:34 pm** there were no backups generated, therefore we'll need to restore the **2:00 pm** backup of the Production database and document library to the DR environment. In addition, we'll get the latest Build number from Production and deploy it to the DR environment, validate if the VPN has been configured properly in the DR environment, and finally change the Custom Domain to serve traffic to users.<br/>The amount of time those steps will take depends on the size of your application and how fast your team will be able to respond, but let's say it took 16 minutes. |
| **3:00 pm**  | Everything is back to normal and now we're serving traffic through the DR environment. |



###  **2. Once the disaster is no longer present**


![Picture 2: Restoring from Disaster Recovery Environment to Production](../../images/dr-02.png)

|              |                                                              |
| ------------ | ------------------------------------------------------------ |
| **4:00 pm**  | we are notified that the *eu-west2* region has been recovered and the incident is over, which means that we are ready to go back to our original Production environment |
| **4:16 pm** | We'll perform a manual backup since there has been new user data generated during the period we were using the DR environment. |
| **4:21 pm**  | We can use the manual backup we just generated to restore the Production environment and then change the Custom Domain. |

Now we're back to serving traffic through the Production environment and our DR environment will stay running for when we need it again.



## Initial Setup

In order to start using the Disaster Recovery environment, it's important to review a few items that are needed to ensure optimum experience and achieve the least downtime possible. These steps can and should be taken before disaster even happens.

### **1. Make sure the Disaster Recovery Environment has been created and that you have Admin access to both environments**

There are certain actions that the Liferay Cloud team needs to perform before you can use this feature such as marking the **dr** environment as a production environment. Make sure to contact us to confirm that these configurations have been made.

You must also be an Admin member on both environments.  

![Picture 3](../../images/dr-03.png)

### 2. Make sure the VPN has been configured in the DR environment**

If you have VPN enabled on the Production environment, you need to make sure that has also been configured in the DR environment.

![Picture 4](../../images/dr-04.png)


### **3. Deploy the latest stable Build from Production to DR environment**

Go to the Builds page and find the number of the latest build that was deployed to the Production environment. Select that build and deploy it to the DR environment.


![Picture 5](../../images/dr-05.png)
![Picture 6](../../images/dr-06.png)

## Automatic Data Recovery Workflow

The ability to recover the database and documents from a backup in an automatic way is not available yet.

However, we will introduce a feature called *Auto Restore* which will allow users to set up a scheduler to automate the restore processes from one environment to the other.

This will improve the overall time to recover DR environments and decrease the chances of human error.

<u>The goal is to deliver this feature to customers in mid-Q4.</u>



## Manual Data Recovery Workflow

The workflow below contains a more detailed list of steps that are currently needed in order to restore a backup of database and documents from the Production environment to the Disaster Recovery environment.

It covers both the event of a disaster happening and also what to do after the disaster is no longer present.

### 1. Once the event of a disaster happens

#### 1.1 Restore the latest Backup from Production to DR environment

Once the build is deployed to the DR environment, restore the latest backup from Production to the DR environment.


![Picture 7](../../images/dr-07.png)
![Picture 8](../../images/dr-08.png)

#### 1.2 Configure Web Server custom domain in the DR environment

Remove the custom domain from the Production environment, update your DNS records, and update the custom domain from the DR environment.

![Picture 9](../../images/dr-09.png)

### 2. Once the disaster is no longer present

#### 2.1 Create a Manual Backup in the DR environment

Before moving back to the Production environment, it's important to create a new backup from the DR environment since there has been new data generated.

![Picture 10](../../images/dr-10.png)

#### 2.2 Restore the Manual Backup from DR to Production environment

Now you can restore the Manual Backup that was just made from DR into the Production environment.


![Picture 11](../../images/dr-11.png)
![Picture 12](../../images/dr-12.png)

#### 2.3 Configure Web Server custom domain in the Production environment

Remove the custom domain from the DR environment, update your DNS records, and update the custom domain from the Production environment.


![Picture 13](../../images/dr-13.png)



## Glossary

|                    |                                                              |
| ------------------ | ------------------------------------------------------------ |
| DR                 | Short for Disaster Recovery                                  |
| Region             | A Region is a separate geographic area                       |
| Availability Zones | Each Region has multiple, isolated locations known as Availability Zones. |
