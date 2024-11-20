# checkov_demo
demo for hashicorp User group workshop 2024

# PRE-COMMIT

1. Switch to the main branch (if you are not already on it):

```
git checkout main
```
2. Create the .gitattributes file:
```
echo ".pre-commit-config.yaml merge=preco" > .gitattributes
```
3. Add and commit the file:

```
git add .gitattributes
git commit -m "Add .gitattributes to keep .pre-commit-config.yaml during merges"
git push origin main
```
After completing these steps, the .gitattributes file will be present in the main branch. When you merge a feature branch into the main branch in the future, Git will use the merge strategy specified in .gitattributes to keep the version of .pre-commit-config.yaml from the main branch and ignore changes from the feature branch.

# Setting Up a `preco` Branch with Pre-Commit Configuration

## Step-by-Step Guide

### 1. Create and Switch to the New Branch `preco`
Open your terminal and run:
```
git checkout -b preco
```

2. Create the .pre-commit-config.yaml File
Create the .pre-commit-config.yaml file with the following content:

```yaml  

repos:
- repo: https://github.com/antonbabenko/pre-commit-terraform
  rev: v1.89.1 # Get the latest from: https://github.com/antonbabenko/pre-commit-terraform/releases
  hooks:
    - id: terraform_checkov
      args:
        - --args=--quiet
        - --args=--skip-check CKV2_AWS_8
        - --args=--framework terraform
        - --args=--compact
        - --args=--file code/deployment_ec2.tf
    - id: terraform_tflint
      args:
        - --args=--module
        - --args=--enable-rule=terraform_documented_variables

- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v4.5.0
  hooks:
    - id: trailing-whitespace
    - id: end-of-file-fixer
    - id: check-yaml
    - id: debug-statements
    - id: double-quote-string-fixer
    - id: name-tests-test
    - id: requirements-txt-fixer
```
3. Add the File to the Staging Area
Run the following command:
```
git add .pre-commit-config.yaml
```
4. Commit the Changes

Commit the changes with a descriptive message:
```
 git commit -m "Add pre-commit configuration for Terraform and general hooks"
```
5. Push the New Branch to the Remote Repository
Finally, push the new branch to your remote repository:
```
git push origin preco
```
By following these steps, you will have a new branch named preco with the specified .pre-commit-config.yaml file added to it.
