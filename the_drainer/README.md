1. Create the new AMI.
2. disable autoscale events (possible? ) http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/US_SuspendResume.html, http://boto.readthedocs.org/en/latest/ref/autoscale.html
3. Update Launch Config with new AMI
3. serially do the following
	*  loop through all machines in autoscale group
	*  terminate instance ( with connection draining )
	*  wait for new instance to be provisioned in its place and considered healthy by LB (LB should increment instance count should be equal to desired state)
	
	or
	
	*  spin up new instance with new AMI, with same VPC, security group info, etc
	*  attach to new autoscale group (may require new function to ec2/ASG module but is a supported AWS feature -- http://boto.readthedocs.org/en/latest/ref/autoscale.html