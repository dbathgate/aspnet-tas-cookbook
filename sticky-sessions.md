# Sticky Sessions

## Overview
* Enabling sticky session in your TAS app creates a persisted session between the user and the original server the load balancer routed them to
* Use this feature when user session state is stored on the application and requires the user to stay on the same app instance

## **DISCLAIMER**
* Use of sticky sessions is consider a cloud **ANTI-PATTERN**
* Using sticky sessions will prevent the application from being dynamically scaled
* Applications also cannot support zero downtime or blue/green deployments
* Any change in application instances (whether it be scale up/down, restart, or restage) will disrupt user sessions
* Consider using a pattern such as [Redis Session Store](redis-session-store.md) instead for a cloud-native approach to session management

## Enabling Sticky Sessions
* By Default, TAS will automatically start sticky sessions if the application creates a cookie named `JSESSIONID`
* Once this cookie is created, TAS will create an additional cookie named `__VCAP_ID__`
* The VCAP ID is the ID of the application instance to send traffic to

### Using ASP.NET Session
* Change the ASP.NET Session cookie name to `JSESSIONID`
#### Edit `Web.config`
```xml
<sessionState cookieName="JSESSIONID" />
```

## Changing the default cookie name
* Adding more sticky session cookie names has to be configured in OpsMan by a operator of TAS
* Every TAS installation has `JSESSIONID` as the default
