## The Sad Panda


Amazon has the [ability to do a rolling update](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html) to instances in a AutoScale group, but only has exposed that feature to AutoScale groups created & managed with CloudFormation.  

The following happens:


1. AMI is generated
	* Temporary instance provisioned
	* Relevant Ansible roles are applied to that instance
	* AMI is generated with ec2_ami module
	* Temporary instance is removed
	
2. VPC, Subnets, Security groups are created with native Ansible ec2* modules.  The Launch Configuration and the AutoScale group are created with a CloudFormation template that is launched with the CloudFormation module in order to leverage the CloudFormation-only update policy described above.  

