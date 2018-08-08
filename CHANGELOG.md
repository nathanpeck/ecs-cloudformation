## August 7th, 2018

- Broke templates up so that there is a cluster template, ingress template, and service template instead of a combined cluster + ingress template.
- Updated the list of allowed instance types in the EC2 cluster template to the latest generation instances of each type.
- Added a CloudWatch logs integration so that logs from containers are automatically sent to CloudWatch
- Added CPU based autoscaling rules to automatically scale the number of containers
- Refactoring all `!Join` to `!Sub` for more readable CloudFormation
