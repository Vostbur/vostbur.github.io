---
categories:
- Programming
date: "2021-07-21"
description: Deploying the django app on the DigitalOcean App Platform.
slug: django-on-digitalocean-app-platform
tags:
- python
- django
- DevOps
- DigitalOcean
title: Deploying the django app on the DigitalOcean App Platform
---

For deploying Django apps, I tried PaaS such as *Heroku* and *Pythonanywhere*. It's the turn of the **DigitalOcean App Platform**. And so far, this is the easiest and fastest way to deploy a Django app in production. There is awesome [guide](https://www.digitalocean.com/community/tutorials/how-to-deploy-django-to-app-platform) on how to do this on DigitalOcean's tutorials.

For testing, I wrote a small [application](https://github.com/Vostbur/cisco-conf-template-generator) that generates a configuration snippet for configuring ssh on cisco devices. For simplicity, I didn't use a database and static content. For such an application, in fact, there is no need for a server side, just JavaScript is enough :)

Comparing with the guide, I skipped the part with the database and statics. To avoid an error when calling `python manage.py collectstatic`, I added the environment variable **DISABLE_COLLECTSTATIC=1**. In addition, as a virtual environment for python, I have use **pipenv**, with which everything is also perfectly deploying.

The result can be seen [here](https://cisco-conf-template-generator-g77mm.ondigitalocean.app/) for some time. When I have played enough, I will delete the application.

![](/images/django-with-do-app-platform.PNG)

----
### Links:

- [1] [How To Deploy a Django App on App Platform](https://www.digitalocean.com/community/tutorials/how-to-deploy-django-to-app-platform)
- [2] [github.com/Vostbur/cisco-conf-template-generator](https://github.com/Vostbur/cisco-conf-template-generator)
