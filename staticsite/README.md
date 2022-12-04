# Template chart for static websites 

This chart deploys a basic website on Kubernetes. There are some parameters that needs to be set in values.yml for each deployment.
 
- repository:  Name of the image on gitlab.hamrah.in
- nameOverride, fullnameOverrid: Set the app name, details are in values.yml
- serviceAccount.name: Name of service account for app
