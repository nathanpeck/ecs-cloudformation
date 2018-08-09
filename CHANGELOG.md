## August 9th, 2018

- Fixed mismatched color in the diagram image for private subnet, private LB
- Fixed bug with export namespacing in Fargate clusters
- Found and fixed a few more !Join functions
- Added an example of private, internal service discovery
- Reorganizing the README.md a little bit

## August 7th, 2018

- Broke templates up so that there is a cluster template, ingress template, and service template instead of a combined cluster + ingress template.
- Updated the list of allowed instance types in the EC2 cluster template to the latest generation instances of each type.
- Added a CloudWatch logs integration so that logs from containers are automatically sent to CloudWatch
- Added CPU based autoscaling rules to automatically scale the number of containers
- Refactoring all `!Join` to `!Sub` for more readable CloudFormation
