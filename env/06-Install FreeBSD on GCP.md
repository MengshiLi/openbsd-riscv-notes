### How to list freeBSD images:
```
gcloud compute images list \
--project freebsd-org-cloud-dev \
--no-standard-images | grep -i freebsd-12
```

### How to instantiate a freeBSD OS on google cloud platform:
From gdk console:
```
gcloud compute instances create "freebsd12-2" \
--zone "us-west1-b" \
--machine-type "n1-standard-8" \
--network "default" \
--maintenance-policy "MIGRATE" \
--image "freebsd-12-1-release-amd64" --image-project=freebsd-org-cloud-dev \
--boot-disk-size "200" 
```
