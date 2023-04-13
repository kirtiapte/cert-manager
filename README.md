# cert-manager

## Concepts

### Cluster Issuers
Issuers, and ClusterIssuers, are Kubernetes resources that represent certificate authorities (CAs) that are able to generate signed certificates by honoring certificate signing requests. All cert-manager certificates require a referenced issuer that is in a ready condition to attempt to honor the request. 
### Certificate
cert-manager has the concept of Certificates that define a desired X.509 certificate which will be renewed and kept up to date. A Certificate is a namespaced resource that references an Issuer or ClusterIssuer that determine what will be honoring the certificate request. Ref
### Certificate Request: 
The CertificateRequest is a namespaced resource in cert-manager that is used to request X.509 certificates from an Issuer. The resource contains a base64 encoded string of a PEM encoded certificate request which is sent to the referenced issuer. A successful issuance will return a signed certificate, based on the certificate signing request. CertificateRequests are typically consumed and managed by controllers or other systems and should not be used by humans - unless specifically needed. Ref
### Orders and challenges: 
Cert-manager supports requesting certificates from ACME servers with use of the ACME Issuer. These certificates are typically trusted on the public Internet by most computers. To successfully request a certificate, cert-manager must solve ACME Challenges which are completed in order to prove that the client owns the DNS addresses that are being requested.
In order to complete these challenges, cert-manager introduces two CustomResource types; Orders and Challenges. Ref

### Orders: 
Order resources are used by the ACME issuer to manage the lifecycle of an ACME 'order' for a signed TLS certificate. An order represents a single certificate request which will be created automatically once a new CertificateRequest resource referencing an ACME issuer has been created. CertificateRequest resources are created automatically by cert-manager once a Certificate resource is created, has its specification changed, or needs renewal. 

As an end-user, you will never need to manually create an Order resource. Once created, an Order cannot be changed. Instead, a new Order resource must be created.
Challenges: Challenge resources are used by the ACME issuer to manage the lifecycle of an ACME 'challenge' that must be completed in order to complete an 'authorization' for a single DNS name/identifier.
When an Order resource is created, the order controller will create Challenge resources for each DNS name that is being authorized with the ACME server.
As an end-user, you will never need to manually create a Challenge resource. Once created, a Challenge cannot be changed. Instead, a new Challenge resource must be created.

## User Flow

- User creates issuer and certificate
<pre>
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: <issuer_name>
  namespace: <namespace>
spec:
  acme:
    email: <email_address>
    #New enrolments only
    server: https://acme.digicert.com/v2/acme/directory/
    externalAccountBinding:
      keyID: <eab_kid>
      keySecretRef:
        name: <eab_secret_name>
        key: secret
      keyAlgorithm: HS256
    privateKeySecretRef:
        name: <account_private_key_name>
    solvers:
    - http01:
        ingress:
          class: nginx

---

apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: <certificate_name>
  namespace: <namespace>
spec:
  dnsNames:
    - <certificate_common_name>
  secretName: <certificate_private_key_name>
  issuerRef:
    name: <issuer_name>

</pre>

## Reference
https://cert-manager.io/
