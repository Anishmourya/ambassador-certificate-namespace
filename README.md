# ambassador-certificate-namespace
This is an example to integrate cert-manager with the ambassador on multiple Kube namespaces

The purpose of this repo to solve the ambassador's namespace certificate issue.
#https://github.com/datawire/ambassador/issues/2989

# Overview
1. Create staging, development, production namespace on Kubernetes to manage services environment properly.
2. Installed Ambassador from official helm chart:- https://github.com/datawire/ambassador-chart
3. Used Cert-Manager to install certificates for domains. https://cert-manager.io/docs/installation/kubernetes/
4. Used Test domain "domain.com" as base domain and for development:- development.domain.com & same for staging.
5. Used Flask app as service for each related environments.
6. Used cluster issuer as letsencrypt production. 


# Solution
Ambassador doesn't support ingress rules, It does support Mappings. While cert-manager still use ingress rules,
to install certificates for domains. Cert manager performs acme-domain-verification to verify requested domains.
To fix it does support:-
```
---
apiVersion: getambassador.io/v2
kind: Mapping
metadata:
  name: acme-challenge-mapping
spec:
  prefix: /.well-known/acme-challenge/
  rewrite: ""
  service: acme-challenge-service
---
apiVersion: v1
kind: Service
metadata:
  name: acme-challenge-service
spec:
  ports:
  - port: 80
    targetPort: 8089
  selector:
    acme.cert-manager.io/http01-solver: "true"
```
but, what happens it needs to apply for all namespaces & renew certificates automatically.
If I do create mapping & services for each environment then it does not perform verification which I mentioned in GitHub issue link.
So its just Mapping issue of ambassador that getting confused to transfer acme-request to correct requested namespaces.

After so many time, I have gone through the docs & also raised an issue on the ambassador, realised again.
1. Ambassador use ambassador_id to identify the namespace for mappings.
2. We can set ambassador_id to use Kubernetes DNS to map the traffic to internal services.
3. If ambassador_id is default then we can add define the service uniquely to add namespace name as the suffix -ie:- acme-challenge-development-service.development
    here development is the namespace of service.
 
 I have tested it perfectly & working as expected, it is installing & renew the certificates perfectly for all domains for different namespaces as well.   
