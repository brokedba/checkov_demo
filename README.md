### Compliance as Code using Checkov
---
### Demo 1 🚀
**Directories:**
![image](https://github.com/user-attachments/assets/96e18200-78c2-41fc-95e8-794a23b63189)

A. [terragoat](https://github.com/bridgecrewio/terragoat): Vulnerable by Design" Terraform repository (AWS, GCP, AZURE, OCI, ALICLOUD)
<p align="justified"> <img src= "https://github.com/user-attachments/assets/52dbb932-5d85-4f16-a10b-30ca36d3b158" width="320" height="250" /> </p>

B. Brokedba [terraform-examples](https://github.com/brokedba/terraform-examples)
  <p align="justified"> <img src= "https://github.com/user-attachments/assets/d00e35c5-2036-43b7-86b9-061a8780691c" width="350" height="300" /> </p>
 
1. Checkov frameworks checks [use your favorite framework)
```bash
// show policies per framework 
checkov -l --framework gitlab_ci
```
Example:
```nginx
// TERRAGOAT Directory
git clone https://github.com/bridgecrewio/terragoat.git
cd terragoat/azure
`alicloud`  
`aws`  
`azure`  
`gcp`  
`oracle`
$ checkov -d oracle --compact --quiet
$ checkov -d alicloud --compact --quiet
$ checkov -d azure/  --compact --quiet | grep Passed 

# -- Compare with other tools 

terrascan scan -i terraform -d alicloud/ -o human -l info
trivy config --misconfig-scanners=terraform oracle/ 
tfsec oracle/ --concise-output -m CRITICAL
trivy config --misconfig-scanners=terraform azure/ | grep Tests: 
 
Results :
`Trivy` => Total Tests: 101 , Total Successes: 15, 
            Total Failures: 86, Total Exceptions: 0
 
`Checov` => Total Checks: 231, Passed checks: 58,
             Failed checks: 173, Skipped checks: 0
```
**Available frameworks:**

`ansible, argo_workflows, arm, azure_pipelines, bicep, bitbucket_pipelines, cdk, circleci_pipelines, cloudformation, dockerfile, github_configuration, 
github_actions, gitlab_configuration, gitlab_ci, bitbucket_configuration, helm, json, yaml, kubernetes, kustomize, openapi, sca_package, sca_image, secrets,
 serverless, terraform, terraform_json, terraform_plan, sast, sast_python, sast_java, sast_javascript, sast_typescript, sast_golang, 3d_policy`


- **Check /skip check Command**
```bash
 git clone https://github.com/brokedba/terraform-examples.git
 cd terraform-examples/terraform-provider-oci
 checkov -d launch-instance/ --compact --quiet
 checkov -d launch-instance/ --compact --quiet --check CKV_OCI_19
 checkov -f database-system/terraform.tfvars  --framework secrets --quiet --compact
```
- **Soft fail:** 
  - If you want your CI job to finish, Applying --soft-fail results in the scan always returning a 0 exit code.

 ```bash
     checkov -d . ; echo $?
     checkov -d launch-instance/ --compact --quiet ; echo exit code: $?
     #### Using softfail 
    checkov -d launch-instance/ --compact --quiet --soft-fail ; echo exit code:$?
```
- **Custom Policies checov-demo**
```yaml
// create a custom policy 
custom-checks/exec_provisioner_check.yaml
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
- Run checkov with defined policy
```
checkov -f exec_provisioner.tf --external-checks-dir ../custom-checks --quiet
```
- **Scan Plans**
```bash
 // Some code
terraform plan -out=tfplan.json
terraform show -json tf.plan | jq '.' > tfplan.json
checkov -f tfplan.json
```
---

### **Demo 2**
## I. Pre-commit (locally)
**1. clone My repo**
```
git clone https://github.com/brokedba/checkov_demo.git
```
**2. install and run pre-commit**
```
pip install pre-commit 
$ pre-commit --version
pre-commit 3.7.1
```
**3. Create .pre-commit-config.yaml**
```yaml
--- 
vi `.pre-commit-config.yaml`
repos:
- repo: https://github.com/antonbabenko/pre-commit-terraform
  rev: v1.89.1 # Get the latest from: https://github.com/antonbabenko/pre-commit-terraform/releases
  hooks:
    - id: terraform_checkov
      args:
        - --args=--quiet
        - --args=--skip-check CKV2_AWS_8
        - --args=--check CKV_AWS_186
        - --args=--framework terraform
        - --args=--compact
        - --args=--file code/deployment_ec2.tf
```
4. run `pre-commit install` to set up the git hook scripts
```bash
$ pre-commit install
pre-commit installed at .git/hooks/pre-commit
```
5. Change a terraform file code/deployment_ec2.tf
```bash
// vi code/deployment_ec2.tf
git add code/deployment_ec2.tf
git commit -m "make a commit to test the pre-commit on tf files"
```
**Note:** use git commit .. `--no-verify` if you want to bypass the failed precomit 
6. Run pre-commit manually 
```bash
// Run full precomit on all files
pre-commit run -a
// Run precommit
git ls-files -- '*.py' | xargs pre-commit run --files
```
## II. Pre-commit in GitHub Actions
- use [checkov action link](https://github.com/marketplace/actions/checkov-github-action)
- WORKFLOW sample fo terraform:  [checkov_workflow.yaml](https://github.com/brokedba/checkov_demo/tree/main/.github/workflows)
- Scan results:
![image](https://github.com/user-attachments/assets/7586ea96-6094-4b33-a8d9-27807674c796)
![image](https://github.com/user-attachments/assets/400bbdd8-1f8e-4253-8df1-7987a00dd00a)

-**Branch protection**
![https___files gitbook com_v0_b_gitbook-x-prod appspot com_o_spaces%2F3M2nrIbUIeC5MIWWDJfM%2Fuploads%2FmN4XGf8Z2cUTJ5Gbb9Wq%2Fimage](https://github.com/user-attachments/assets/66a95811-114a-4804-a404-b05d9af7fa73)


