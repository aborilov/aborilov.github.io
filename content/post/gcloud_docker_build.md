---
title: "Build docker container in GCP container builder"
date: 2018-07-30T09:00:16+03:00
---

If you need to build docker container and want to save your laptop battery, you can easily
build your container in GCP container builder.

You will need to have `gcloud` on your system.

Authorize the gcloud command-line tool to access your project:

```
gcloud auth login
```

Configure your project for the gcloud tool, where [PROJECT_ID] is your GCP project ID that you created or selected in the previous section:

```
gcloud config set project [PROJECT_ID]
```

and now you can build your image using GCP container builder

```
gcloud builds submit --tag gcr.io/[PROJECT_ID]/quickstart-image .
```

you run this image locally you need to configure Docker to use your Container Registry credentials when interacting with Container Registry (you are only required to do this once):

```
gcloud auth configure-docker
```

and now you can pull your image and run
```
docker run gcr.io/[PROJECT_ID]/quickstart-image
```

You also can use build config file
```
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [ 'build', '-t', 'gcr.io/$PROJECT_ID/quickstart-image', '.' ]
images:
- 'gcr.io/$PROJECT_ID/quickstart-image'
```

and if you want to use cache to speed up your builds you can add
```
'--cache-from', 'gcr.io/$PROJECT_ID/quickstart-image',
```
to your config
```
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: [
            'build',
            '-t', 'gcr.io/$PROJECT_ID/quickstart-image',
            '--cache-from', 'gcr.io/aborilov/senseware',
            '.'
        ]
images:
- 'gcr.io/$PROJECT_ID/quickstart-image'
```
and don't forget to pull newly builded image

```
docker pull gcr.io/[PROJECT_ID]/quickstart-image
```

> use separate folder for build, because gcloud will compress it and send to cloud


