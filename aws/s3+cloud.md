# AWS S3 + CloudFront (Most Popular)
## Step1:
create s3 bucket 
block all - uncheck , bucket versiong - enable , create bucket . 
open bucket - *properties* -bucket versinng - enable , static web hosting - enable - index.html - save
              *permissions*- public access -off , 
              *objects*    - upload , car website - open , carwebsite main - open , drag and drop azure and github , 
                            open car website and drang and drop css , images , js and all except README.md
                            
## step2: open cloud front from AWS
create distirbution
pay-as-you-go- next , name - next , 
origin - browser s3 - select s3 bucket - next - next - next- create .

open distirubution - create invalidation  - obejeck path - /* , create .
open distirubution - origin - select s3 bucket , origin access - occ and copy policy 

## step3
open s3 bucket - permissions - bucket policy updated 
open objects in top - index.html - click on url (output will come )

if not go to s3 buket and permissions 
edit bucket policy

`{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::demo-training-s3new/*"
        }
    ]
}`

change bucket ARN - arn:aws:s3:::demo-training-s3new with top ARN (our ARN ) code 

completed 




                         
