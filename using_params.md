# CreateParamsFromCFN

## Purpose 
To deploy a public template using a local template/parameters file (or.. use S3.. whatever) 

## Overview
So - there is a rather amazing WordPress reference architecture that I happened upon a few months ago.  I use it for many things.  I also spin-up and tear-down the environment quite frequently .. and doing it from the webUI is tedious.

## Process
Run the CFN once using the CloudFormation webUI via the console.  
Grab the Parameters and create a text file.
Parse the text file and run some foo to create the JSON output.
Party

```
REGION="us-west-2"   
STACK_NAME="HEXGL"  
OUTPUT="params-${STACK_NAME}-${REGION}.json"  
INPUT="params-${STACK_NAME}-${REGION}.cfn"  
OPTIONS=" --disable-rollback"  
#- create params.cfn which will be a cut-and-paste of the "Parameters" section of the CFN creation page.  
# I added "DUMMY" as there is a new field (-k3) in the output now (what a PITA :-(
grep -v ^# $INPUT | while read ParameterKey ParameterValue DUMMY; do echo -e "  {\n    \"ParameterKey\": \"${ParameterKey}\",\n    \"ParameterValue\": \"${ParameterValue}\"\n  },"; done > ${OUTPUT}  
#- once you have the params.json created, add open/close square-brackets at the beginning and end  
case `sed --version | head -1` in  
  *GNU*)  
    # Remove the entries which are just a hypen "-"
    sed -i -e 's/"-"/""/g' $OUTPUT
    # Add the '[' to the beginning of the file  
    sed -i -e '1i[' ${OUTPUT}  
    # Remove the trailing ',' from the last entry
    sed  -i '$ s/},/}/' ${OUTPUT}
  ;;
  *)
sed -i'' '1i\
[\
' ${OUTPUT}
  ;;
esac
echo "]" >> ${OUTPUT}

aws cloudformation create-stack --stack-name "${STACK_NAME}" \
  --template-body https://s3.amazonaws.com/aws-refarch/wordpress/latest/templates/aws-refarch-wordpress-master-newvpc.yaml \
  --parameters file://`pwd`/${OUTPUT} \
 --region ${REGION} --capabilities CAPABILITY_IAM ${OPTIONS}

aws cloudformation describe-stack-events --stack-name "${STACK_NAME}" --region ${REGION}
```

https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=WordPress&templateURL=https://s3.amazonaws.com/aws-refarch/wordpress/latest/templates/aws-refarch-wordpress-master-newvpc.yaml
