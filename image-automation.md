

# Image Automation

Inside the services folder we have the image-automation folder. This repository is responsible for image automation.
If image is updated in the image repository then new image will be pushed into this repository in deployment.yaml file of particular service.
Flux will reconcile with that image and sync cluster.

Image automation is a feature of flux that is used to fetch new images from registry.
Inside image automation folder for every environment, there is an image automation folder for every service.

`image-policy.yaml` - This is the policy that is used to determine the latest image for the registery. Our images are tagged in the following format `registry/<service-name>:<branch-name>-<commit-hash-7-digits>-<timestamp>`
The image policy figures out the timestamp from the tag using a regex pattern and gets the latest image in the ascending order.

`image-repository.yaml` - The image repository file specifies from which registry the image has to be fetched and it is reference in the `image-policy.yaml` file.

`image-update-automation.yaml` - When the image policy detects a new image from registry according to the timestamp, the image update automation will come into picture and it will try to push a commit into the repository on the specified branch. Flux will then sync the updated repository with the state in the cluster and the new image will be used by the service.

Where does image update automation make the udpate ?

In the deployment yaml file there is a comment against the image name `# {"$imagepolicy": "<image-policy-namespace>:<image-policy-name>"}`. This is where the new image id is put.

## Changes required for adding image automation to a new service.

1. Create services/railtel-new/image-automation folder for service.
2. Copy over all files from sample like    `ray-audit-service`.
3. Change names in all files.
4. Edit kustomization file of the service itself and import the image automation folder.
5. Add comment in the deployment file for manifest.