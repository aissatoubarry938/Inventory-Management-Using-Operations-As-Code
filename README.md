
# INVENTORY MANAGEMENT USING OPERATIONS AS CODE
  
## Management Tools: Systems Manager
 
### 4.1 Setting up Systems Manager
1. Use your administrator account to access the Systems Manager console at https://console.aws.amazon.com/systems-manager/.
2. Choose Fleet Manager from the navigation bar in the Node Management menu. If you have not satisfied the pre-requisites for Systems Manager, you will arrive at the AWS Systems Manager Managed Instances page.
      - As a user with AdministratorAccess permissions, you already have User Access to Systems Manager.
      - The Amazon Linux AMIs used to create the instances in your environment are dated 2017.09. They are supported operating systems and have the SSM Agent installed by default.
      - If you are in a supported region the remaining step is to configure the IAM role for instances that will process commands.
3. Create an Instance Profile for Systems Manager managed instances:
     1. Navigate to the IAM console
     2. In the navigation pane, choose Roles.
     3. Then choose Create role.
     4. In the Select type of trusted entity section, verify that the default AWS service is selected.
     5. In the Choose the service that will use this role section, scroll past the first reference to EC2 (EC2 Allows EC2 instances to call AWS services on your behalf) and choose EC2 from within the field of services. This will open the Select your use case section further down the page.
     6. In the Select your use case section, choose EC2 Role for AWS Systems Manager to select it.
     7. Then choose Next: Permissions.
4. Under Attached permissions policy, verify that AmazonEC2RoleforSSM is listed, and then choose Next: Tags.
5. In the Tags section:
    1. Add one or more keys and values, then choose Next: Review.
6. In the Review section:
   1. Enter a Role name, such as ManagedInstancesRole.
   2. Accept the default in the Role description.
   3. Choose Create role.
7. Apply this role to the instances you wish to manage with Systems Manager:
   1. Navigate to the EC2 Console and choose Instances.
   2. Select the first instance and then choose Actions, Security, and Modify IAM Role.
   3. Under Modify IAM Role, select ManagedInstancesRole from the drop down list and choose Save.
   4. Repeat this process, assigning ManagedInstancesRole to each of the 3 remaining instances.
8. Return to the Systems Manager console and choose Fleet Manager from the navigation bar in the Node Management menu. Periodically choose Fleet Manager until your instances begin to appear in the list. Over the next couple of minutes your instances will populate into the list as managed instances.
  
**Note** If desired, you can use a more restrictive permission set to grant access to Systems Manager.
  
![roles](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/2d36d738-6ac3-4d8d-b81f-b47e6a596d5d)
  
![Managed role](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/377d0e23-0ec7-4c1f-b73a-1aed956d51ba)
  
![fleet manager](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/6ccb6b88-38fc-4b3c-a6a3-2532272b1d25)


## 4.2 Create a Second CloudFormation Stack
  
1. Create a second CloudFormation stack using the procedure in 3.1 with the following changes:
  - In the Specify Details section, define a Stack name, such as OELabStack2.
  - Specify the InstanceProfile using the ManagedInstancesRole you defined.
  - Define the Workload Name as Prod.
  
 ![stack2](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/eb724c40-63e9-4f7c-a2b9-70c613b7fafa)
  
![stack2b](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/eda8fbd9-00cc-4d9c-bc00-1e90bfc72019)

# Systems Manager: Inventory
  
You can use AWS Systems Manager Inventory to collect operating system (OS), application, and instance metadata from your Amazon EC2 instances and your on-premises servers or virtual machines (VMs) in your hybrid environment. You can query the metadata to quickly understand which instances are running the software and configurations required by your software policy, and which instances need to be updated.

## 4.3 Using Systems Manager Inventory to Track Your Instances
1. Under Node Management in the AWS Systems Manager navigation bar, choose Inventory.
    1. Scroll down in the window to the Corresponding managed instances section. Inventory currently contains only the instance data available from the EC2
    2. Choose the InstanceID of one of your systems.
    3. Examine each of the available tabs of data under the Instance ID heading.
2. Inventory collection must be specifically configured and the data types to be collected must be specified
    1. Choose Inventory in the navigation bar.
    2. Choose Setup Inventory in the top right corner of the window
3. In the Setup Inventory screen, define targets for inventory:
    1. Under Specify targets by, select Specifying a tag
    2. Under Tags specify Environment for the key and OELabIPM for the value
  
**Note** You can select all managed instances in this account, ensuring that all managed instances will be inventoried. You can constrain inventoried instances to those with specific tags, such as Environment or Workload. Or you can manually select specific instances for inventory.

4. Schedule the frequency with which inventory is collected. The default and minimum period is 30 minutes
    1. For Collect inventory data every, accept the default 30 Minute(s)
5. Under parameters, specify what information to collect with the inventory process
    1. Review the options and select the defaults
6. (Optional) If desired, you may specify an S3 bucket to receive the inventory execution logs (you will need to create a destination bucket for the logs prior to proceeding):
    1. Check the box next to Sync inventory execution logs to an S3 bucket under the Advanced options.
    2. Provide an S3 bucket name.
    3. (Optional) Provide an S3 bucket prefix.
7. Choose Setup Inventory at the bottom of the page (it can take up to 10 minutes to deploy a new inventory policy to an instance).
8. To create a new inventory policy, from Inventory, choose Setup inventory.
9. To edit an existing policy, from State Manager in the left navigation menu, select the association and choose Edit.
  
**Note** You can create multiple Inventory specifications. They will each be stored as associations within Systems Manager State Manager.
  
 ![bucket](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/4d33a8e3-9ff5-43f9-a68f-9efd420b503f)

![inventory](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/6ca9df5c-bebf-4851-b668-261c55b0f2fa)

# Systems Manager: State Manager
  
In State Manager, an association is the result of binding configuration information that defines the state you want your instances to be in to the instances themselves. This information specifies when and how you want instance-related operations to run that ensure your Amazon EC2 and hybrid infrastructure is in an intended or consistent state.

An association defines the state you want to apply to a set of targets. An association includes three components and one optional set of components:

  - A document that defines the state
  - Target(s)
  - A schedule
  - (Optional) Runtime parameters.
  
When you performed the Setup Inventory actions, you created an association in State Manager.

## 4.4 Review Association Status

 1. Under Node Management in the navigation bar, select State Manager. At this point, the Status may show that the inventory activity has not yet completed.
    1. Choose the single Association id that is the result of your Setup Inventory action.
    2. Examine each of the available tabs of data under the Association ID heading.
    3. Choose Edit.
    4. Enter a name under Name - optional to provide a more user friendly label to the association, such as InventoryAllInstances (white space is not permitted in an Association Name).
  
*Inventory* is accomplished through the following:

- The activities defined in the AWS-GatherSoftwareInventory command document.
- The parameters provided in the Parameters section are passed to the document at execution.
- The targets are defined in the Targets section.
  
**Important** In this example there is a single target, the wildcard. The wildcard matches all instances making them all targets.

- The schedule for this activity is defined under Specify schedule and Specify with to use a CRON/Rate expression on a 30 minute interval.
- There is the option to specify Output options.
  
**Note** If you change the command document, the Parameters section will change to be appropriate to the new command document.

2. Navigate to Fleet Manager under Node Management in the navigation bar. An Association Status has been established for the inventoried instances under management.
3. Choose one of the Instance ID links to go to the inventory of the instance. The Inventory tab is now populated and you can track associations and their last activity under the Associations tab.
4. Navigate to Compliance under Node Management in the navigation bar. Here you can view the overall compliance status of your managed instances in the Compliance Resources Summary and the individual compliance status of systems in the Corresponding managed instances section below.
  
 ![Compliance](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/f6f93c44-089b-40a9-aa4a-929bedf0a737)
  
![inventory_c](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/9c4258b4-691c-4a85-9722-57e03ddf9c17)
  
![inventory_b](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/2d21ff39-4b95-4a31-afb7-47d35f6fbf50)
  
![Compliance_b](https://github.com/aissatoubarry938/Inventory-and-Patch-Management/assets/115582067/b6f20abc-1112-49a7-94b8-2416e11920d4)

  
