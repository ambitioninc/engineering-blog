---
title: Syncing Django static and media files to S3
layout: post
author: Micah Hausler
comments: true
---

An early problem we had at Ambition was syncing user-uploaded profile photos and other content across multiple app servers. Originally, we used a shared NFS mount, (or as one engineer called it, the Nightmare File System) and that soon had its own set of I/O scaling issues. Seeking a more scalable solution as we moved into Amazon Web Services, we looked into [django-storages](http://django-storages.readthedocs.org/en/latest/), and specifically their [s3boto backend](http://django-storages.readthedocs.org/en/latest/backends/amazon-S3.html). We also had much success with [django-extensions](http://pythonhosted.org/django-extensions/sync_media_s3.html) `sync_media_s3` for uploading existing media to S3.

First Try
---------

Our first configuration was based from a [stackoverflow.com answer](http://stackoverflow.com/questions/10390244/how-to-set-up-a-django-project-with-django-storages-and-amazon-s3-but-with-diff) that separated the static and media content. After digging into the [django-storages source code](https://bitbucket.org/david/django-storages/src/e27c8b61ab57e5afaf21cccfee005c980d89480f/storages/backends/s3boto.py?at=default#cl-200), we realized there were a host of other AWS specific settings at our disposal to configure.

```
DJANGO_APPS = ( 
    # ... 
    'storages',
    # ... 
)


STATICFILES_STORAGE = 'ambition.settings.s3_utils.StaticRootS3BotoStorage'
STATIC_URL = 'https://ambition-{}-static.s3.amazonaws.com/'.format(os.environ.get('CUSTOMER'))

DEFAULT_FILE_STORAGE = 'ambition.settings.s3_utils.MediaRootS3BotoStorage'
MEDIA_URL = 'https://ambition-{}-media.s3.amazonaws.com/'.format(os.environ.get('CUSTOMER'))

AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')

# separate s3_utils.py file
from storages.backends.s3boto import S3BotoStorage

StaticRootS3BotoStorage = lambda: S3BotoStorage(
    bucket_name='ambition-{}-static'.format(os.environ.get("CUSTOMER")),
    reduced_redundancy=True
    headers = {}, 
    file_overwrite = True,
    gzip = False)

MediaRootS3BotoStorage  = lambda: S3BotoStorage(
    bucket_name='ambition-{}-media'.format(os.environ.get("CUSTOMER")))
```
Second Try
----------

After sitting in AWS a couple months, we realized two things. 1. syncing static assets to S3 made `manage.py collectstatic` take a long time, and we could really optimize static content using [Google's nginx pagespeed module](https://github.com/pagespeed/ngx_pagespeed). 2. Amazon has a hard limit of 100 S3 buckets. At this rate, we will run out of buckets when we approach 50 customers. Since we have separate credentials for every customer, we can provision their IAM users to access the same bucket, but a different [location within the bucket](https://bitbucket.org/david/django-storages/src/e27c8b61ab57e5afaf21cccfee005c980d89480f/storages/backends/s3boto.py?at=default#cl-228). Here is how our configuration now stands.

```
 # regular Django staticfiles settings
STATIC_URL = ... 
STATIC_ROOT = ... 

MEDIA_URL = 'https://ambition-media.s3.amazonaws.com/{}/'.format(os.environ.get("CUSTOMER"))
DEFAULT_FILE_STORAGE = 'ambition.settings.s3_utils.MediaRootS3BotoStorage'

# separate s3_utils.py file
from storages.backends.s3boto import S3BotoStorage

MediaRootS3BotoStorage  = lambda: S3BotoStorage(
    bucket_name='ambition-media',
    location=os.environ.get("CUSTOMER"),
    )   
```

My slides from djangocon 2013 are now [posted on speakerdeck](https://speakerdeck.com/micahhausler/django-syncing-static-and-media-to-s3).
