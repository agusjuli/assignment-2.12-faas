# assignment-2.12-faas

1. What is the purpose of the execution role on the Lambda function?

The execution role assigned to the AWS Lambda function is an IAM role that grants the function permissions to interact with other AWS services. Its primary purposes include:
a). Accessing the S3 Bucket – The role must have permissions (such as s3:GetObject) to read the newly created files and process them.
b). Writing Logs to CloudWatch – The role typically includes permissions (logs:CreateLogGroup, logs:CreateLogStream, logs:PutLogEvents) to send
    logs to Amazon CloudWatch, enabling debugging and monitoring.
c). Interacting with Other AWS Services – If the function interacts with other AWS services (such as DynamoDB, SNS, or Step Functions), the role 
    must include appropriate permissions.
d). Ensuring Least Privilege – The execution role should follow the principle of least privilege, granting only the necessary permissions to 
    perform the required tasks.



2. What is the purpose of the resource-based policy on the Lambda function?

A resource-based policy on an AWS Lambda function specifies who (which AWS principals or services) can invoke the function and what actions they are allowed to perform. It is particularly useful when external AWS services or accounts need to invoke the function. Its primary purposes include:
a). Allowing S3 to Invoke the Lambda Function – When an S3 bucket triggers the Lambda function on object creation, the resource-based policy must 
    include permissions that allow S3 to invoke the function (lambda:InvokeFunction).
b). Enabling Cross-Account Access – If another AWS account needs to invoke the Lambda function, a resource-based policy can explicitly grant 
    access to that account.
c). Securing Function Access – The policy helps define which AWS services, IAM roles, or external accounts can call the function, preventing 
    unauthorized access.
d). Overriding or Supplementing IAM Policies – While the Lambda execution role controls what the function can do (e.g., access S3 or write logs), 
    the resource-based policy controls who can invoke the function.



3.A. If the function needs to upload a file into an S3 bucket, What is the needed update on the execution role?
   
If AWS Lambda function needs to upload a file into an S3 bucket, we must update the execution role to grant it the necessary permissions. This is done by modifying the IAM policy attached to the role.
a). Update the IAM Execution Role
    We need to add S3 write permissions to the IAM role that the Lambda function assumes when it runs.

    IAM Policy Update for S3 Upload:
   
{
    "Effect": "Allow",
    "Action": [
        "s3:PutObject"
    ],
    "Resource": "arn:aws:s3:::your-destination-bucket/*"
}

Explanation:
s3:PutObject → Allows the Lambda function to upload (write) objects into the specified S3 bucket.
arn:aws:s3:::your-destination-bucket/* → Grants access to all objects (*) within the bucket your-destination-bucket.

b). (Optional) Additional Permissions
    Depending on the requirements, we might also need to add:

s3:PutObjectAcl → If we need to set object permissions (ACLs).
s3:ListBucket → If the function needs to check if the bucket exists or list objects.
s3:GetBucketLocation → If the function needs to verify the bucket's region.

Example IAM Policy with Additional Permissions

{
    "Effect": "Allow",
    "Action": [
        "s3:PutObject",
        "s3:PutObjectAcl",
        "s3:ListBucket",
        "s3:GetBucketLocation"
    ],
    "Resource": [
        "arn:aws:s3:::your-destination-bucket",
        "arn:aws:s3:::your-destination-bucket/*"
    ]
}

c). Applying the Policy

Go to AWS IAM Console → Roles.
Find the execution role assigned to your Lambda function.
Attach or update the IAM policy with the above permissions.
Save and test the Lambda function to ensure it can upload files.

Summary:
Modify the execution role to include s3:PutObject permission.
Specify the correct S3 bucket ARN (arn:aws:s3:::your-destination-bucket/*).
Optionally add more permissions if needed (like ACLs or bucket listing).



3.B. What is the new resource-based policy that needs to be added (if any)?

In most cases, we do not need to add a resource-based policy on the Lambda function just to upload a file to an S3 bucket. Instead, we only need to update the IAM execution role of the function, as explained earlier.
However, we might need a resource-based policy in the following situations:
A. If Another AWS Account Owns the S3 Bucket
If the Lambda function runs in Account A and the S3 bucket is in Account B, then Account B must allow Account A's Lambda function to write to the bucket. This is done by adding a bucket policy to the S3 bucket in Account B:

Bucket Policy in Account B:

{
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::account-A-id:role/your-lambda-execution-role"
    },
    "Action": "s3:PutObject",
    "Resource": "arn:aws:s3:::destination-bucket-in-account-b/*"
}

This allows the Lambda function from Account A to upload objects to Account B's S3 bucket.

B. If Another AWS Service (Like EventBridge) Needs to Invoke the Lambda Function
If we plan to trigger the Lambda function using EventBridge (or another AWS service outside S3), we might need to add a resource-based policy to the Lambda function:

Example Resource-Based Policy for EventBridge:

{
    "Effect": "Allow",
    "Principal": {
        "Service": "events.amazonaws.com"
    },
    "Action": "lambda:InvokeFunction",
    "Resource": "arn:aws:lambda:your-region:your-account-id:function:your-function-name"
}

This allows AWS EventBridge to invoke the Lambda function.

Summary:
If the Lambda function only uploads files to S3, no resource-based policy is needed—just update the IAM execution role.
If the S3 bucket is in another AWS account, update the bucket policy in the destination account.
If another AWS service (like EventBridge) invokes the function, update the Lambda resource-based policy to allow invocation.
