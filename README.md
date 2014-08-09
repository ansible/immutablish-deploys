## ami_build


1. AMI is generated
2. Temporary instance provisioned
3. Relevant Ansible roles are applied to that instance
4. AMI is generated with ec2_ami module
5. Temporary instance is removed
6. AMI id is registered




## cfn_update_policy.yml
Amazon has the [ability to do a rolling update](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html) to instances in a AutoScale group, but only has exposed that feature to AutoScale groups created & managed with CloudFormation.  


Inputs: ami_build's registered AMI id

The following happens:

	
1. VPC, Subnets, Security groups are created with native Ansible ec2* modules.  
2. The Launch Configuration and the AutoScale group are created with a CloudFormation template that is launched with the CloudFormation module in order to leverage the CloudFormation-only update policy described above. 


## blue.yml / violet.yml

Inputs: ami_build's registered AMI id

1. One provisioned autoscaling group is running
2. New autoscaling group and launch configuration provisioned side-by-side
3. Role waits for new instances to become available
3. Role terminates old autoscaling group 

## rolling_ami.yml

Inputs: ami_build's registered AMI id

1. New launch configuration is generated if current launch_config does not match
2. Role builds autoscale group 
2. Role applies rolling AMI approach to ASG instances if the launch_configuration has changed
	
	
## blue green deployments

Theoretical for now.

http://www.thoughtworks.com/insights/blog/implementing-blue-green-deployments-aws

Provision new stuff.  
Must pre-warm ELB before cut-over (start with pre-warmed ELB???), open ticket with AWS.
Make DNS change.
On each server, direct old traffic to new ELB (socat) to account for DNS propogation.
Once old traffic is finished, terminate old ELB and Autoscale Group.## The Sad Panda

