---
title: Publish SharePoint Add-ins
description: Decide where to publish your SharePoint Add-ins.
ms.date: 12/09/2019
ms.prod: sharepoint
ms.localizationpriority: high
---
# Publish SharePoint Add-ins

You've finished developing your SharePoint Add-in—the final step is to make it available to your users. You can do this by publishing the add-in to one of the following:

- **AppSource** - Publish your add-in to AppSource to make it publically available, so that it can be acquired by users of any SharePoint deployment.
- **An internal organization add-in catalog** - Publish your add-in to an internal organization add-in catalog, hosted on your SharePoint deployment, to make it available to users who access that SharePoint deployment.

For information about how to package your add-in for publication by using Visual Studio 2012, see [Publish SharePoint Add-ins by using Visual Studio](publish-sharepoint-add-ins-by-using-visual-studio.md).

## Publishing to AppSource

To publish an add-in to AppSource, you must first [open a developer account](/office/dev/store/open-a-developer-account).

When you upload an add-in to AppSource for publication, Microsoft performs a validation check. For example, it checks that the add-in is free of viruses and that the add-in manifest markup is valid and complete, and verifies that any SharePoint solution packages (.wsp files) that you included in the add-in do not contain elements that aren't allowed, or SharePoint features with a scope that is broader than web. The package is also inspected for objectionable content. If the add-in package passes validation, it's wrapped into a file and signed by Microsoft.

> [!NOTE]
> Pricing model management is not supported for Office marketplace products. Existing paid products that migrated from Seller Dashboard will need to move to a SaaS model or be made free by July 2020. For details, see [Moving from paid to free add-ins](/office/dev/store/moving-from-paid-to-free-addins). You can monetize your add-in through the Microsoft Commercial Marketplace; for details, see [Monetize your add-in](/office/dev/store/monetize-addins-through-microsoft-commercial-marketplace).

## Publishing to an add-in catalog

If you're creating SharePoint Add-ins for your own company's use or a specific corporate client, instead of the general public, you'll likely want to publish your add-in to an internal add-in catalog hosted on SharePoint. A private add-in catalog is a dedicated site collection in a SharePoint web application (or a SharePoint Online tenancy) that hosts document libraries for SharePoint Add-ins and Office Add-ins. Putting the catalog into its own site collection makes it easier for the web application administrator or tenant administrator to limit permissions to the catalog.

Uploading a SharePoint Add-in to a corporate add-in catalog is as easy as uploading any file to a SharePoint document library. You fill out a pop-up form in which you supply the local URL of the add-in package and other information, such as the name of the add-in. When you upload the add-in to an add-in catalog, there are similar checks, and add-ins that do not pass are marked as invalid or disabled in the catalog.

## Deciding where to publish your SharePoint Add-in

The following table offers a comparison of publishing to AppSource or to an add-in catalog, and lists issues to consider when deciding where to publish your add-in. We recommend you decide where you plan to publish your add-in before you design and develop it; in some cases, such as licensing, where you publish your add-in will affect the design and development of your add-in.

**Table 1. Considerations for where to publish your add-in**

|**AppSource**|**Add-in Catalog**|
|:-----|:-----|
|Add-in is publicly available.|Add-in is available to users with access to this SharePoint deployment.|
|Licensing framework is available.|Licensing framework is not available for use.|
|Add-in package is verified by Microsoft for technical and content adherence to policies.|Add-in package verification is performed by SharePoint when add-in is uploaded.|
|You must be signed up with Partner Center to upload add-ins.|No registration with Microsoft is required.|

## See also

- [Make your solutions available in AppSource and within Office](/office/dev/store/submit-to-appsource-via-partner-center)
- [Create or update client IDs and secrets in the Seller Dashboard](/office/dev/store/create-or-update-client-ids-and-secrets)
- [Submit your Office solution to AppSource via Partner Center](/office/dev/store/use-partner-center-to-submit-to-appsource)
- [Validation policies for apps and add-ins submitted to AppSource](/office/dev/store/validation-policies)
- [Get started creating SharePoint-hosted SharePoint Add-ins](get-started-creating-sharepoint-hosted-sharepoint-add-ins.md)
- [Get started creating provider-hosted SharePoint Add-ins](get-started-creating-provider-hosted-sharepoint-add-ins.md)
- [Deploying and installing SharePoint Add-ins: methods and options](deploying-and-installing-sharepoint-add-ins-methods-and-options.md)
- [Tenancies and deployment scopes for SharePoint Add-ins](tenancies-and-deployment-scopes-for-sharepoint-add-ins.md)
- [Publish SharePoint Add-ins by using Visual Studio](publish-sharepoint-add-ins-by-using-visual-studio.md)