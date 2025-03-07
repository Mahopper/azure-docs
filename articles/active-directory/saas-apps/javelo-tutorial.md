---
title: 'Tutorial: Azure AD SSO integration with Javelo'
description: Learn how to configure single sign-on between Azure Active Directory and Javelo.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 06/06/2022
ms.author: jeedes

---

# Tutorial: Azure AD SSO integration with Javelo

In this tutorial, you'll learn how to integrate Javelo with Azure Active Directory (Azure AD). When you integrate Javelo with Azure AD, you can:

* Control in Azure AD who has access to Javelo.
* Enable your users to be automatically signed-in to Javelo with their Azure AD accounts.
* Manage your accounts in one central location - the Azure portal.

## Prerequisites

To get started, you need the following items:

* An Azure AD subscription. If you don't have a subscription, you can get a [free account](https://azure.microsoft.com/free/).
* Javelo single sign-on (SSO) enabled subscription.
* Along with Cloud Application Administrator, Application Administrator can also add or manage applications in Azure AD.
For more information, see [Azure built-in roles](../roles/permissions-reference.md).

## Scenario description

In this tutorial, you configure and test Azure AD SSO in a test environment.

* Javelo supports **SP** initiated SSO.
* Javelo supports **Just In Time** user provisioning.

> [!NOTE]
> Identifier of this application is a fixed string value so only one instance can be configured in one tenant.

## Add Javelo from the gallery

To configure the integration of Javelo into Azure AD, you need to add Javelo from the gallery to your list of managed SaaS apps.

1. Sign in to the Azure portal using either a work or school account, or a personal Microsoft account.
1. On the left navigation pane, select the **Azure Active Directory** service.
1. Navigate to **Enterprise Applications** and then select **All Applications**.
1. To add new application, select **New application**.
1. In the **Add from the gallery** section, type **Javelo** in the search box.
1. Select **Javelo** from results panel and then add the app. Wait a few seconds while the app is added to your tenant.

## Configure and test Azure AD SSO for Javelo

Configure and test Azure AD SSO with Javelo using a test user called **B.Simon**. For SSO to work, you need to establish a link relationship between an Azure AD user and the related user in Javelo.

To configure and test Azure AD SSO with Javelo, perform the following steps:

1. **[Configure Azure AD SSO](#configure-azure-ad-sso)** - to enable your users to use this feature.
    1. **[Create an Azure AD test user](#create-an-azure-ad-test-user)** - to test Azure AD single sign-on with B.Simon.
    1. **[Assign the Azure AD test user](#assign-the-azure-ad-test-user)** - to enable B.Simon to use Azure AD single sign-on.
1. **[Configure Javelo SSO](#configure-javelo-sso)** - to configure the single sign-on settings on application side.
    1. **[Create Javelo test user](#create-javelo-test-user)** - to have a counterpart of B.Simon in Javelo that is linked to the Azure AD representation of user.
1. **[Test SSO](#test-sso)** - to verify whether the configuration works.

## Configure Azure AD SSO

Follow these steps to enable Azure AD SSO in the Azure portal.

1. In the Azure portal, on the **Javelo** application integration page, find the **Manage** section and select **single sign-on**.
1. On the **Select a single sign-on method** page, select **SAML**.
1. On the **Set up single sign-on with SAML** page, click the pencil icon for **Basic SAML Configuration** to edit the settings.

   ![Screenshot shows to edit Basic S A M L Configuration.](common/edit-urls.png "Basic Configuration")

1. On the **Basic SAML Configuration** section, Upload the **Service Provider metadata file** which you can download from the `https://api.javelo.io/omniauth/<CustomerSPIdentifier>_saml/metadata` and perform the following steps:

	a. Click **Upload metadata file**.

    ![Screenshot shows Basic SAML Configuration with the Upload metadata file link.](common/upload-metadata.png "Folder")

	b. Click on **folder logo** to select the metadata file and click **Upload**.

	![Screenshot shows a dialog box where you can select and upload a file.](common/browse-upload-metadata.png "Logo")

	c. Once the metadata file is successfully uploaded, the necessary URLs get auto populated automatically.

	d. In the **Sign-on URL** text box, type a URL using the following pattern:
    `https://<CustomerSubdomain>.javelo.io/auth/login`

    > [!NOTE]
	> This value is not real. Update this value with the actual Sign-on URL. Contact [Javelo Client support team](mailto:Support@javelo.io) to get these values. You can also refer to the patterns shown in the **Basic SAML Configuration** section in the Azure portal.

1. On the **Set up single sign-on with SAML** page, In the **SAML Signing Certificate** section, click copy button to copy **App Federation Metadata Url** and save it on your computer.

	![Screenshot shows the Certificate download link.](common/copy-metadataurl.png "Certificate")

### Create an Azure AD test user

In this section, you'll create a test user in the Azure portal called B.Simon.

1. From the left pane in the Azure portal, select **Azure Active Directory**, select **Users**, and then select **All users**.
1. Select **New user** at the top of the screen.
1. In the **User** properties, follow these steps:
   1. In the **Name** field, enter `B.Simon`.  
   1. In the **User name** field, enter the username@companydomain.extension. For example, `B.Simon@contoso.com`.
   1. Select the **Show password** check box, and then write down the value that's displayed in the **Password** box.
   1. Click **Create**.

### Assign the Azure AD test user

In this section, you'll enable B.Simon to use Azure single sign-on by granting access to Javelo.

1. In the Azure portal, select **Enterprise Applications**, and then select **All applications**.
1. In the applications list, select **Javelo**.
1. In the app's overview page, find the **Manage** section and select **Users and groups**.
1. Select **Add user**, then select **Users and groups** in the **Add Assignment** dialog.
1. In the **Users and groups** dialog, select **B.Simon** from the Users list, then click the **Select** button at the bottom of the screen.
1. If you are expecting a role to be assigned to the users, you can select it from the **Select a role** dropdown. If no role has been set up for this app, you see "Default Access" role selected.
1. In the **Add Assignment** dialog, click the **Assign** button.

## Configure Javelo SSO

1. Log in to your Javelo company site as an administrator.

1. Go to **Admin** view and navigate to **SSO** tab > **Azure Active Directory** and click **Configure**.

1. In the **Enable SSO with Azure Active Directory** page, perform the following steps:

    ![Screenshot that shows the Configuration Settings.](./media/javelo-tutorial/settings.png "Configuration")

    a. Enter a valid name in the **Provider** textbox.

    b.  In the **Entity ID** textbox, paste the **Azure AD Identifier** value which you have copied from the Azure portal.

    c. In the **Metadata URL** textbox, paste the **App Federation Metadata Url** which you have copied from the Azure portal.

    d. Click **Test URL**.

    e. Enter a valid domain in the **Email Domains** textbox.

    f. Click **Enable SSO with Azure Active Directory**.

### Create Javelo test user

In this section, a user called B.Simon is created in Javelo. Javelo supports just-in-time user provisioning, which is enabled by default. There is no action item for you in this section. If a user doesn't already exist in Javelo, a new one is created after authentication.

## Test SSO 

In this section, you test your Azure AD single sign-on configuration with following options. 

* Click on **Test this application** in Azure portal. This will redirect to Javelo Sign-on URL where you can initiate the login flow. 

* Go to Javelo Sign-on URL directly and initiate the login flow from there.

* You can use Microsoft My Apps. When you click the Javelo tile in the My Apps, this will redirect to Javelo Sign-on URL. For more information about the My Apps, see [Introduction to the My Apps](https://support.microsoft.com/account-billing/sign-in-and-start-apps-from-the-my-apps-portal-2f3b1bae-0e5a-4a86-a33e-876fbd2a4510).

## Next steps

Once you configure Javelo you can enforce session control, which protects exfiltration and infiltration of your organization’s sensitive data in real time. Session control extends from Conditional Access. [Learn how to enforce session control with Microsoft Defender for Cloud Apps](/cloud-app-security/proxy-deployment-any-app).
