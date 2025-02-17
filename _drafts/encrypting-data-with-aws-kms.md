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
