# Integrating Okay to your server

 To proceed with Okay integration, you are required to create an account using this [link](https://okaythis.com/signup). Once you are successfully signed up, login in with the credentials that you used to create the account [here](https://demostand.okaythis.com/multi-tenant-admin/login).

 Once you are logged in to your [dashboard](https://demostand.okaythis.com/pss-admin/dashboard) click on **Tenants** in the top toolbar, then select [**tenants**](https://demostand.okaythis.com/multi-tenant-admin/tenants) from the drop down menu.

 ![Dashboard Toolbar Image](/images/toolbar-tenants.png)

 The [**Tenants**](https://demostand.okaythis.com/multi-tenant-admin/tenants) web page is where you will register your server as an SPS that will communicate with Okay servers in order to verify/intiate secure transactions/authentications. You tenant page should present a table that looks like the table below.

 ![Tenant Table Image](/images/tenant-dashboard.png)

## Overview of the Tenant web table

 As you can see, you already have an entry in the **Tenants** table. The contents of that row are essential in understanding how to integrate Okay into your server.

### Tenant ID

 The first column under the table is what we refer to as your **Tenant ID** referring to the image above, my **Tenant Id** is `40007`. It is very important that you take note of this value as we will be using this value for our transactions/authentication.

### Name

 The text under the **Name** column in the **Tenants** table is the name of the company you provided at the time of your sign up. 

### Status

### Trial Expiration

### Actions

![Action Column Image](/images/tenants-action-column.png)

The **Action** column has three button that allows us to edit our tenant credentials.

## Adding Credentials to your Tenant

To make our tenant useful we will be adding more information to the tenant to connect properly/securely to Okay servers. Click on the pencil icon under **Actions** to complete the tenant registration.

![Edit Tenant Image](/images/edit-tenant.png)

To be able to recieve feedbacks from Okay servers you will need to add a valid callback url (A callback url is an endpoint on your server that will be used as a point of communication by **Okay** to notify your server about the status of transactions/authentication) to the **Callback** input field and also generate a secret secure token(or secret) that will be used to verify all transactions by **Okay** secure servers. The tokens could be any aphanumeric secure string that contains between 0-30 characters and must be kept secret. 

**Note:** we will be referring to our **Token** as **secret** in further illustration.

**LINKING USERS**
===============

Before we can authorize transactions using **Okay** we need to link our users to **Okay** so that we can identify all transactions coming from different users.

### Provide a Unique Value Generator

Before we proceed to linking your users to **Okay**. We need to generate and store a **Unique Identifier** for every end-user in order to differentiate all your users. You can use any alpha-numeric character to compose this value, for example a **UUID**. We will be using this value in all our requests as **User Unique Identifier**. This normally serves as value to the **"userExternalId"** key in our payload.

### ***A typical structure of our JSON payload for linking users***

```JSON
  {
    "tenantId": "<your tenant id>",
    "userExternalId": "User Unique Identifier",
    "signature": "BASE64[SHA256(tenantId | userExternalId | secret)]"
  }
```

The `tenantId` key in our payload above, is the **ID** we get from out ***Tenants*** table. Please refer to **Integrating Okay to your Server** section of this document. 

The `userExternalId` key in our palyload above is the **User Unique Identifier** you created for your users in order to differentiate them as described in the **Provide a Unique Value Generator** section of this documentation above.

The `signature` key in our payload above is a hash that is generated from concatenating your `tenantId` + `userExternalId` + `secret` (also know as the **Token** you added to your tenant) then passing the concatenated string as value to `SHA256()` algorithm. Then we encode whatever value or string we get from the `SHA256()` algorithm in `BASE64`.

```js
  const PSS_BASE_URL = 'https://demostand.okaythis.com';
  const TENANT_ID = 40007;
  const USER_EXTERNAL_ID = 'uid406jkt';
  const SECRET = 'securetoken';

  const hashStr = `${TENANT_ID}${USER_EXTERNAL_ID}${SECRET}`;
  const signature = base64(sha256(hashStr));
  ...
```

If all is set, we proceed to linking our user. To link a user we need to send our JSON payload as a **POST** request to this endpoint `https://demostand.okaythis.com/gateway/link`.

```js
  const PSS_BASE_URL = 'https://demostand.okaythis.com';
  const TENANT_ID = 40007;
  const USER_EXTERNAL_ID = 'uid406jkt';
  const SECRET = 'securetoken';

  const hashStr = `${TENANT_ID}${USER_EXTERNAL_ID}${SECRET}`;
  const signature = base64(sha256(hashStr));

  const payload = {
    tenantId,
    userExternalId,
    signature
  }

  fetch(`${PSS_BASE_URL}/gateway/link`, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify(payload)
  })
  .then(response => console.log(response.json));
```

When your request is correct you'll get a response with the following body:

```JSON
  {
    "status": {
        "code": 0,
        "message": "OK"
    },
    "linkingCode": "unique short-living code",
    "linkingQrImg": "base64-encoded image of QR code"
  }
```

For better reference to all possible status code and messages you can recieve from **Okay** server please refer to this [link](https://github.com/Okaythis/okay-example/wiki/Server-Response-Status-Codes).

## Authenticate User/Authorize User Action

After Linking a user, we can now authenticate that user or authorize the user's action.

Just like linking a user, we will be sending a JSON payload as a **POST** request to **Okay** using this link `https://demostand.okaythis.com/gateway/auth`.

### ***A typical structure of our JSON payload for authenticating users/authorizing user actions***

```JSON
  {
    "tenantId": "<your tenant id>",
    "userExternalId": "User Unique Identifier",
    "type": "<Authorization type>",
    "authParams": {
        "guiText": "message that is shown in the Okay application",
        "guiHeader": "header of the message that is shown in the Okay application"
    },
    "signature": "BASE64[SHA256(tenantId | userExternalId | secret)]"
  }
```

For this request, we will be adding two new fields, the `type` and `authParams` fields.

The `type` key in our JSON payload is a field that allows us to clearly specify the kind of authorization/authentication we choose to initiate. The `type` key can take as value any of these values listed below.

- "AUTH_OK"
- "AUTH_PIN"
- "AUTH_PIN_TAN"
- "AUTH_PIN_PROTECTORIA_OK"
- "GET_PAYMENT_CARD"
- "ENROLLMENT"
- "ENROLLMENT_PROTECTORIA_OK"
- "UNKNOWN"

The `authParams` just contains a message and its header that will be displayed on the Okay App. The message is intended for the user to read, in order to grant Okay the required permission to complete a transaction/authentication.

We can now proceed to sending our request to `Okay` like so.

```js

  const PSS_BASE_URL = 'https://demostand.okaythis.com';
  const TENANT_ID = 40007;
  const USER_EXTERNAL_ID = 'uid406jkt';
  const SECRET = 'securetoken';

  const hashStr = `${TENANT_ID}${USER_EXTERNAL_ID}${SECRET}`;
  const signature = base64(sha256(hashStr));
  const authParams = {
        "guiText": 'Do you okay this transaction',
        "guiHeader": 'Authorization requested'
  };
  const type = 'AUTH_OK';

  const payload = {
    tenantId,
    userExternalId,
    type,
    authParams,
    signature
  }

  fetch(`${PSS_BASE_URL}/gateway/auth`, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify(payload)
  })
  .then(response => console.log(response.json));

```

When your request is correct you'll get a response with the following body structure:

```
{
  "status": {
    "code": <status code>,
    "message": "status message"
  },
  "sessionExternalId": "unique session identifier"
}
```

For better reference to all possible status code and messages you can recieve from **Okay** server please refer to this [link](https://github.com/Okaythis/okay-example/wiki/Server-Response-Status-Codes).

