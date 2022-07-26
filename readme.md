# Pathauth

Pathauth is a middleware plugin for [Traefik](https://github.com/traefik/traefik) to apply more detailed authorization
to multiple endpoints at once. This plugin was developed to work well together
with [Traefik Enterprise OpenId Connection Authentication Middleware](https://doc.traefik.io/traefik-enterprise/middlewares/oidc)
and [thomseddon traefik-forward-auth](https://github.com/thomseddon/traefik-forward-auth)

## Configuration

### Static

```yaml
pilot:
  token: "xxxxx"

experimental:
  plugins:
    pathauth:
      moduleName: "github.com/portofrotterdam/pathauth"
      version: "v0.1.5"
```

### Dynamic

```yaml
http:
  middlewares:
    pathauth-foo:
      pathauth:
        source:
          type: "header" # optional, default = header
          name: "X-Forwarded-User"
          delimiter: "," # the delimiter is useful if the input header has multiple values, for example roles. We can then check if the request meets any of the values from the headers. optional
        authorization:
          - path:
              - ".*/admin/.*" # regex, multiple paths allowed
              - ".*/swagger-ui/.*"
            host:
              - "^example.com$" # regex, optional, multiple hosts allowed
              - "^foo.example.com$"
            allowed:
              - "update-only-user"
              - "admin-user"
            priority: 0 # the priority in which the authorization rule will be checked. Higher priority will be checked first. optional, default = 0
          - path: ".*/admin/.*"
            method: # http methods which this rule matches with. optional, default = all methods
              - GET
            allowed: "admin-user"
            priority: 1
```

## Authorization Rules

Authorization rules are being processed in descending order of their assigned priority. Using overlapping authorization
rules with the same priority should be avoided as there is no guarantee in which order rules with the same priority will
be processed.

### Authorization requirements

* A request will only be declined, by way of a 403 response, when an authentication rule matches on path, method and
  host but not on the allowed parameter.
* If a request does match to path, method, host and allowed parameter it will directly be allowed.
* If a request does not match on path, method and host with any of the authorization rules then the request will be
  allowed.
