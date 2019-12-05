# KrakenD on OpenShift

This repo contains Kubernetes/OpenShift manifests required for deploying the KrakenD API Gateway.
There are also instructions below on how to integrate JWT Validation with Keycloak/Red Hat SSO.

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

### JWT Validation with Keycloak/Red Hat Single Sign On (SSO)

#### Preliminary Keycloak/RH SSO Setup

1. Deploy Keycloak, create admin user
2. Create Realm
3. Create Client for KrakenD
4. Create Mapper for Client
  - Name: userRealmRoles
  - Mapper Type: User Realm Role
  - Multivalued: On
  - Token Claim Name: userRealmRoles
  - Add to ID token: On
  - Add to access token: On
  - Add to userinfo: On
5. Create ***user_basic*** Realm Role
6. Create ***Admin*** & ***Users*** Groups
7. Create test User
8. Assign Groups the ***user_basic*** Realm Role
9. Assign the test User to the Groups


You may be using this to proxy API requests in order to wrap them with JWT for authentication.  Maybe you're even using Keycloak...

1. Obtain your JWKS/OpenID Connect Configuration URL

When configuring JWT Validation in a KrakenD Endpoint, you'll need the URL with the configuration information for your Realm in Keycloak.

Assuming your Realm is named **example**...

```https://sso-keycloak.domain.here.com/auth/realms/example/.well-known/openid-configuration```

2. Extract the **jwks_uri**

Now with that OpenID Connect Configuration information, you need to find a key in that JSON called ***jwks_uri*** - this is the URL you pass to KrakenD for the JWK URL.
Should look something like...

```https://sso-keycloak.domain.here.com/auth/realms/example/protocol/openid-connect/certs```


3. Extract the **issuer**

This one is pretty easy, it's the first key in that OpenID Connect config JSON called ***issuer***.
Should look something like...

```https://sso-keycloak.domain.here.com/auth/realms/example```

4. Set Configuration in KrakenD to connect to Keycloak

Set the extracted jwks_url as your JWK URI and issuer for Issuer.

Odds are if you're using Keycloak you're using the RS256 algo.  Change the Algorithm to reflect this.
Set your ***Roles*** to *user_basic* and the ***Roles Key*** to *userRealmRoles*