# Environment details

**Last tested:** 05-NOV-2025
**ORDS version used:** 24.4
**Jetty version used:** unknown
**Java version used:** unknown

## BIG-IP F5 settings for routing HTTP traffic through ORDS

### Question

What are the recommended BIG-IP F5 settings for routing HTTP traffic through ORDS, where:
- no SSL,
- in an Oracle EBS-integrated environment?

### Answer

Two settings for the upstream traffic:
- Set the Host header to the HTTPS hostname so that generated links produced by ORDS contain the hostname that the client used.
- Set the `X-Forwarded-Proto` header value to `https`

#### ORDS configuration settings

The relevant configuration setting for your ORDS instances. Set the ORDS `security.httpsHeaderCheck` configuration setting: 

```shell
<copy>
ords config set security.httpsHeaderCheck "X-Forwarded-Proto: http"
</copy>
```

When the ORDS configuration setting `security.httpsHeaderCheck` is set to `X-Forwarded-Proto: http` this tells ORDS that it can consider the transport (request) secure, *even though it is listening for HTTP traffic*.[^1] Because the request was originally received by the Load Balancer over HTTPS.

[^1]: [About](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/X-Forwarded-Proto) the `X-Forwarded-Proto` header.

#load-balancer #lb #load-balance #httpsheadercheck #x-forwarded-proto #configuration #network #https #header #headers #F5 #f5 #big-ip #http-traffic