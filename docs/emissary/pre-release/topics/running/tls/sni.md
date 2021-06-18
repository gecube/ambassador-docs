# Server Name Indication (SNI)

$productName$ supports serving multiple `AmbassadorHost`s behind a single IP address, each
with their own certificate. 

This is as easy to do as creating an `AmbassadorHost` for each domain or subdomain you 
want $productName$ to serve, getting a certificate for each, and telling 
$productName$ which `AmbassadorHost` the route should be created for.

The example below configures two `AmbassadorHost`s and assigns routes to them.

## Configuring an `AmbassadorHost`

The `AmbassadorHost` resources lets you separate configuration for each distinct domain 
and subdomain you plan on serving behind $productName$.

Let's start by creating a simple `AmbassadorHost` and providing our own certificate in 
the `host-cert` secret.

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: example-host
spec:
  hostname: host.example.com
  acmeProvider:
    authority: none
  tlsSecret:
    name: host-cert
```

Now lets, create a second `AmbassadorHost` for a different domain we want to serve behind
$productName$. This second `AmbassadorHost` we can use $AESproductName$'s automatic TLS
to get a certificate from Let's Encrypt.

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorHost
metadata:
  name: foo-host
spec:
  hostname: host.foo.com
  acmeProvider:
    email: julian@example.com
```

We now have two `AmbassadorHost`s with two different certificates.

## Configuring routes

Now that we have two domains behind $productName$, we can create routes for either
or both of them.

We do this by setting the `host` attribute of an `AmbassadorMapping` to the domain the
`AmbassadorMapping` should be created for. 

```yaml
---
apiVersion: getambassador.io/v2
kind:  AmbassadorMapping
metadata:
  name:  httpbin
spec:
  prefix: /httpbin/
  service: httpbin.org:80
  host_rewrite: httpbin.org
  host: host.example.com
```
Will create a `/httpbin/` endpoint for `host.example.com`
```yaml
---
apiVersion: getambassador.io/v2
kind:  AmbassadorMapping
metadata:
  name:  mockbin
spec:
  prefix: /foo/
  service: foo-service
  host: host.foo.com
```
Will create a `/foo/` endpoint for `host.foo.com`

```yaml
---
apiVersion: x.getambassador.io/v3alpha1
kind: AmbassadorMapping
metadata:
  name: frontend
spec:
  prefix: /bar/
  service: bar-endpoint
```
Will create a `/bar/` endpoint for all `AmbassadorHost`s.