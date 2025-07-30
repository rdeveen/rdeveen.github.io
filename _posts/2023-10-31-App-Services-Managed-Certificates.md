---
title: "Create a (free!) App Services Managed Certificates with Bicep"
date: 2023-10-30 00:00:00 +1000
categories: Azure
tags:
  - Azure
  - Bicep
excerpt_separator: <!--more-->
---
An certificates in Azure App Services is bind to an host name, this can be an apex (or naked) domain (<https://robertdeveen.com>) or a subdomain (<https://www.robertdeveen.com> or <https://subdomain.robertdeveen.com>), or a combination of these two (for example one certificate for <https://robertdeveen.com> and <https://www.robertdeveen.com>).

To create an App Services Managed Certificate there are two ways to create a certificate with Bicep. One for a apex domain and one for an subdomain. The validation of the ownership of the domain is the main difference. To generate a certificate the certificate authority would like to validate that the domain you try to get a certificate for is yours. That you are the owner of that (sub)domain name.

<!--more-->

As a prerequisite you need to have an App Service Plan and an App Service or Function App with a custom domain ready. The Web App does not have a valid certificate already configured.

## Create a Host Name binding without a certificate

{% highlight bicep %}
param customHostname string = 'www.robertdeveen.com'
param webAppName string = 'robertdeveen'

resource hostNameBindingWithoutCertificate 'Microsoft.Web/sites/hostNameBindings@2022-03-01' = {
  name: '${webAppName}/${customHostname}'
  properties: {
    siteName: webAppName
    hostNameType: 'Verified'
    azureResourceType: 'Website'
    azureResourceName: webAppName
    customHostNameDnsRecordType: 'CName'
    sslState: 'Disabled'
  }
}
{% endhighlight %}

## Create a App Services Managed Certificate for a subdomain

### Create a CName record

In the DNS Zone, create a CName record pointing to *.azurewebsite.net. This is needed for the validation of the ownership of the domain.

| Type | Record | Value |
| --- | --- | --- |
| CNAME | www.robertdeveen.com | robertdeveen.azurewebsites.net |

### Create a App Services Managed Certificate

{% highlight bicep %}
param canonicalName string = 'www.robertdeveen.com'
param appServicePlanName string = 'robertdeveen-plan'
param location string = 'westeurope'

resource appServicePlan 'Microsoft.Web/serverfarms@2022-03-01' existing =  {
  name: appServicePlanName
}

resource certificate 'Microsoft.Web/certificates@2022-03-01' = {
  name: '${canonicalName}-certificate'
  location: location
  properties: {
    serverFarmId: appServicePlanResourceId
    // domainValidationMethod: 'cname-delegation' // or 'http-token'
    hostNames: [ canonicalName ]
    canonicalName: canonicalName
  }
}
{% endhighlight %}

> Note: The documentation is not clear about the meaning of the `domainValidationMethod` field, it is a string. But the value that is accepted should be `cname-delegation` or `http-token`. Other values give the error message: **"The parameter Properties.DomainValidationMethod has an invalid value."**
The value `cname-delegation` is the only one working these days. The value `http-token` is not working anymore and just waiting a long time to end. The best solution is to not add that field.

## Create a App Services Managed Certificate for an apex domain

### Create a A record

In the DNS Zone, create an A records pointing to the IP address of the webapp. This is needed for the validation of the ownership of the domain.

| Type | Record | Value |
| --- | --- | --- |
| A | robertdeveen.com | IP-address-of-webapp |

{% highlight bicep %}
param canonicalName string = 'robertdeveen.com'
param location string = 'westeurope'

resource nakedCertificate 'Microsoft.Web/certificates@2022-09-01' = {
  name: '${canonicalName}-certificate'
  location: location
  properties: {
    serverFarmId: appServicePlan.id
    canonicalName: canonicalName
  }
}
{% endhighlight %}

## Create a App Services Managed Certificate for an apex and subdomain

**THIS DOESN'T WORK!** The documentation is not clear about this, but it is not possible to create a certificate for an apex and subdomain at the same time. You need to create two certificates, one for the apex and one for the subdomain.

{% highlight bicep %}
// param canonicalName string = 'robertdeveen.com'
// param hostNames = [ 'www.robertdeveen.com', 'robertdeveen.com' ]
// param location string = 'westeurope'

// resource nakedCertificate 'Microsoft.Web/certificates@2022-09-01' = {
//   name: '${canonicalName}-certificate'
//   location: location
//   properties: {
//     serverFarmId: appServicePlan.id
//     hostNames: [ hostNames ]
//     canonicalName: canonicalName
//   }
// }
{% endhighlight %}

## Bind certificate to the host name

We need to use a module to enable the certificate on the host name, as Bicep/ARM forbids using resource with this same type-name combination twice in one deployment.

{% highlight bicep %}
module hostnameBindingWithCertificate './modules/hostname-binding.Bicep' = {
  name: '${customHostname}-hostnamebinding'
  params: {
    certificateThumbprint: certificate.outputs.thumbprint
    customHostname: customHostname
    webAppName: webAppName
  }
}
{% endhighlight %}

### `./module/hostname-binding.Bicep`

{% highlight bicep %}
resource hostNameBindingWithCertificate 'Microsoft.Web/sites/hostNameBindings@2022-03-01' = {
  name: '${webAppName}/${customHostname}'
  properties: {
    siteName: webAppName
    sslState: 'SniEnabled'
    thumbprint: certificateThumbprint
  }
}
{% endhighlight %}
