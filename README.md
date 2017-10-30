# A basic demo of Apigee Edge features

## Introducton
This is a sample [Apigee Edge](https://apigee.com/api-management) API proxy that demonstrates the use of some basic features.

## Prerequisites
- You need an Apigee Edge developer account. See [docs](http://docs.apigee.com/api-services/content/creating-apigee-edge-account) for more details on how to setup your account.
- Optionally, you'll need [Maven](https://maven.apache.org/download.cgi) to use [Apigee Deploy Maven Plugin](https://github.com/apigee/apigee-deploy-maven-plugin) to install code from this repo to your Edge org/env.

## Install
The code in this repo is in an intermediate state of the intended demo. You can either follow the steps in demo to setup the proxy, products, apps, etc. or install the intermediate state directly with the below instructions.


`git clone` or download this repository.

Set your Apigee Edge username and password in environment variables.

```bash
$ set EDGE_USERNAME=<Apigee Edge Username>
$ set EDGE_PASSWORD=<Apigee Edge Password>
```

Deploy the proxy.

```bash
$ cd apibin-v1
$ mvn install -P test -Dusername=$EDGE_USERNAME -Dpassword=$EDGE_PASSWORD -Dorg=<Your Apigee Edge Org> -Denv=<Your Apigee Edge Env>
```

Deploy the configuration items like target server, KVM, products, apps, etc.

```bash
$ cd config
$ mvn install -P test -Dapigee.config.options=create -Dusername=$EDGE_USERNAME -Dpassword=$EDGE_PASSWORD -Dorg=<Your Apigee Edge Org> -Denv=<Your Apigee Edge Env>
```


## Demo

The code in this repo is in an intermediate state (upto step 5 below) to help with the demo.

1. Create a reverse proxy with taget as http://httpbin.org

2. Test APIs like `https://org-test.apigee.net/v1/apibin/ip`, `https://org-test.apigee.net/v1/apibin/get`, `https://org-test.apigee.net/v1/apibin/headers`, `https://org-test.apigee.net/v1/apibin/basic-auth/:user/:passwd`

3. Create a target server for env `demo-target` pointing to `httpbin.org` on port 80

4. Change proxy target config to:

```xml
    <HTTPTargetConnection>
        <LoadBalancer>
            <Server name="demo-target"/>
        </LoadBalancer>
        <Path>/</Path>
    </HTTPTargetConnection>
```

5. Implement basic authentication using /basic-auth/:username/:password path suffix.
- Create a KVM entry with username and password
- Create a new target endpoint

```xml
<TargetEndpoint name="basic-auth">
    <Description/>
    <FaultRules/>
    <PreFlow name="PreFlow"/>
    <PostFlow name="PostFlow"/>
    <Flows/>
    <HTTPTargetConnection>
        <LoadBalancer>
            <Server name="demo-target"/>
        </LoadBalancer>
    </HTTPTargetConnection>
</TargetEndpoint>
```

- Create a route rule for `basic-auth` path

```xml
    <RouteRule name="basic-auth">
        <TargetEndpoint>basic-auth</TargetEndpoint>
        <Condition>(proxy.pathsuffix MatchesPath "/basic-auth")</Condition>
    </RouteRule>
```

- In the proxy read KVM values

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<KeyValueMapOperations async="false" continueOnError="false" enabled="true" name="KVM-GetTargetCreds" mapIdentifier="httpbin">
    <DisplayName>KVM-GetTargetCreds</DisplayName>
    <Get assignTo="private.creds.username" index="1">
        <Key>
            <Parameter>username</Parameter>
        </Key>
    </Get>
    <Get assignTo="private.creds.password" index="1">
        <Key>
            <Parameter>password</Parameter>
        </Key>
    </Get>
    <Scope>environment</Scope>
</KeyValueMapOperations>
```

- Update target request with username and password as basic authentication header

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<BasicAuthentication async="false" continueOnError="false" enabled="true" name="BA-TargetAuthN">
    <DisplayName>BA-TargetAuthN</DisplayName>
    <Operation>Encode</Operation>
    <IgnoreUnresolvedVariables>false</IgnoreUnresolvedVariables>
    <User ref="private.creds.username"/>
    <Password ref="private.creds.password"/>
    <AssignTo createNew="false">request.header.Authorization</AssignTo>
</BasicAuthentication>
```

- Update target path with username and password variables

```xml
    <HTTPTargetConnection>
        <LoadBalancer>
            <Server name="demo-target"/>
        </LoadBalancer>
        <Path>/basic-auth/{private.creds.username}/{private.creds.password}</Path>
    </HTTPTargetConnection>
```

- Add JavaScript policy to not add path suffix to target URL

```javascript
context.setVariable("target.copy.pathsuffix", false);
context.setVariable("target.copy.queryparams", false);
```


6. Move KVM extraction and basic authentication header update to a shared flow

7. Add conditional flow for `/get` path and JSONToXML policy to display `/get` results in XML instead of JSON.

8. Create a product, developer and app. Add verify API key policy and test

9. Add spike arrest policy and test

10. Add quota policy and test

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Quota async="false" continueOnError="false" enabled="true" name="Quota-1" type="calendar">
    <DisplayName>Quota-1</DisplayName>
    <Properties/>
    <Allow count="5"/>
    <Interval>1</Interval>
    <TimeUnit>minute</TimeUnit>
    <Distributed>true</Distributed>
    <Synchronous>true</Synchronous>
    <StartTime>2017-10-28 12:00:00</StartTime>
</Quota>
```


## Note
All of the material here is released under the [MIT license](https://github.com/gahana/edge-trg-labs/blob/master/LICENSE)

## Support
If you have any questions, search and/or ask questions on [community](https://community.apigee.com)
