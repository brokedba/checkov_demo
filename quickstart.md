### I. Installation
Ubuntu
```bash
apt-get update && apt-get install -y apt-utils curl
sudo apt install python3-pip

pip3.x --version   OR python3 -m pip --version
pip3.x install checkov
create an alias pip3='pip3.x'
```

**Known issues** 
`libc6: (>= 2.25) but 2.23-0ubuntu11 is to be installed`

```apt-get update && apt-get install -y apt-utils curl
  apt-utils : Depends: apt (= 1.2.35) but 1.2.31 is to be installed
  curl : Depends: libcurl3-gnutls (= 7.47.0-1ubuntu2.19) but 7.47.0-1ubuntu2.16 is to be installed
 libssl1.1 : Depends: libc6 (>= 2.25) but 2.23-0ubuntu11 is to be installed
```
-- fix
```bash
sudo apt upgrade
sudo apt update
sudo apt-get --fix-broken install
```


**Warning: pip is being invoked by an old script wrapper**
![image](https://github.com/user-attachments/assets/0a39b1d2-2685-47d6-81c0-d29a1076210c)

Fix: 
replace /usr/bin/pip content by the below
```bash
vi /usr/bin/pip
#!/usr/bin/python3
# -*- coding: utf-8 -*-
import re
import sys

from pip._internal.cli.main import main

if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw?|\.exe)?$', '', sys.argv[0])
    sys.exit(main())
```

### II. Integrating with Terraform
You can integrate Checkov with your Terraform code in the following ways:
**Scan directories**

Frameworks can also be selected or omitted for a particular scan.
ex: enable secret scan for all files
```
checkov -d . --framework secrets --enable-secret-scan-all-files
checkov -d . --skip-framework dockerfile
```
Check the exit code when running checkov -d .  with and without the --soft-fail option.
``` checkov -d . ; echo $?
checkov -d . --soft-fail ; echo $?
```
**Scan .tf files**

```bash
checkov --directory /path/to/terraform/files
```
This will scan all .tf files in the given directory.

```bash
checkov -d launch-instance --quiet --compact
```

**terraform scan results:**

```nginx
Passed checks: 10, Failed checks: 5, Skipped checks: 0

Check: CKV_AZURE_50: "Ensure Virtual Machine Extensions are not Installed"
        FAILED for resource: azurerm_linux_virtual_machine.terravm
        File: /compute.tf:38-87
Check: CKV_AZURE_160: "Ensure that HTTP (port 80) access is restricted from the internet"
        FAILED for resource: azurerm_network_security_group.terra_nsg
        File: /vnet.tf:36-70
Check: CKV_AZURE_9: "Ensure that RDP access is restricted from the internet"
        FAILED for resource: azurerm_network_security_group.terra_nsg
        File: /vnet.tf:36-70
Check: CKV_AZURE_10: "Ensure that SSH access is restricted from the internet"
        FAILED for resource: azurerm_network_security_group.terra_nsg
        File: /vnet.tf:36-70
Check: CKV_AZURE_119: "Ensure that Network Interfaces don't use public IPs"
        FAILED for resource: azurerm_network_interface.Terranic
        File: /compute.tf:10-21
```

**Scan Terraform plan files**

You can generate a Terraform plan and scan the JSON output:
```
terraform plan -out=tfplan.json
terraform show -json tfplan.json > tfplan.json
checkov -f tfplan.json
```
Note: terraform show output file tf.json will be a single line. For that reason all findings will be reported line number 0 by Checkov
```bash
check: CKV_AWS_21: "Ensure all data stored in the S3 bucket have versioning enabled"
	FAILED for resource: aws_s3_bucket.customer
	File: /tf/tf.json:0-0
	Guide: https://docs.prismacloud.io/en/enterprise-edition/policy-reference/aws-policies/s3-policies/s3-16-enable-versioning
```
If you have installed jq you can convert json file into multiple lines with the following command:
```bash
 terraform show -json tf.plan | jq '.' > tf.json
```
Scan result would be much user friendly.
```
checkov -f tf.json
Check: CKV_AWS_21: "Ensure all data stored in the S3 bucket have versioning enabled"
	FAILED for resource: aws_s3_bucket.customer
	File: /tf/tf1.json:224-268
	Guide: https://docs.prismacloud.io/en/enterprise-edition/policy-reference/aws-policies/s3-policies/s3-16-enable-versioning

		225 |               "values": {
		226 |                 "acceleration_status": "",
		227 |                 "acl": "private",
		228 |                 "arn": "arn:aws:s3:::mybucket",
```

**Specify the repo root**

To enrich the output with the appropriate file path, line numbers, and codeblock of the resource(s), specify the repo root path used to generate the plan file using --repo-root-for-plan-enrichment:
```bash
checkov -f tfplan.json --repo-root-for-plan-enrichment /path/to/terraform/files
```
This will enrich the output with file paths and line numbers.

**Use Docker**

You can also run Checkov in a Docker container:
```bash
docker run --volume /path/to/terraform/files:/tf bridgecrew/checkov --directory /tf
```
**Sample Output**

The output contains information about failed and passed checks, including file paths and code blocks:
```bash
Check: CKV_AWS_21: "Ensure all data stored in the S3 bucket have versioning enabled"
   FAILED for resource: aws_s3_bucket.customer
   File: /tf/tf.json:0-0 
   ... 
Check: "Ensure all data stored in the S3 bucket is securely encrypted at rest"
   PASSED for resource: aws_s3_bucket.template_bucket
```
## **Custom Policies**
Checkov supports the creation of Custom Policies for users to customize their own policy and configuration checks. Custom policies can be written in YAML (recommended) or python and applied with the --external-checks-dir or --external-checks-git flags.
Let's create a custom policy to check for local-exec and remote-exec Provisioners being used in Terraform resource definitons. (Follow link to learn more about provisioners and why it is a good idea to check for them).
```yaml
metadata:
 name: "Terraform contains local-exec and/or remote-exec provisioner"
 id: "CKV2_TF_1"
 category: "GENERAL_SECURITY"
definition:
 and:
  - cond_type: "attribute"
    resource_types: all 
    attribute: "provisioner/local-exec"
    operator: "not_exists"
  - cond_type: "attribute"
    resource_types: all
    attribute: "provisioner/remote-exec"
    operator: "not_exists"
```
Add the above code to a new file within a new direcotry.
```bash mkdir custom-checks/
vim custom-checks/check.yaml
```
Save the file. Then run checkov with the --external-checks-dir to test the custom policy.
```bash
checkov -f simple_ec2.tf --external-checks-dir custom-checks
```
## **Baselines**

Leverage Checkov’s new baseline feature, which enables you to set a baseline for a directory — not an individual file — so that future runs skip all existing misconfigurations.
To use the baseline feature, first use
```bash
checkov -d path/to/directory --create-baseline
```
- To set a baseline file .checkov.baseline in the scanned directory. 
- For later runs, use 
```bash
checkov -d path/to/directory --baseline path/to/directory/.checkov.baseline 
```
- So that you’re only checking for newly identified misconfigurations.

##  Addressing Complexity of declarative IAC with mappings: 
 With Chekov's graph-based mapping, those complex dependencies can be fully mapped out and understood prior to the development process.
It will go and check hidden dependencies and spot misconfigurations (instance can be exposed )

This drastically improves the accuracy of scans and helps to provide better risk prioritization while reducing false positives
![image](https://github.com/user-attachments/assets/faf5771d-515b-44de-88f3-708392023563)

