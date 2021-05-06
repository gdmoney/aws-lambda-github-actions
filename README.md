# AWS Lambda and GitHub Actions Integration

Integrates GitHub and AWS Lambda to automate updating and redeployment an **existing** Python based function on code changes.

### Usage
In the example below, the workflow will run and update the AWS Lambda function every time `lambda_function.py`, `urls.py`, or `update-lambda.yml` files are modified and changes are pushed to the `master` branch. It can also be triggerred manually with the `workflow_dispatch` event.

- GitHub > Settings > Secrets >  
  - New secret > Name: `AWS_ACCESS_KEY_ID`, Value: `...` > Add secret  
  - New secret > Name: `AWS_SECRET_ACCESS_KEY`, Value: `...` > Add secret
  - New secret > Name: `AWS_REGION`, Value: `...` > Add secret
  
- GitHub > Actions > New workflow > set up a workflow yourself > *modify the parameters below and copy & paste in the editor*

### Parameters
- **`files`** - folder containing the function code and any dependencies  
- **`dest`** - filename  
- **`--function-name`** - name of the existing Lambda function  
- **`--zip-file`** - zip filename

```
name: update-lambda

on:
  push:
    branches:
    - master
    paths:
    - 'AWS/lambda_function.py'
    - 'AWS/urls.py'
    - '.github/workflows/update-lambda.yml'
  workflow_dispatch:
  
jobs:
  
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Check out repo
      uses: actions/checkout@v2

    - name: Zip folder
      uses: papeloto/action-zip@v1
      with:
        files: AWS/ urls.py
        recursive: true
        dest: lambda.zip

    - name: Setup Python 3.8
      uses: actions/setup-python@v2
      with:
        python-version: 3.8

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install awscli
        
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    - name: Run
      run: |
        aws lambda update-function-code \
          --function-name  product-availability-checker \
          --zip-file fileb://lambda.zip
```
