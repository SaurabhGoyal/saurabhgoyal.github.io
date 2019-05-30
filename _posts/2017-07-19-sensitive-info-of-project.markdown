---
layout: post
title:  "Sensitive info in the project!"
subtitle: Are your AWS keys public?
date:   2017-07-19 22:48:07 +0530
categories: tech-blog
---
I have been more and more involved in tasks such as deployment and server management recently and I am still struggling through the nuts and bolts of the whole process from pushing the changes in the repository to deploying those and making sure that each resource in the infrastructure has successfully started running with the new changes.

When going through our deployment scripts and resources the other day, this question popped up in my head.
>How do we manage our sensitive info in the project?

The sensitive info that consists of any of the below and more-
* ```API tokens``` to third party services. (such as Google APIs, Twilio etc.)

* ```Credentials/access keys``` to connect to either your own resources or others'. (such as AWS secret and access keys, address of resources such as servers, database instances etc.)

* ```Configurations``` for various services for your app. (such as Email configuration, Encryption keys, Critical settings or flags etc.)

After some digging, I found we have stored these into the codebase only and is included in the version control system(git). This is a basic mistake that people do in the initial phase of a project without understanding the consequences. ```I'll talk about it more later in the post.```

Before proceeding, we must understand how and why above happens.
So you have created your awesome app which uses some third party APIs or have some critical configuration. The data for those is sensitive and must not be made public, so you have setup your project such that whenever someone is initiating your app, he or she will provide all that sensitive data in a configuration file or through environment variables manually and your app will use that.

All your sensitive info is off the code-base, secured to one or more authorized personnels and things work fine ```UNTILL``` your product starts to grow and you face following problems.
* You have ```multiple environments``` of your app - dev, staging, prod, each with a different set of sensitive data/configuration.

* The sensitive data/configuration itself starts getting big as ```number of services and resources you use increase```.

* You have whole ```system and separate team for DevOps```, and the work is not restricted to a set of personnel anymore. The way they store/manage the sensitive data is a big concern for you and may cause you tons of losses in worst cases.

That is to say, **the offline method is not scalable**. When above happens and there are multiple versions of configuration files, you need to find a way to store them securely and manage them better, also making sure their availability and integrity. You can't keep all those offline anymore.

## The intuitive and probably the best option is to put them in a private secure location (accessible only be authorized personnels such as devops engineers) and whenever instantiating the app, all data is pulled from there.
* The locations can be places like a ```repository``` or ```AWS S3 bucket``` or any of the multiple available services to store sensitive data (```PassPack, Swordfish, CHEF server``` etc.). In any case, **we must always store encrypted data to those places and decrypt before using at our end to ensure security even if data from that location is compromised.**

* Although, in that case you'll have to provide keys/credentials to fetch the data from that secure location. But this is not same as before because we have successfully abstracted all sensitive data, which may increase a lot, behind one secure location/service, size of credentials to which is and will always be fixed.

The above is the basic idea on how to automate deployment and configuration of one or more servers while keeping the sensitive data managed at one place securely. Of-course it'll need some work at our end to modify our deployment script accordingly. To give you a better understanding, I'll explain what We did in one of my projects-

* We stored all the sensitive data in an S3 bucket. The data was encrypted before upload using a password. We used a tool called ```Ansible-Vault``` for encryption which is good because we use ```Ansible``` for all our deployment scripts and ```Ansible-Vault``` is a module of ```Ansible```. The major benefit of using ```Ansible-Vault``` is that when running Ansible scripts, if we provide the password of encryption used with Ansible-Vault then ```Ansible```, when processing any variable file, decrypts the file before its use automatically.

* Alternatively, we could use something else for encryption and modify our deployment script to decrypt the files manually.

* Now whenever deploying, all that is needed by the script are AWS keys for fetching file from S3 and the password/key for decrypting that file.

**Note:-**

__*A repository is better in terms of modification and history management of sensitive data itself, which is not fully available with S3. The benefit mainly comes when you have lots and lots of sensitive data and multiple DevOps engineers who may be making changes to same key/s simultaneously. Although this is improbable in most projects.*__

---

Now let's discuss what I wrote earlier - **Committing any sensitive data to version control system is a big mistake.**
Because-

* The sensitive data is in the history even after moving out. Proper removal will require-

    * ```History manipulation``` - which although possible is not very clean and smooth.

    * ```Rotation of all your sensitive data``` - Once a piece of sensitive data is public, it's never fully safe to use that. Eventually you'll have to rotate it for a new value which may be quite some work based on it's purpose and howmany places is it being used.

With all that said, there are multiple tools such as ```truffleHog``` which search repos for any sensitive info in full history. They must be used if not confident.

