# AWS SDK ZF2 Module

[![Latest Stable Version](https://poser.pugx.org/aws/aws-sdk-php-zf2/v/stable.png)](https://packagist.org/packages/aws/aws-sdk-php-zf2)
[![Total Downloads](https://poser.pugx.org/aws/aws-sdk-php-zf2/downloads.png)](https://packagist.org/packages/aws/aws-sdk-php-zf2)

## Introduction

This module provides a simple wrapper for the AWS SDK for PHP. It registers the AWS service builder as a service in the
ZF2 service manager, making it easily accessible anywhere in your application.

## Installation

Install the module using Composer into your application's vendor directory. Add the following line to your
`composer.json`. This will also install the AWS SDK for PHP.

```json
{
    "require": {
        "aws/aws-sdk-php-zf2": "1.0.*"
    }
}
```

## Configuration

Enable the module in your `application.config.php` file.

```php
return array(
    'modules' => array(
        'Aws'
    )
);
```

Copy and paste the `aws.local.php.dist` file to your `config/autoload` folder and customize it with your credentials and
other configuration settings. Make sure to remove `.dist` from your file. Your `aws.local.php` might look something like
the following:

```php
<?php

return array(
    'aws' => array(
        'key'    => '<your-aws-access-key-id>',
        'secret' => '<your-aws-secret-access-key>',
        'region' => 'us-west-2'
    )
);
```

> NOTE: If you are using [IAM Instance Profile
credentials](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/UsingIAM.html#UsingIAMrolesWithAmazonEC2Instances)
(also referred to as IAM Roles for instances), you can omit your `key` and `secret` parameters since they will be
fetched from the Amazon EC2 instance automatically.

## Usage

You can get the AWS service builder object from anywhere that the ZF2 service locator is available (e.g. controller
classes). The following example instantiates an Amazon DynamoDB client and creates a table in DynamoDB.

```php
public function indexAction()
{
    $aws    = $this->getServiceLocator()->get('aws');
    $client = $aws->get('dynamodb');

    $table = 'posts';

    // Create a "posts" table
    $result = $client->createTable(array(
        'TableName' => $table,
        'KeySchema' => array(
            'HashKeyElement' => array(
                'AttributeName' => 'slug',
                'AttributeType' => 'S'
            )
        ),
        'ProvisionedThroughput' => array(
            'ReadCapacityUnits'  => 10,
            'WriteCapacityUnits' => 5
        )
    ));

    // Wait until the table is created and active
    $client->waitUntilTableExists(array('TableName' => $table));

    echo "The {$table} table has been created.\n";
}
```

## View Helpers

Starting from version 1.0.2, the AWS SDK ZF2 Module now provides two view helpers to generate links for Amazon S3 and
Amazon CloudFront resources.

> **Note:** Both of the view helpers generate URLs with an HTTPS scheme by default. This is ideal for security, but
please keep in mind that Amazon CloudFront charges more for HTTPS requests. You can turn SSL off by calling the
`setUseSsl` method on both helpers.

### S3Link View Helper

To create a S3 link in your view:

```php
<?php echo $this->s3Link('my-object', 'my-bucket');
```

The default bucket can be set globally by using the `setDefaultBucket` method:

```php
<?php
    $this->s3Link->setDefaultBucket('my-bucket');
    echo $this->s3Link('my-object');
```

You can also create signed URLs for private content by passing a third argument which is the expiration date:

```php
<?php echo $this->s3Link('my-object', 'my-bucket', '+10 minutes');
```

### CloudFrontLink View Helper

To create CloudFront link in your view:

```php
<?php echo $this->cloudFrontLink('my-object', 'my-domain');
```

The default domain can be set globally by using the `setDefaultDomain` method:

```php
<?php
    $this->cloudFrontLink->setDefaultDomain('my-domain');
    echo $this->cloudFrontLink('my-object');
```

You can also create signed URLs for private content by passing a third argument which is the expiration date:

```php
<?php echo $this->cloudFrontLink('my-object', 'my-bucket', time() + 60);
```

## Filters

Starting from version 1.0.3, the AWS SDK ZF2 module provides a simple file filter that allow to directly upload to S3.
The `S3RenameUpload` extends `RenameUpload` class, so please refer to [its
documentation](http://framework.zend.com/manual/2.2/en/modules/zend.filter.file.rename-upload.html#zend-filter-file-rename-upload)
for available options.

This filter only adds one option to set the bucket name (through the `setBucket` method, or by passing a `bucket` key
to the filter's `setOptions` method).

```php
$request = new Request();
$files   = $request->getFiles();
// e.g., $files['my-upload']['tmp_name'] === '/tmp/php5Wx0aJ'
// e.g., $files['my-upload']['name'] === 'profile-picture.jpg'

// Fetch the filter from the Filter Plugin Manager to automatically handle dependencies
$filter = $serviceLocator->get('FilterManager')->get('S3RenameUpload');

$filter->setOptions(array(
    'bucket'    => 'my-bucket',
    'target'    => 'users/5/profile-picture.jpg',
    'overwrite' => true
));

$filter->filter($files['my-upload']);

// File has been renamed and moved to 'my-bucket' bucket, inside the 'users/5' path
```

## Related Modules

The following are some ZF2 modules that use the AWS SDK for PHP by including this module:

* [SlmMail](https://github.com/juriansluiman/SlmMail) - Module that allow to send emails with various providers
  (including Amazon SES)
* [SlmQueueSqs](https://github.com/juriansluiman/SlmQueueSqs) – Module that simplifies the use of Amazon SQS

## Links

* [AWS SDK for PHP on Github](http://github.com/aws/aws-sdk-php)
* [AWS SDK for PHP website](http://aws.amazon.com/sdkforphp/)
* [AWS on Packagist](https://packagist.org/packages/aws)
* [License](http://aws.amazon.com/apache2.0/)
* [ZF2 website](http://framework.zend.com)
