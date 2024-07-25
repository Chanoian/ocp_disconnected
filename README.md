
![image description](screenshot.svg)

# ocp_disconnected

This repo will have valuable instructions and manifests for an AirGapped OCP Deployment.

## Required Tools:
    - oc-mirror
    - mirror-registry
    - openenshift-install

## OCP Pull Secret
The default path for the pull secret is `~/.docker/config.json`. you can obtain that from https://console.redhat.com/openshift/install/pull-secret

## Running OCP-MIRROR
ocp-mirror --config manifests/ImageSetConfiguration.yaml file://./folder_path

This command will use the ImageSetConfiguration.yaml file to download the installation images and operators.

## Install mirror-registry (local quay.io)
copying the mirror-registry.tar.gz file to the highside system using rsync (which is the disconnected system) and then install it uisng:
```
./mirror-registry install --initPassword <password>
```

## Trusting the Mirror-Registry Self-Signed Certificate
since this is a self signed certificate , that is not trusted by anything , not even the highside system where this is installed.
we can run this command to add the certificate to the trusted list:
```
sudo cp -v $HOME/quay-install/quay-rootCA/rootCA.pem /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust
```
to test it out we can use podman login
```
podman login -u init -p <password> $(hostname):8443
```

## Copying the images to the highside system
```
rsync -avP /from/bastion /to/disconnected
```
## Afterwards we have to upload the copied images to the mirror-registry running instance on the disconnected env
after making sure that the mirror-registry is running on the disconnected env we have to upload the copied data over to it.

```
oc-mirror --from=/folder_path/mirror.tar docker://$(hostname):8443
```

## Create SSH Key to debug the cluster

```
ssh-keygen -C "OpenShift Debug" -N "" -f /mnt/high-side-data/id_rsa
echo "sshKey: $(cat /mnt/high-side-data/id_rsa.pub)" | tee -a /mnt/high-side-data/install-config.yaml
```

this will add ssKey to the install-config.yaml file.

## Adding pullSecret and imageContentSourcePolicy to the install-config.yaml

these two are important for the installation

## Install OCP
```
openshift-install create cluster --dir /mnt/install-config-path
```

## Post Install
There are three things mainly to do after disconnected installation


1. Disable the default app / Operator CatalogSources

2. Create your own app / Operator CatalogSource

3. Install additional ImageContentSourcePolicies

