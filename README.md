# aws-cloudformation-hexgl
## Purpose:  
Deploy a multi-AZ instance of bkcore's HexGL game.  Additionally this example employs a nested stack approach - each stack addressing a particular "concern".  
This was particularly fun.  I learned some (nuanced) things while exploring the addition of route53 (code below).

## Status
Work in Progress: 
I decided to put all this in it's own repo and may still need some tweaking as a result.

## Resources 
The file names should be fairly explanatory.  
[templates](templates) -- directory for all the nested templates  
[templates/aws-hexgl-01-newvpc.yaml](templates/aws-hexgl-01-newvpc.yaml)  
[templates/aws-hexgl-02-securitygroups.yaml](templates/aws-hexgl-02-securitygroups.yaml)  
[templates/aws-hexgl-03-bastion.yaml](templates/aws-hexgl-03-bastion.yaml)  
[templates/aws-hexgl-03-publicalb.yaml](templates/aws-hexgl-03-publicalb.yaml)  
[templates/aws-hexgl-04-web.yaml](templates/aws-hexgl-04-web.yaml)  
[templates/aws-hexgl-05-route53.yaml](templates/aws-hexgl-05-route53.yaml)  

## CFN Launch Links
stackName: HEXGL

| AWS Region | Region Name | Launch Button
| --- | --- | ---
| us-east-1 | US East (N. Virginia) |  [![cloudformation-launch-stack](Images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=HEXGL&templateURL=https://s3.amazonaws.com/cloudxabide/aws-cloudformation-hexgl/templates/aws-hexgl-master-newvpc.yaml) |
| us-east-2 | US East (Ohio) | [![cloudformation-launch-stack](Images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/new?stackName=HEXGL&templateURL=https://s3.amazonaws.com/cloudxabide/aws-cloudformation-hexgl/templates/aws-hexgl-master-newvpc.yaml) |
| us-west-1 | US West (N. California) | [![cloudformation-launch-stack](Images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-1#/stacks/new?stackName=HEXGL&templateURL=https://s3.amazonaws.com/cloudxabide/aws-cloudformation-hexgl/templates/aws-hexgl-master-newvpc.yaml) |
| us-west-2 | US West (Oregon) | [![cloudformation-launch-stack](Images/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=HEXGL&templateURL=https://s3.amazonaws.com/cloudxabide/aws-cloudformation-hexgl/templates/aws-hexgl-master-newvpc.yaml) |

Or, if you prefer the CLI:  
```
REGION="us-east-1"
STACK_NAME="HEXGLE1"
aws cloudformation create-stack --stack-name "${STACK_NAME}" --template-url https://s3.amazonaws.com/cloudxabide/aws-cloudformation-hexgl/templates/aws-hexgl-master-newvpc.yaml --parameters https://s3.amazonaws.com/cloudxabide/aws-cloudformation-hexgl/Params/params-${REGION}.json --region ${REGION} --capabilities CAPABILITY_IAM ${OPTIONS}
```
## References
[Wordpress multi-AZ reference architecture](https://github.com/aws-samples/aws-refarch-wordpress)  This is the example from which I based my work on.  
[Tips and Tricks](./tips-n-tricks.md)  
[HexGL BKcore](http://hexgl.bkcore.com/)

## Troubleshooting and "Lessons Learned"
### Reference Architectures may not be infallible over time  
While I was debriefing and explaining that I had discovered a potential issue with the Reference Architecture  to one of our senior DevOps folks, he replied "Ya... I told you... this happens".  He was referring to a previous conversation about some old technical debt and how it had once been a Gold Standard and now needed refactoring because of extended periods of neglect and changing infrastrcuture.

Observe the following code (and see if you can immediately see what the potential issue is/was:
```
$ grep -A1 ^\ \ DnsHostId: *route*
  DnsHostId:
      AllowedPattern: ^[A-Z0-9]{14}$
```

Now - what made this issue interesting, is that my CloudFormation worked in us-east-1... then us-east-2.  But, not when I attempted in us-west-2.  My stack would eventually fail, and rollback my "route53" template, generating the message "DnsHostId must be ^[A-Z0-9]{14}$" (paraphrasing).  So, at my first thoughts were that either
* my route53 interaction was failing (returning null, or an error)
* there were some characters being returned that were not Capital letters [A-Z] or Number [0-9]

It wasn't until I figured out how to "--disable-rollback" and troubleshooting the output, that I discovered the DnsHost Zone Id values being returned were different lengths.
```
$ aws route53 list-resource-record-sets --hosted-zone-id Z1F045AL9FKCNH | grep HostedZoneId
                "HostedZoneId": "Z3AADJGX6KTTL2",
                "HostedZoneId": "Z1H1FL5HABSF5",

Z368 ELLR RE2K J0  (succeed)
Z1H1 FL5H ABSF 5   (fail)
```
For fun...
```
$ for ENTRY in `aws route53 list-resource-record-sets --hosted-zone-id Z1F045AL9FKCNH | grep HostedZoneId | awk -F'"' '$0=$4'`; do echo $ENTRY | wc ; done
         1       1      15
         1       1      14
```

I have not been able to find what the "standard" is regarding the length, but, for now, I have modified the code.
```
  DnsHostId:
    AllowedPattern: ^[A-Z0-9]{10,14}$
```
