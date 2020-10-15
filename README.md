# aws-lambda-github-actions

Integrate GitHub and AWS Lambda to audo deploy on code changes.

### Usage
- GitHub > Settings > Secrets >  
  - New secret > Name: `AWS_ACCESS_KEY_ID`, Value ... > Add secret  
  - New secret > Name: `AWS_SECRET_ACCESS_KEY`, Value ... > Add secret
  - New secret > Name: `AWS_REGION`, Value ... > Choose a region
  
- GitHub > Actions > New workflow > set up a workflow yourself > ...

### Parameters
`files` - folder containing the code and any required libraries
`dest` - filename
`--function-name` - name of the existing Lambda function
`--zip-file` - zip filename

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
        files: AWS
        dest: lambda.zip

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
          --function-name  product-availability-checker \
          --zip-file fileb://lambda.zip
```
