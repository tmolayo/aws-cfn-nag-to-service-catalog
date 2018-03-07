# aws-cfn-nag-to-service-catalog

## Warning
Hey - hi - over here. So listen. I want to tell you something that's pretty important. I know - I know - I don't want this to be awkward either but we should have THE talk. Let's just do this quickly so that we can get on with it.

> THIS CODE IS FOR DEMONSTRATION PURPOSES ONLY. DO NOT DEPLOY IT INTO PRODUCTION. 

Okay. With that out of the way let's do this.

Git clone this repo.

```
git clone https://github.com/frc9/aws-cfn-nag-to-service-catalog.git
```

Submit the cloudformation template in the cloned repo to the cloudformation service. Note that based on the command line below the codecommit repo is going to be called central-service-catalog. It can be whatever you like as long as you stick to the following: 

>Any combination of letters, numbers, periods, underscores, and dashes between 1 and 100 characters in length. Repository names cannot end in .git and cannot contain any of the following characters: ! ? @ # $ % ^ & * ( ) + = { } [ ] | \ / > < ~ ` ‘ “ ; :

```
aws cloudformation deploy --template-file ./aws-cfn-nag-to-service-catalog/pipeline-to-service-catalog.yaml \
--stack-name service-catalog-sync-pipeline --capabilities CAPABILITY_NAMED_IAM \
--parameter-overrides RepositoryName=central-service-catalog
```

Once the stack is done building in the account you will now need to clone the newly created central-service-catalog repo.

Paste the following commands:

```
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true
```
Clone your repository to your local computer and start working on code:

```
git clone https://git-codecommit.us-west-2.amazonaws.com/v1/repos/central-service-catalog
```

So now that you have this empty repo you have to put into it the required files that make the magic of the service catalog stuff work. Things like the mapping file and how portfolios are constructed are explained here: https://github.com/awslabs/aws-pipeline-to-service-catalog.

For now you can use the sample portfolio construct that I showed you that shoves an ec2 instance product into the service catalog. Execute the following:

```
cp ./aws-cfn-nag-to-service-catalog/lambda-cloudformation.yaml ./central-service-catalog/
cp -R ./aws-cfn-nag-to-service-catalog/portfolio-infrastructure/ ./central-service-catalog/portfolio-infrastructure/
cp -R ./aws-cfn-nag-to-service-catalog/scripts/ ./central-service-catalog/scripts/
```

Your central-service-catalog directory should now look like this:

```
bash-3.2$ tree ./central-service-catalog/
./central-service-catalog/
├── lambda-cloudformation.yaml
├── portfolio-infrastructure
│   ├── mapping.yaml
│   └── product-company-ec2-p6.yml
└── scripts
    ├── requirements.txt
    └── sync-catalog.py
```

Before we commit this repo to start the pipeline lets take a look at the mapping.yaml file and then the product-company-ec2-p6.yml.

This mapping file maps the cloduformation template. In this example you'll want to change the accountID's or remove them if you don't want to share the portfolios and you'll want to make sure that the principals exist in your aws account. Again you can find more info here: https://github.com/awslabs/aws-pipeline-to-service-catalog.

```
bash-3.2$ cat ./central-service-catalog/portfolio-infrastructure/mapping.yaml 
name: COMPANY_CENTRALARCH_BU_IAAS 
description: Company Central IT approved IAAS portfolio BU
owner: company_bu_iaas_portfolio_owner@company.com
products:
- name: company-EC2-P6
  template: product-company-ec2-p6.yml
  owner: company_ec2_pattern_owners@company.com
  description: Creates a multi-tenant ec2 instance with KMS encrypted root and data volumes and dynamic public ip address. Allowed data classifications include public non-material.
accounts:
- identifier: Sample AWS Account
  number: 111111111111 
- identifier: Another
  number: 111111111112
tags:
- Key: ProductCode 
  Value: /aws/bu/iaas/ec2/company-ec2-p6
- Key: ProductWiki
  Value: https://w.company.com/aws/bu/iaas/ec2/company-ec2-p6/README.md
principals:
- user/demouser 
```

The product-company-ec2-p6.yml file is written such that it will pass the cfn nag build stage. It will not actually deploy within service catalog because it has a security egreess rule without a vpc id. I suggest once you have the pipeline working to take one of your own cloudformation templates and run it through. You'll be surprised how picky cfn nag is.

Now you're ready to commit and push the central service catalog repo into the deployed pipeline like so:

```
cd central-service-catalog/
git add --all
git commit -am "initial commit"
git push
```

If the demo gods are just and all is good with the world you'll end up with a portfolio in service catalog with you new product. Ta da! (Or sorry - give me call)
