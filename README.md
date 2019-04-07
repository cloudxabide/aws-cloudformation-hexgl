# aws-cloudformation-hexgl
Cloudformation using nested templates to deploy blkore-hexgl

## Status
Work in Progress: 
  I am moving this to it's own Repo 

## Resources 
[templates](templates) -- Adding route53 to my nested stack deployment.
[templates/aws-hexgl-01-newvpc.yaml](templates/aws-hexgl-01-newvpc.yaml)
[templates/aws-hexgl-02-securitygroups.yaml](templates/aws-hexgl-02-securitygroups.yaml)
[templates/aws-hexgl-03-bastion.yaml](templates/aws-hexgl-03-bastion.yaml)
[templates/aws-hexgl-03-publicalb.yaml](templates/aws-hexgl-03-publicalb.yaml)
[templates/aws-hexgl-04-web.yaml](templates/aws-hexgl-04-web.yaml)
[templates/aws-hexgl-05-route53.yaml](templates/aws-hexgl-05-route53.yaml)
[templates/aws-hexgl-master-newvpc.yaml](templates/aws-hexgl-master-newvpc.yaml)


## CFN Launch Links
stackName: HEXGL

| AWS Region | Region Name | Launch Button
| --- | --- | ---
| us-east-1 | US East (N. Virginia) |  [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=HEXGL&templateURL=https://s3.amazonaws.com/cloudxabide/aws-cloudformation/templates/aws-hexgl-master-newvpc.yaml) |
| us-east-2 | US East (Ohio) | [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/new?stackName=HEXGL&templateURL=https://s3.amazonaws.com/cloudxabide/aws-cloudformation/templates/aws-hexgl-master-newvpc.yaml) |
| us-west-1 | US West (N. California) | [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-1#/stacks/new?stackName=HEXGL&templateURL=https://s3.amazonaws.com/cloudxabide/aws-cloudformation/templates/aws-hexgl-master-newvpc.yaml) |
| us-west-2 | US West (Oregon) | [![cloudformation-launch-stack](images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=HEXGL&templateURL=https://s3.amazonaws.com/cloudxabide/aws-cloudformation/templates/aws-hexgl-master-newvpc.yaml) |

If you prefer the CLI
```
REGION="us-east-1"
STACK_NAME="HEXGLE1"
aws cloudformation create-stack --stack-name "${STACK_NAME}" --template-url https://s3.amazonaws.com/cloudxabide/aws-cloudformation/templates/aws-hexgl-master-newvpc.yaml --parameters https://s3.amazonaws.com/cloudxabide/aws-cloudformation/templates/params-${REGION}.json --region ${REGION} --capabilities CAPABILITY_IAM ${OPTIONS}
```
## References
[Wordpress multi-AZ reference architecture](https://github.com/aws-samples/aws-refarch-wordpress)
[Tips and Tricks](./tips-n-tricks.md)
