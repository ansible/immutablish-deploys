http://www.thoughtworks.com/insights/blog/implementing-blue-green-deployments-aws

Provision new stuff.  
Must pre-warm ELB before cut-over (start with pre-warmed ELB???), open ticket with AWS.
Make DNS change.
On each server, direct old traffic to new ELB (socat) to account for DNS propogation.
Once old traffic is finished, terminate old ELB and Autoscale Group.