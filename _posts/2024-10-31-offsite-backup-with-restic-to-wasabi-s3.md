---
layout: post
title: "Offsite backup with Restic to Wasabi S3"
date: 2024-10-31
categories: [admin]
tags: [backup, s3, restic, wasabi]
---

In today's tutorial I wanted to share the idea of versioned, encrypted backup, pushed to cloud storage. Offsite backup is extremely important for our data preservation in case of physical damage of our local drives. As a self hosted enthusiast, I can only consider solutions, which allows me to encrypt my data before giving it to someone else's hands and versioning allows us to prevent situation, when we unconsciously push corrupted data to our backup device and we can't restore the previous state.

I will present the simplest way to send backup to the cloud and restore it. There is no need to focus on the more advanced features of both technologies for now. I want you to understand the simplest workflow.

# Tools Overview

**S3** is a standard of object storage developed by Amazon. It has been implemented also by other providers. Wasabi is one of them and has very attractive price. If you want to follow this tutorial, please register you Wasabi account: https://wasabi.com/. They provide long trial period without charging.

You can upload and download files straight from web ui of Wasabi, but it doesn't seem to be convenient and probably never ment to be. If you really want, you can even combine S3 storage with desktop applications, like FileZilla, in order to create own file server. It is also often used as a storage for NextCloud instances, but this is not in the scope of this tutorial.

**Restic** is a command line software for files backup. It creates versioned and encrypted backup, that can either be stored locally, or can be sent to S3. The behavior is quite simmilar to git repositiory - first, we create it, encrypt with password, an than we "push" the state of certain folder to it. Once we want to restore backup, we "pull" the chosen version of the backup.

All you need to follow along with this tutorial is the Wasabi account and any computer or VM with Ubuntu.

# Setting up S3 storage

Create new bucket - this is going to be your external "drive", where you will be sending your files. 

* Click on "Create Bucket".
* For te sake of this tutorial, please call it "tst".
* Chose preferred region.
* Don't change anything in step 2, just confirm defaults and create bucket.

# Creating Wasabi user

In the next step, we need to create user, that will have api keys, that will allow him to access storage created in the previous step. We will need those keys to access storage from the host machine.

* Click on "Users" in the left side menu and click on "Create User" in top right corner.
* For the sake of this tutorial, please set username to 'tstuser'.
* Tick option called "Programmatic (create API key) and click next.
* In the next step, we need to choose policy, the set of permissions for that user. Let's keep it simple for now and choose "WasabiFullAccess", which will give full access to all buckets for that user.
* Click "next" up to review page and confirm by clicking "Create User"
* In the next step, click on "COPY KEYS TO CLIPBOARD", we will need that keys in the next step.

# Creating Restic repository on S3 bucket and pushing backup

Install restic on your machine with this simple command:

```
sudo apt install restic
```

Create restic repo on your S3 bucket. In order to do that, we need to keys from the previous step as environmental variables (remember to paste them between quotes) and paste you bucket name into url:

```
export AWS_ACCESS_KEY_ID="<aws-access-key-id>"
export AWS_SECRET_ACCESS_KEY="<aws-secret-access-key>"

restic -r s3:https://s3.eu-central-1.wasabisys.com/<bucket-name> init
```

You can refresh your Wasabi webpage and notice, that restic repo had been created inside your bucket.

# Backup test

Create test folder with some files, example: `mkdir ~/tst && touch ~/tst/file{00..10}` and push that foldert to restic repo:

```
restic backup ~/tst -r s3:https://s3.eu-central-1.wasabisys.com/<bucket-name>
```

You can eperiment - delete some files or create new and repeat previous command, to push new changes to your repo. Check available versions with command:

```
restic snapshots -r s3:https://s3.eu-central-1.wasabisys.com/<bucket-name>
```

# Restore test

Restore the latest version with command:

```
restic restore latest --target ~/restore -r s3:https://s3.eu-central-1.wasabisys.com/<bucket-name>
```

If you want to restore certain version, you can do it by replacing "latest" with the number of version (check it with the "snapshots" option from the previous example):

```
restic restore <snapshot number> --target ~/restore -r s3:https://s3.eu-central-1.wasabisys.com/<bucket-name>
```

I hope, that was helpful. If you have any questions, please leave in the comments below.
