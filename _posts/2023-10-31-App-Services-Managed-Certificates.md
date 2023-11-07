---
title: "Create a App Services Managed Managed Certificates with Bicep"
date: 2023-10-30 00:00:00 +1000
categories: Azure
tags:
  - Azure
excerpt_separator: <!--more-->
---

# Create a App Services Managed Certificate with Bicep

An certificates in Azure App Services is bind to an host name, this can be an apex (or naked) domain (https://robertdeveen.com) or a subdomain (https://www.robertdeveen.com or https://subdomain.robertdeveen.com), or a combination of these two (for example one certificate for https://robertdeveen.com and https://www.robertdeveen.com).

To create an App Services Managed Certificate there are two ways to create a certificate with Bicep. One for a apex domain and one for an subdomain. The validation of the ownership of the domain is the main difference. To generate a certificate the certificate authority would like to validate that the domain you try to get a certificate for is yours. That you are the owner of that (sub)domain name.

As a prerequisite you need to have an App Service Plan and an App Service or Function App with a custom domain ready. The web app does not have a valid certificate already configured.

## Create a Host Name binding without a certificate

```Bicep
resource hostNameBindingWithoutCertificate 'Microsoft.Web/sites/hostNameBindings@2022-03-01' = {
  name: '${webAppName}/${customHostname}'
  properties: {
    siteName: webAppName
    hostNameType: 'Verified'
    // azureResourceType: 'Website'
    // azureResourceName: webAppName
    customHostNameDnsRecordType: 'CName'
    sslState: 'Disabled'
  }
}

```

## Create a App Services Managed Certificate

```Bicep
resource certificate 'Microsoft.Web/certificates@2022-03-01' = {
  name: '${canonicalName}-certificate'
  location: location
  properties: {
    serverFarmId: appServicePlanResourceId
    domainValidationMethod: 'cname-delegation' \\ Or "http-token"
    hostNames: [ canonicalName ]
    canonicalName: canonicalName
  }
}
```

> Note: The documentation is not clear about the meaning of the `domainValidationMethod` field, it is a string. When creating a certificate for an APEX domain, the HTTP Token (http-token) validation method is needed. When creating a certificate for an subdomain, the CName delegation method (cname-delegation) is needed. 

## Bind certificate to the host name

We need to use a module to enable the certificate on the host name, as Bicep/ARM forbids using resource with this same type-name combination twice in one deployment.

```Bicep
module hostnameBindingWithCertificate './modules/hostname-binding.Bicep' = {
  name: '${customHostname}-hostnamebinding'
  params: {
    certificateThumbprint: certificate.outputs.thumbprint
    customHostname: customHostname
    webAppName: webAppName
  }
}
```

### `./module/hostname-binding.Bicep`

```Bicep
resource hostNameBindingWithCertificate 'Microsoft.Web/sites/hostNameBindings@2022-03-01' = {
  name: '${webAppName}/${customHostname}'
  properties: {
    siteName: webAppName
    sslState: 'SniEnabled'
    thumbprint: certificateThumbprint
  }
}
```