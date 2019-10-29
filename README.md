# Redemption Platform

## Introduction

Welcome to Iguama's Redemption Platform (from now on referred to as "The Platform") documentation. Our platform allows your customers to redeem their reward points in the digital merchant of their choice, in order to achieve that, there are some API requirements that need to be provided by any loyalty program interested in integrating our platform.

## Overall flow

The overall process is composed of three steps:
* Authentication
* Redemption
* Redemption Reversal

## Authentication

The Plaform handles user authentication through OAuth2 authorization framework, so the authorization itself happens within the Loyalty program's website, The Platform interacts with loyalty program's technology to obtain the authorization grant needed to access the user's account. Further information about OAuth2 can be found [here](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2).

The complete authentication process is described below:

1. The Platform will redirect it's users to the loyalty program's website to start the authentication flow, within this URL redirect, we will send the following parameters:

Field | Type | Description
----- | ---- | -----------
client_id | String | The Platform's clientID (assigned by the loyalty program's platform)
redirect_uri | URI | URI where the service redirect's the user-agent after an authorization code is granted
scope | Enum (READ, REDEEM) | Specifies the level of access that The Platform is requesting 


2. The Platform will handle the callback request, this request should contain at least the following information:

Field | Required | Description
----- | -------- | -----------
authorization_code | Yes | Authorization code needed to obtain the access token

3. The Platform will request an access token from the Loyalty Program's API, by passing the authorization code along with authentication details, including the following information:

Field | Type | Description
----- | ---- | -----------
client_id | String | The Platform's clientID (assigned by the loyalty program's platform)
client_secret | String | The Platform's clientSecret (assigned by the loyalty program's platform)
grant_type | Enum (AUTHORIZATION_CODE, CLIENT_CREDENTIALS) | Specifies the grant_type used to request authorization, in this case, AUTHORIZATION_CODE will be sent.

The response we expect from the API must contain at least:

Field | Type | Description
----- | ---- | -----------
access_token | String | Token needed to interact with the Loyalty Program's API to request user's information.
refresh_token | String | Token that can be used to request a new access token, once the current one has expired.
expires_in | Integer | Time to Live of the issued token in miliseconds (Other time units can be used, though miliseconds is recommened).

4. Using the access token obtained in the previous step, The Platform will get the user's basic information from the Loyalty Program's API, below you will find an example cURL call and the minimum required information:

```bash
curl -X POST -H "Authorization: Bearer ACCESS_TOKEN" "https://api.loyaltyprogram.com/me"
```

**Expected response**

```json
{
    "id": "123ABC098",
    "email": "john.doe@email.com",
    "first_name": "John",
    "last_name": "Doe",
    "available_balance": 1000
}
```

Field | Type | Description
----- | ---- | -----------
id | String | User's id within the loyalty program's platform
email | String | Member's email
first_name | String | User's first name.
last_name | String | User's last name.
available_balance | Integer | User's reward points balance.

After this API call, the authentication flow is complete.

## Redemption

Once the user has placed the order, The Platform will start the redemption process with goal of allocate the necessary funds to eCard provided.

Within our funds allocation process, we redeem the appropriated amount of points from the user's account.

In this process the redemption API is called.

Below is an example of the API call that The Platform will perform:

```bash
curl -X POST \
  https://api.loyaltyprogram.com/redemptions \
  -H 'Authorization: Bearer ACCESS_TOKEN' \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
        "member_id": "123ABC098",
        "total": "1000",
        "description": "Amazon Order #1344-2342344234-2334",
        "transaction_id": "[our transaction ID]"
    }'
```

Field | Type | Description
----- | ---- | -----------
member_id | String | User's id within the loyalty program's platform
total | Integer | Total amount of reward points to redeem.
description | String | Redemption's description.
transaction | String | Our transaction id for reference.

**Expected response**
```json
{
    "id": "123ABC098"
}
```

Field | Type | Description
----- | ---- | -----------
id | String | Id of the redemption

## Redemption Reversal

The Platform may need to revert a redemption in two cases:
1. The user requests it. (Restrictions apply)
2. The redemption is flagged as fraudulent.

In these cases, The Platform will call the redemption reversal API.

Below is an example of the API call that The Platform will perform:

```bash
curl -X DELETE -H "Authorization: Bearer ACCESS_TOKEN" "https://api.loyaltyprogram.com/redemptions/:id"
```

The parameter `:id` is the ID returned by the redeem API call.
