# Gateway File

[TOC]


The document briefly covers the use of a gateway within the Helm Template.

A gateway describes a load balancer operating at the edge of the mesh that receives incoming or outgoing HTTP/TCP connections (entrypoint into your cluster, alternative to an ingress controller).

The gateway will direct traffic to a microservice in the server using a virtualservice.

The specification with the declaration file will define a set of ports that should be exposed, the type of protocol to use, SNI configuration for the load balancer etc.

> **Note:** To learn more about the use of Gateway, please refer to the following [Istio Documentation](https://istio.io/latest/docs/reference/config/networking/gateway/)


The gateway configuration sets up a proxy to acts as a load balancer which exposes specified ports for ingress.

The gateway will be applied to the proxy running on a pod with the label specified.

A virtualService can then be bound to a gateway to control the forwarding of traffic arriving at a particular host or Gateway port.

> **Note:** It is the responsibility of the developer to ensure that the external traffic to these ports are allowed into the Service Mesh.


```
{{- $serviceMesh := include "eric-assurance-visualization.service-mesh.enabled" . | trim -}}
{{- $serviceMeshIngress := include "eric-assurance-visualization.service-mesh-ingress.enabled" . | trim -}}
{{- if and (eq $serviceMesh "true") (eq $serviceMeshIngress "true") -}}
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: {{ template "eric-assurance-visualization.name" . }}-gas-gateway
  annotations:
  {{- include "eric-assurance-visualization.helm-annotations" .| nindent 4 }}
  labels:
  {{- include "eric-assurance-visualization.labels" .| nindent 4 }}
spec:
  selector:
    app: service-mesh-ingress-gateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
      - {{ required "A valid .Values.global.hosts.gas entry required" .Values.global.hosts.gas }}
    tls:
     httpsRedirect: true # sends 301 redirect for http requests
  - port:
      name: https-gas
      number: 443
      protocol: HTTPS
    hosts:
      - {{ required "A valid .Values.global.hosts.gas entry required" .Values.global.hosts.gas }}
    tls:
      mode: SIMPLE # enables HTTPS on this port
      credentialName: {{ required "A valid .Values.ingress.tls.secretName entry required" .Values.ingress.tls.secretName }}
{{- end }}
```


- In the example gateway above, the gateway configuration sets up a proxy to act as a load balancer exposing port 80 (HTTP) and port 443 (HTTPS) for ingress.


- For the exposure of port 80 (HTTP), all HTTP requests will send a 301 redirect in order to notify the user to use HTTPS.


- For the exposure of port 443 (HTTPS), indicates how strictly TLS is enforced, with a reference to the secret that holds the TLS certs (Including the certificate authority certificates) for Kubernetes.

```
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
      - {{ required "A valid .Values.global.hosts.gas entry required" .Values.global.hosts.gas }}
    tls:
     httpsRedirect: true # sends 301 redirect for http requests
  - port:
      name: https-gas
      number: 443
      protocol: HTTPS
    hosts:
      - {{ required "A valid .Values.global.hosts.gas entry required" .Values.global.hosts.gas }}
    tls:
      mode: SIMPLE # enables HTTPS on this port
      credentialName: {{ required "A valid .Values.ingress.tls.secretName entry required" .Values.ingress.tls.secretName }}
```

- The gateway defined will be applied to the proxy running on a pod with the follow label:

```
app: service-mesh-ingress-gateway
```

Since we have already established a gateway that takes in HTTP and HTTPS requests, we do not need to change anything.

However, if you wanted to add other protocol such as MONGO, TCP, HTTP2 etc., a new port would have to be exposed with the desired protocol specified.


If we want to open up the internal service (Health-checker in the GAS application) for the Health-Check url:

We must use the port within the gateway (Port 80/Port 443) in order to allow http requests which will be used to reroute to the health-checker service defined in the virtual service.

This will allow for the gateway to redirect traffic to the Health-Checker internal service.


| Parameter  | Description                                                                                                                                                                                            | Default              |
|------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|
| apiVersion | What version of the Kubernetes API you're using to create the Object                                                                                                                                   | networking.k8s.io/v1 |
| kind       | Which kind of object you want to create.                                                                                                                                                               | Gateway              |
| metadata   | Data that helps uniquely identify the object (Could include name, UID, namespace, labels, annotations etc.)                                                                                            |                      |
| spec       | Contains the information needed to configure a Gateway.                                                                                                                                                |                      |
| servers    | A list of server specifications.                                                                                                                                                                       |                      |
| selector   | One or more services that indicate a specific set of pods/VMs on which this Gateway configuration should be applied. By default workloads are searched across all namespaces based on label selectors. |                      |


The Server section of the gateway definition above also contains a Parameter List:

| Parameter | Description                                                                                                                                                                                                                      |
|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Port      | Port on which proxy should listen for incoming connections.                                                                                                                                                                      |
| hosts     | One or more hosts exposed by this Gateway. A host is specified as a DNS name with an optional prefix. A Virtual Service must be bound to the Gateway and must have one or more hosts that match the hosts specified in a server. |
| tls       | Set of TLS related options that govern the server's behaviour. Can control if all HTTP requests should be redirected to HTTPS, and the TLS modes to use.                                                                         |


The port section underneath the server section of the gateway definition above also contains a parameter List:

| Parameter  | Description                                                         |
|------------|---------------------------------------------------------------------|
| number     | The specified Port number that you want to open for ingress.        |
| protocol   | Protocol you want to expose on the Port (HTTP/HTTPS/MONGO/TCP etc.) |
| name       | Label assigned to the Port.                                         |