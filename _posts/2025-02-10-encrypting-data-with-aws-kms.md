---
layout: post
title: "Encrypting data with AWS KMS"
date: 2025-02-10 12:00:00 +0100
categories: [tech, devops, aws, security, software_engineering]
tags: [tech, devops, aws, security, software_engineering]
---



## Introduction

A few months ago, in my previous role, an engineer posted a question in Slack about using AWS KMS to encrypt data. They’re a bright and thoughtful chap, and their product manager had set a requirement to ensure the data was encrypted at rest at the data level. I was really pleased to see this being discussed as a non-functional requirement, especially given the sensitivity of the data. This kind of thinking is crucial when considering defence-in-depth.

However, when it came to implementation, the engineer found that AWS KMS didn’t seem to be working as expected—encryption and decryption were taking longer than anticipated. After a quick refresher on envelope encryption with AWS Encryption SDK Guide, I joined the discussion. It struck me that this exact question had come up in previous roles too. To ensure I—and others—have something to refer back to in the future, I decided to document it properly.

## What is AWS KMS and the AWS Encryption SDK?

Straight from the [horse's mouth](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html) - AWS KMS is a AWS managed service that makes it easy for you to create and control the encryption keys that are used to encrypt your data. The AWS KMS you create are protected by [FIPS-140-3](https://en.wikipedia.org/wiki/FIPS_140-3) Security Level 3 hardware security modules (HSM), which is the cryptographic standard approved by the US Government.

When encrypting data, it’s essential to secure the encryption key used to generate the ciphertext. If that key is encrypted, the key protecting it must also be safeguarded, and so on. Eventually, this chain leads back to the root key, which never leaves AWS KMS. This is critical because if the root key were ever compromised, all other encryption keys derived from it would be compromised as well.

![AWS KMS High Level](/assets/img/media/aws-kms-blogpost-kms-overview.gif)

Lastly, we should quickly talk about KMS key types, of which there are three:

* __Customer Managed Keys__ - This key type is the de facto choice for customers that want full control of the usage and lifecycle policy. This means that the customer is responsible for setting rotation, deletion and regional location of keys. Auditing/logging is available for CMK's via AWS CloudTrail or Event Data Store. You pay a monthly fee for the existence of keys (pro-rated hourly), and also the customer is charged for key usage.
* __AWS Managed Keys__ - This key type is a KMS key that exists in your account, but can only be used under certain circumstances. Specifically, it can only be used in the context of the AWS service you’re operating in and it can only be used by principals within the account that the key exists. AWS managed keys are a legacy key type that is no longer being created for new AWS services as of 2021. You get the same auditing/logging capabilities as customer managed keys, but a slight difference in that there is no monthly fee; but the caller is charged for API usage on these keys. The main key difference between AWS managed keys and customer managed keys, is that AWS Managed keys manage rotation, deletion and regional location etc. This significantly reduces the overhead of managing keys, so you can just concentrate on encrypting the data and forget about the management of the root keys.
* __AWS Owned Keys__ - We mentioned the discontionuation of AWS Managed Keys, because the new key on the block (sorry I couldn't resist!) is AWS Owned Keys. So, what do they do? An AWS owned key is a KMS key that is in an account managed by the AWS service, so the service operators have the ability to manage its lifecycle and usage permissions. By using AWS owned keys, AWS services can transparently encrypt your data and allow for easy cross-account or cross-region sharing of data without you needing to worry about key permissions. This is an incredibly important feature of using AWS Owned Keys, which are effectively AWS Managed Keys but with reduced blast radiuses. Exclusively controlled and only viewable by the AWS service that encrypts your data, and you also lose auditing/logging capabilities, for now. AWS service manages rotation, deletion, and Regional location, exactly as AWS Managed Keys do. You pay no charges for using AWS Owned keys which is by far one of the main advantages of using them!

The last thing we need to quickly discuss in this section is what the AWS Encryption SDK is, and envelope encryption. Straight from the [horse's mouth](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/introduction.html) again, the AWS Encryption SDK is a client-side encryption library designed to make it easy for everyone to encrypt and decrypt data using industry standards and best practices. It enables you to focus on the core functionality of your application, rather than on how to best encrypt and decrypt your data. The AWS Encryption SDK is provided free of charge under the Apache 2.0 license. 

The security of your encrypted data depends in part on protecting the data key that can decrypt it. One accepted best practice for protecting the data key is to encrypt it. To do this, you need another encryption key, known as a key-encryption key or wrapping key. The practice of using a wrapping key to encrypt data keys is known as envelope encryption. See the images below for a visual explanation on enevelope encryption:

![AWS Envelope Encryption](/assets/img/media/aws-kms-blogpost-envelope-encryption.gif)

Multiple wrapping keys can be used to encrypt the same data key, which adds some really good fault tolerance/disaster recovery controls:

![AWS Envelope Encryption Wrapper Keys](/assets/img/media/aws-kms-blogpost-wrapper-keys.gif)

Ok, this wraps up all that I wanted to cover about KMS and Enevelope Encryption for now. The AWS documentation is fantastic for further reading if you really want to know more. Now we will run through an example in Python, and call it a day.

## Lets play with Envelope Encryption

First of all, we need to import a bunch providers and helpers from the aws cryptographic material providers sdk.

{% highlight python %}
import boto3
from aws_cryptographic_material_providers.mpl import AwsCryptographicMaterialProviders
from aws_cryptographic_material_providers.mpl.config import MaterialProvidersConfig
from aws_cryptographic_material_providers.mpl.models import CreateAwsKmsKeyringInput
from aws_cryptographic_material_providers.mpl.references import IKeyring
from typing import Dict 

import aws_encryption_sdk
from aws_encryption_sdk import CommitmentPolicy
{% endhighlight %}

Next, we can create a global variable just purely as an example, with a byte string value. Lets also create the SDK+KMS clients and the encryption context:

{% highlight python %}
EXAMPLE_DATA: bytes = b"Hello KMS Learners"


def encrypt_and_decrypt_with_keyring(kms_key_id: str):
    client = aws_encryption_sdk.EncryptionSDKClient(
        commitment_policy=CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT
    )

    kms_client = boto3.client('kms', region_name="us-west-2")

    encryption_context: Dict[str, str] = {
        "encryption": "example context",
        "is not": "secret",
        "but adds": "useful metadata",
        "that can help you": "be confident that",
        "the data you are handling": "is what you think it is",
    }
{% endhighlight %}

We need to create the KMS keyring next:

{% highlight python %}
    material_provider: AwsCryptographicMaterialProviders = AwsCryptographicMaterialProviders(
        config=MaterialProvidersConfig()
    )

    keyring_input: CreateAwsKmsKeyringInput = CreateAwsKmsKeyringInput(
        kms_key_id=kms_key_id,
        kms_client=kms_client
    )

    kms_keyring: IKeyring = material_provider.create_aws_kms_keyring(
        input=keyring_input
    )
{% endhighlight %}

Next up we want to encrypt our example data, and do a quick assertion to confirm its now ciphertext (encrypted message):

{% highlight python %}
    ciphertext, _ = client.encrypt(
        source=EXAMPLE_DATA,
        keyring=kms_keyring,
        encryption_context=encryption_context
    )

    assert ciphertext != EXAMPLE_DATA, \
        "Ciphertext and plaintext data are the same. Invalid encryption"
{% endhighlight %}

And lastly, do a quick decryption test:

{% highlight python %}
    plaintext_bytes, _ = client.decrypt(
        source=ciphertext,
        keyring=kms_keyring,
        # Provide the encryption context that was supplied to the encrypt method
        encryption_context=encryption_context,
    )

    assert plaintext_bytes == EXAMPLE_DATA, \
        "Decrypted plaintext should be identical to the original plaintext. Invalid decryption"
{% endhighlight %}

This playground example is just demonstrating a single key keyring, have a look at `discovery_multi_keyring` for multi key keyrings.

## Letssss Goooo

Thank you for reading this post and hopefully you found some value in it! Some quick takeaway points:

* We learnt what KMS and the AWS Encryption SDK is, as well as Envelope Encryption.
* We learnt about the KMS key types, and a little bit of information on each type.
* You can have multiple wrapping keys for one data key, which is a really nice fault tolerance feature in the event of a disaster.
* Do you still need to think about things like [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) for additional hashing of ciphertext? Yes, I highly recommend adding these additional layers of defence in. Remember, your security is only as good as the layers of controls you put in-place to protect your data. 
* I do recommend thinking about using Customer Managed Keys over AWS Managed/Owned keys, and also thinking about multi-region and multiple wrapping keys. Think about how to reduce blast radiuses in the event of a major security incident.


Thanks all, and see you next time!