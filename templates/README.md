# hexGL on AWS 

## Purpose:
Deploy a multi-AZ instance of bkcore's HexGL game.  
This was particularly fun.  I learned some (nuanced) things while exploring the addition of
route53 (code below).

## NOTES:
There is still some clean up to be done.  I have been using this project to develop my AWS and DevOps skills.  
It will only run on Firefox nowadays.

## References:
http://hexgl.bkcore.com/  
https://github.com/aws-samples/aws-refarch-wordpress


## Troubleshooting and "Lessons Learned"
### Reference Architectures are not infallable.
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

Z1H1 FL5H ABSF 5   (fail)
Z368 ELLR RE2K J0  (succeed)
```

I have not been able to find what the "standard" is regarding the length, but, for now, I have modified the code.
```
  DnsHostId:
    AllowedPattern: ^[A-Z0-9]{10,14}$
```
