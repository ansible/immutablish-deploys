# Building and Deploying AMIs to AutoScaling Groups with Ansible
---


As we discussed before, there are mutliple ways to use AWS AutoScaling groups with Ansible.  In the previous article, we covered how you can use Ansible Tower's callback mechanism to provision freshly provisioned AutoScale instances.  This time, we are going to talk about how to use Ansible to build and deploy AMIs to AutoScale groups.

## Building AMIs

Ansible is fully capable of building an AMI for your application using the ec2_ami module.  The ec2_ami will take a running instance and create an AMI from it.  You first want to have an Ansible role(s) you'd like to apply to an instance.  Once your role development is complete, you can use it to define how you'd like your AMI to appear. 

The typical AMI-building playbook will:

1. Launch a temporary instance.
2. Apply a set of roles to that instance.
3. Run the ec2_ami module against that instance.
4. Terminate the temporary instance.

[This playbook](build_ami.yml) is a reference example of how to build a custom AMI.  Once that custom AMI is complete, the AMI it generates can be used as input to one of the following approaches.



## AutoScale Update Policy


Amazon has the [ability to do a rolling update](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html) to instances in a AutoScale group, but only has exposed that feature to AutoScale groups created & managed with CloudFormation.  

When using an ASG update policy, if the ASG's associated Launch Configuration has changed, the instances in the ASG will be replaced serially to match the new Launch Configuration.  Since an AMI ID is part of a Launch Configuration, we can leverage this update policy feature to deploy instances into a ASG with a newly specified AMI, and do so in a rolling fashion.


If you take a look at [this playbook](playbooks/cfn_update_policy.yml) and its respective, you can see the following happening:

	
1. VPC, Subnets, Security groups [are created](roles/infra) with native Ansible ec2* modules.  
2. [In a separate role](roles/asgcfn/tasks/main.yml), the Launch Configuration and the AutoScale group are created with a [CloudFormation template](roles/asgcfn/files/asg-cfgn.json) that is launched with the CloudFormation module in order to leverage the CloudFormation-only update policy described above. 

This workflow can be launched with:

	ansible-playbook -i build_ami cfn_update_policy.yml


## Rolling AMI deploy with pure Ansible

You don't *have* to use CloudFormation's Update Policies to leverage a rolling replacement approach for intsances in an ASG, you can also do it with native Ansible modules.  The ec2_asg module in development branch of Ansible (scheduled for 1.8) supports this.


1. New AMI is generated
2. New launch configuration name is generated with new AMI ID.
3. Autoscale Group is assigned the launch configuration
4. A list of instances not associated with the current launch configuration is collected
5. The desired capacity and minimum size of the ASG are altered to account for the batch size of the rolling termination.  
6. The list of instances we collected through before is looped through by the batch size, and each instance is terminated.  After an instance terminated, we wait for the replacement to come online.
7. After all instances are replaced, we set the ASG to its normal state.

This workflow can be launched with:

	ansible-playbook build_ami rolling_ami.yml -vv -e "deploy=yes"



## Blue - Violet

The concept behind the blue violet approach is as follows:

1. One provisioned autoscaling group is running and behind a ELB
2. New autoscaling group and launch configuration provisioned side-by-side, behind the same ELB.
3. Ansible waits for new instances to become available in the AutoScale group.
4. Role terminates old autoscaling group and associated instances.

The main draw back to this approach is that you effectively have to have double the instances for a small amount of time while you are deploying your new AMI to the new ASG.  The plus side is that is much faster than performing a rolling approach.

This workflow can be launched with:

	ansible-playbook build_ami blue.yml -vv -e "deploy=yes"
	
and when you're ready for the violet to take over

	ansible-playbook build_ami violet.yml -vv -e "deploy=yes terminate_relative=yes"
	
## Blue Green deployments

The approach behind a blue-green deployment is outlined here:

http://www.thoughtworks.com/insights/blog/implementing-blue-green-deployments-aws

It entails provisioning a new Autoscale group and ELB and cutting the traffic over to the new cluster. The problem with this approach is two fold: you must pre-warm ELB before the cut-over.  This is a manual process which requires contacting AWS support.  You'd also have to make DNS change so that your website points to the CNAME of the new ELB.  This is not instant--it could take days for DNS changes to propagate amungst the world's DNS infrastructure.  One potential workaround would be to use a tool such as socat on the old cluster to direct incoming traffic to the new ELB while DNS is propagating the updated record.  Over time, the connections to the old clusters should diminish, and then the cluster can be manually terminated.

