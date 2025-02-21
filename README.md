# aws-cdk-dynamodb-seeder [![Mentioned in Awesome CDK](https://awesome.re/mentioned-badge.svg)](https://github.com/eladb/awesome-cdk)

![build](https://github.com/elegantdevelopment/aws-cdk-dynamodb-seeder/workflows/build/badge.svg)
[![codecov](https://codecov.io/gh/elegantdevelopment/aws-cdk-dynamodb-seeder/branch/master/graph/badge.svg)](https://codecov.io/gh/elegantdevelopment/aws-cdk-dynamodb-seeder)
[![dependencies Status](https://david-dm.org/elegantdevelopment/aws-cdk-dynamodb-seeder/status.svg)](https://david-dm.org/elegantdevelopment/aws-cdk-dynamodb-seeder)
[![npm](https://img.shields.io/npm/dt/aws-cdk-dynamodb-seeder)](https://www.npmjs.com/package/aws-cdk-dynamodb-seeder)

[![npm version](https://badge.fury.io/js/aws-cdk-dynamodb-seeder.svg)](https://badge.fury.io/js/aws-cdk-dynamodb-seeder)
[![NuGet version](https://badge.fury.io/nu/ElegantDevelopment.AWSCDKDynamoDBSeeder.svg)](https://badge.fury.io/nu/ElegantDevelopment.AWSCDKDynamoDBSeeder)
[![PyPI version](https://badge.fury.io/py/aws-cdk-dynamodb-seeder.svg)](https://badge.fury.io/py/aws-cdk-dynamodb-seeder)
[![Maven Central](https://img.shields.io/maven-central/v/io.github.elegantdevelopment/AWSCDKDynamoDBSeeder?color=brightgreen)](https://repo1.maven.org/maven2/io/github/elegantdevelopment/AWSCDKDynamoDBSeeder/)

A simple CDK JSON seeder for DynamoDB

Forked by Logitech in May 2021

Every significant change to this repo should result in a new tag, most easily created with:  
```shell
npx standard-version
```

## Why this package

Glad you asked!

Using [AWS CDK] for automating infrastructure deployments is an amazing way of integrating the development and operations into one process and one codebase.

However, building dev or test environments that come pre-populated with data can be tricky, especially when using [Amazon DynamoDB].

## How do I use it

Install using your favourite package manager:

```sh
yarn add aws-cdk-dynamodb-seeder
```

### Example TypeScript usage

```ts
import { Seeder } from 'aws-cdk-dynamodb-seeder';
...
const myTable = new Table(stack, "MyTable", {
    tableName: "MyTable",
    partitionKey: { name: "Id", type: AttributeType.STRING },
});
...
new Seeder(stack, "MySeeder", {
    table: myTable,
    setup: require("./items-to-put.json"),
    teardown: require("./keys-to-delete.json"),
    refreshOnUpdate: true  // runs setup and teardown on every update, default false
});
```

For a more in-depth example, see: [elegantdevelopment/aws-cdk-dynamodb-seeder-examples](https://github.com/elegantdevelopment/aws-cdk-dynamodb-seeder-examples).

### Importing seed data

Data passed into `setup` ("Items" to put) or `teardown` ("Keys" to delete) should be an `array` of objects (that are, in turn, representations of `string` to [AttributeValue] maps).

* `setup` elements should use the format of `params.Item` from [AWS.DynamoDB.DocumentClient.put()]
* `teardown` elements should use the format of `params.Key` from [AWS.DynamoDB.DocumentClient.delete()]

## Versioning

We will *attempt* to align the major and minor version of this package with [AWS CDK], but always check our release descriptions for compatibility.

We currently support [![GitHub package.json dependency version (prod)](https://img.shields.io/github/package-json/dependency-version/elegantdevelopment/aws-cdk-dynamodb-seeder-examples/@aws-cdk/core)](https://github.com/aws/aws-cdk)

## Internals

Behind the scenes we use an [AwsCustomResource] as a representation of the related table's seed state. The custom resource's event handlers invoke a [Function] to perform setup and/or teardown actions.

### Deploying a stack

On deployment, we write copies of your seed data locally and use a [BucketDeployment] to write it to an S3 [Bucket].

We then create the handler function and custom resource to field seed requests (the `onCreate` event will immediate fire as the stack deploys, reading the data from the bucket and seeding the table using [AWS.DynamoDB.DocumentClient]).

### Updating a stack

On a stack update, the `onUpdate` handler is triggered when `refreshOnUpdate` is `true`.

This will run [AWS.DynamoDB.DocumentClient.delete()] on every teardown "Key" followed by [AWS.DynamoDB.DocumentClient.put()] on every setup "Item".

### Destroying a stack

When the stack is destroyed, the event handler's `onDelete` function will be invoked, providing `teardown` is set.

This simply runs [AWS.DynamoDB.DocumentClient.delete()] on every teardown "Key" before destroying the `Seeder`'s resources.

[aws cdk]: https://aws.amazon.com/cdk
[amazon dynamodb]: https://aws.amazon.com/dynamodb

[AttributeValue]: https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_AttributeValue.html
[AWS.DynamoDB.DocumentClient]: https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html
[AWS.DynamoDB.DocumentClient.put()]: https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#put-property
[AWS.DynamoDB.DocumentClient.delete()]: https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#delete-property

<!-- Internals -->
[AwsCustomResource]: https://docs.aws.amazon.com/cdk/api/latest/typescript/api/custom-resources/awscustomresource.html
[Function]: https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-lambda/function.html#aws_lambda_Function
[Bucket]: https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3/bucket.html#aws_s3_Bucket
[BucketDeployment]: https://docs.aws.amazon.com/cdk/api/latest/typescript/api/aws-s3-deployment/bucketdeployment.html#aws_s3_deployment_BucketDeployment
