# AWS Lambda and GitHub Actions Integration

Integrate GitHub and AWS Lambda to auto deploy an existing function on code changes.

### Usage
- GitHub > Settings > Secrets >  
  - New secret > Name: `AWS_ACCESS_KEY_ID`, Value: ... > Add secret  
  - New secret > Name: `AWS_SECRET_ACCESS_KEY`, Value: ... > Add secret
  - New secret > Name: `AWS_REGION`, Value: ... > Add secret
  
- GitHub > Actions > New workflow > set up a workflow yourself > ...
  - modify the parameters below and then copy & paste in the editor

- Lambda function will be updated every time the Python code is changed
- deployments can also be automated by scheduling to run the workflow at specific times using `schedule` and `cron` commands

### Parameters
**`files`** - folder containing the function code and any dependencies  
**`dest`** - filename  
**`--function-name`** - name of the existing Lambda function  
**`--zip-file`** - zip filename  
if using Windows, replace `\` with `^`

```
name: update-lambda

on:
  push:
    branches:
    - master
    paths:
    - '**.py'
  pull_request:
    branches:
    - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout source code
      uses: actions/checkout@v2

    - name: Zip folder
      uses: papeloto/action-zip@v1
      with:
        files: FOLDER_NAME
        dest: FILENAME.zip

    - name: Set up Python 3.8
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
          --function-name  FUNCTION_NAME \
          --zip-file fileb://FILENAME.zip
```
