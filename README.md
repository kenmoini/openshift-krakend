# KrakenD on OpenShift

This repo contains Kubernetes/OpenShift manifests required for deploying the KrakenD API Gateway.

## Requirements

To deploy KrakenD you need a KrakenD configuration file.  This can be generated via the [KrakenD Designer](https://designer.krakend.io/).  There is also a dummy/test KrakenD configuration file included with this repo that has a Kubernetes/OpenShift ReadinessProbe health endpoint.

## Usage

### 1. Clone down the repo and enter the directory

```bash
$ git clone https://github.com/kenmoini/openshift-krakend
$ cd openshift-krakend
```

### 2. Setup your OpenShift Project

```bash
$ oc new-project my-krakend-gateway
```

### 3. Create your ConfigMap of the KrakenD config

The KrakenD container will look for a `/etc/krakend/krakend.json` file that needs to be mounted to the running Pods.  This is easily achieved with a ConfigMap.

Assuming you've either created a KrakenD configuration file via the KrakenD Designer, have one from a previous export, or are using the dummy/test config included with this repo...

```bash
$ oc create configmap krakend-config --from-file=krakend.json
```

...where krakend.json is your configuration file.  The included configuration will create a KrakenD service on port 8080, with a single ***/healthz*** endpoint used by the ReadinessProbe.

### 4. Deploy the Template

Now that you have the ConfigMap available in the namespace/project, you can deploy one of the Templates that will instanciate the resources...

```bash
# Persistent Volume
$ oc apply -f krakend-persistent-configmap-template.yaml
# Ephemeral Storage
$ oc apply -f krakend-configmap-template.yaml
# Persistent Volume, Edge terminated SSL
$ oc apply -f krakend-persistent-configmap-secure-template.yaml
# Ephemeral Storage, Edge terminated SSL
$ oc apply -f krakend-configmap-secure-template.yaml
```

### 5. Secure the API Gateway

At this point you should only have to wait momentarily to have the KrakenD API Gateway come up.  Once it does, I highly suggest securing the Route with SSL if you did not do so with one of the ```-secure``` templates.

## Notes on using KrakenD with OpenShift/Kubernetes

- If you are using some sort of Ingress you may need to set CORS for your reverse proxy