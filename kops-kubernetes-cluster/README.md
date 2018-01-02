# Kubernetes Cluster

With this code you will be able to deploy y aws a Kubernetes cluster with 3 masters deployed in three different availability zones in the selected region. Also, you can include as many slaves as you want spread on those three availability zones.

## Prerequisite

Tools needed to use and work on this project.

- AWS Cli (http://docs.aws.amazon.com/cli/latest/userguide/installing.html)
- kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- kops (https://github.com/kubernetes/kops#installing)

## Configure AWS Cli

Run this command to configure your credentials and profile to be used by AWS Cli and Kops.

    aws configure  

## Creating a Kubernetes Cluster  

1- Create Hosted Zone, ensure the DNS is resolvable from your computer.

    aws route53 create-hosted-zone --name <HOSTED_ZONE_NAME> --caller-reference <CALLER_REFERENCE>

Where:  

- HOSTED_ZONE_NAME: The name of the domain. For resource record types that include a domain name, specify a fully qualified domain name, for example, www.example.com. The trailing dot is optional; Amazon Route 53 assumes that the domain name is fully qualified. This means that Amazon Route 53 treats www.example.com (without a trailing dot) and www.example.com. (with a trailing dot) as identical. If you're creating a public hosted zone, this is the name you have registered with your DNS registrar. If your domain name is registered with a registrar other than Amazon Route 53, change the name servers for your domain to the set of NameServers that create-hosted-zone returns in DelegationSet.  
- CALLER_REFERENCE: A unique string that identifies the request and that allows failed create-hosted-zone requests to be retried without the risk of executing the operation twice. You must use a unique CallerReference string every time you submit a create-hosted-zone request. CallerReference can be any unique string, for example, a date/time stamp.

2- Create S3 Bucket

    aws s3api create-bucket --bucket <BUCKET_NAME> --region <AWS_REGION> --acl private

- BUCKET_NAME: String containing the name of the bucket.  
- AWS_REGION: Region to create the bucket.  

Export KOPS_STATE_STORE variable. A valid value follows the format s3://<bucket>

    export KOPS_STATE_STORE=s3://<bucket>

3- Create and fill properties file with your values.

- Create a new yml file including all the properties listed in cluster_properties.yml.  
- Edit the new file with your values.

4- Create template with properties values.

    mkdir tmp  
    kops toolbox template --template cluster_template.yml --values cluster_default_properties.yml --values <YOUR_PROPERTIES_FILE>.yml --format-yaml=true >> ./tmp/tmp_template.yml  

5- Create Kubernetes cluster. The following command will create a new cluster with name equals to HOSTED_ZONE_NAME.

    kops create -f ./tmp/tmp_template.yml

6- Deploy cluster resources

    kops update cluster <HOSTED_ZONE_NAME> --yes

7- Create SSH Public Key in AWS

    kops create secret --name <HOSTED_ZONE_NAME> sshpublickey admin -i <SSH_PUBLIC>
    kops update cluster <HOSTED_ZONE_NAME> --yes

8- Export configuration to use this cluster as default in kubectl

    kops export kubecfg cl10.glb.southwest-aws.com

9- Delete the cluster

    kops delete cluster --name=<HOSTED_ZONE_NAME> --yes