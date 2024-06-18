---
title: S3, IAM, Serverless and Access Denied
tags:
  - s3
  - iam
  - serverless
  - typescript
  - aws
url: s3-iam-serverless-access-denied.html
categories:
 - aws
 - typescript
date: 2023-10-17
---

## s3 IAM woes when using serverless
Recently I was attempting to deploy a new lambda server over some existing aws architecture, the s3 buckets already existed as well as lots of permissioning. This should be easy? Arguably it was easy and probably would have been easier if I already knew AWS/IAM/etc - though I don't so pain was in store for me.

In general my [serverless](https://www.serverless.com/) configuration for the aws/iam permissions looked like this;

```
provider:
  name: aws
  runtime: nodejs18.x
  region: us-gov-west-1
  endpointType: REGIONAL
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - s3:GetObject
            - s3:ListObject
          Resource:
            - 'arn:aws-us-gov:s3:::bucket-name/*'
```

Which I thought would be ok, as this was the generic snippet of code in which I was attempting to list a buckets objects and get tagging information on them;

```
    const client = new S3Client({
      region,
    });

    const command = new ListObjectsV2Command({
      Bucket: 'bucket-name',
      Delimiter: '/',
      Prefix: 'some/folder/struct/inside/bucket/,
    });

    const { Contents } = await client.send(command);

...

        const command = new GetObjectTaggingCommand({
          Bucket: bucket,
          Key: object.Key,
        });

        const { TagSet } = await client.send(command).then();
```

This worked fine locally when running `serverless --offline` as the `S3Client` would grab my local AWS credentials which had (seemingly) the same permissions. This was a sort of, false reality for me, as it took me a moment to even realize that this was happening. After figuring it out, of course it was using my local credentials which had already been over permissioned. Though when `serverless` bundled and pushed everything utilizing `CloudFormation`, it was not provisioning the correct permissions for the lambda. This lead in the `client.send` throwing an `Access Denied` error. So helpful, just tell me what permission I need already.

### Finding the permissions
Per some Googling and reading of the documentation, the command [ListObjectsV2Command](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/classes/listobjectsv2command.html);

```
To use this operation, you must have READ access to the bucket.

To use this action in an Identity and Access Management (IAM) policy, you must have permission to perform the s3:ListBucket action. The bucket owner has this permission by default and can grant this permission to others. For more information about permissions, see Permissions Related to Bucket Subresource Operations and Managing Access Permissions to Your Amazon S3 Resources in the Amazon S3 User Guide.
```

So we will need `READ` access, meaing we should be able to `s3:ListBucket`. I assumed `READ` meant `s3:GetObject`, so I think we got it, it looks like we already had this?

For [GetObjectTaggingCommand](https://docs.aws.amazon.com/AmazonS3/latest/API/API_GetObjectTagging.html) the documentation states;

```
To use this operation, you must have permission to perform the s3:GetObjectTagging action. By default, the GET action returns information about current version of an object. For a versioned bucket, you can have multiple versions of an object in your bucket. To retrieve tags of any other version, use the versionId query parameter. You also need permission for the s3:GetObjectVersionTagging action.
```

So `s3:GetObjectTagging` is likely enough? Even though we haven't gotten to this line yet in our execution.

### Over-permissioning for Testing
Annoyingly I tried many permutations and couldn't seem to get anything right, so I tried just over permissioning it to see if this would simply allow it;

```
         - Effect: Allow
           Action:
             - s3:*
           Resource:
             - 'arn:aws-us-gov:s3:::*'
```

This above one worked - as excepted. At least at this point we know it is possible and we can try to trim down the permissions.

### Minimizing permissions
Re-(re-re-re-)reading AWS docs, I tried to break out the permissions into different roles to avoid over permissioning things. Something I had missed originally was that you want to literally list a bucket (root directory) with no glob when specifying `s3:ListBucket` which is required for `ListObjectsV2Command` so this should work;

```
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - s3:ListObject
          Resource:
            - 'arn:aws-us-gov:s3:::bucket-name/*'
        - Effect: Allow
          Action:
            - s3:ListBucket
          Resource:
            - 'arn:aws-us-gov:s3:::bucket-name'
```

This allowed our `ListObjectsV2Command` to work! Now we got an error on the `GetObjectTaggingCommand`. Digging back into this side, I added the `s3:GetObject` and `s3:GetObjectTagging` permissions to the globbed bucket;

```
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - s3:GetObject
            - s3:ListObject
            - s3:GetObjectTagging
          Resource:
            - 'arn:aws-us-gov:s3:::bucket-name/*'
        - Effect: Allow
          Action:
            - s3:ListBucket
          Resource:
            - 'arn:aws-us-gov:s3:::bucket-name'
```

Excellent, this worked as well. Now to minimize the scope of what this lambda is allowed to access. 

## Final minimization of scope
Since I originally didn't set up these buckets, there actually had been a decent amount of files stored here and while this lambda doesn't try to interact with it, I just don't want to let it even have permissions to look at those files. Per the example, I abstracted it to `some/folder/struct/inside/bucket/` and let's pretend I also want to access `some/other/folder`;

```
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - s3:GetObject
            - s3:ListObject
            - s3:GetObjectTagging
          Resource:
            - 'arn:aws-us-gov:s3:::bucket-name/some/other/folder/*'
            - 'arn:aws-us-gov:s3:::bucket-name/some/folder/struct/inside/bucket/*'
        - Effect: Allow
          Action:
            - s3:ListBucket
          Resource:
            - 'arn:aws-us-gov:s3:::bucket-name'
```

Bingo! Once you see it all layed out and have done it once, it all makes more sense and seems simple. Though, when you're grinding for a day or two on the issue and waiting for the lambda to compile/upload/etc it can be quiet annoying...
