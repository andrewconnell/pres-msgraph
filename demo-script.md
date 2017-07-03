# demos

https://developer.microsoft.com/en-us/graph

- check the site out


## authorization

- documentation => authorization => Azure AD v2.0 Endpoint
- app registratino portal: https://apps.dev.microsoft.com


## use graph explorer for demos

- https://graph.microsoft.com/v1.0/me 
- https://graph.microsoft.com/v1.0/me/messages
- https://graph.microsoft.com/beta/me/mailFolders/Inbox/messages/
- https://graph.microsoft.com/v1.0/me/contacts 
- https://graph.microsoft.com/v1.0/me/drive/root/children
- https://graph.microsoft.com/v1.0/me/events 


## paging results

- if `$skiptoken` is returned in previous response, can use it for next page:
  - `https://graph.microsoft.com/v1.0/users?$top=5$skiptoken=X'4453707402.....0000'`
- to go backwards, combine `$skiptoken` with `&previous-page=true`
  - `https://graph.microsoft.com/v1.0/users?$top=5$skiptoken=X'4453707.....00000'&previous-page=true`


## delta query

1. The application begins by calling a GET request with the delta function on the desired resource.
1. Microsoft Graph will send a response containing the requested resource and a state token.
    1. `nextLink`
    1. `deltaLink`
1. When the application needs to learn about changes to the resource, it makes a new request using the deltaLink URL received in step 2. This request may be made immediately after completing step 2 or when the application checks for changes.
1. Microsoft Graph returns a response describing changes to the resource since the previous request, and either a nextLink URL or a deltaLink URL.


### track message changes

- Track message changes in a folder

    ```
    GET https://graph.microsoft.com/beta/me/mailFolders/Inbox/messages/delta
    ```


## webhooks

- subscribe
  - create a webhook in zapier
  - get the url and create a call

      ```
      POST https://graph.microsoft.com/v1.0/subscriptions
      Content-Type: application/json
      {
        "changeType": "created,updated",
        "notificationUrl": "https://webhook.azurewebsites.net/notificationClient",
        "resource": "/me/mailfolders('SentItems')/messages",
        "expirationDateTime": "2017-04-06T01:00:00.0000000Z",
        "clientState": "SecretClientState"
      }
      ```

- go send an email & check for the message on zapier

## custom data with open extensions

1. Add roaming profile information

    ```
    POST https://graph.microsoft.com/beta/me/extensions
    Content-type: application/json
    {
        "@odata.type":"microsoft.graph.openTypeExtension",
        "extensionName":"com.contoso.roamingSettings",
        "theme":"dark",
        "color":"purple",
        "lang":"Japanese"
    }
    ```

1. Retrieve roaming profile information

    ```
    GET https://graph.microsoft.com/beta/me?$select=id,displayName,mail,mobilePhone&$expand=extensions
    ```

1. Change roaming profile information

    ```
    PATCH https://graph.microsoft.com/beta/me/extensions/com.contoso.roamingSettings
    Content-type: application/json
    {
        "theme":"light",
        "color":"yellow",
        "lang":"Swahili"
    }
    ```

1. Delete a user's roaming profile

    ```
    DELETE https://graph.microsoft.com/beta/me/extensions/com.contoso.roamingSettings
    ```

## custom data with schema extensions

1. View available schema extensions

    ```
    GET https://graph.microsoft.com/beta/schemaExtensions/?$filter=id eq 'graphlearn_test'
    ```

1. Register a schema extension definition that describes a training course

    ```
    POST https://graph.microsoft.com/beta/schemaExtensions
    Content-type: application/json
    {
        "id":"graphlearn_courses",
        "description": "Graph Learn training courses extensions",
        "targetTypes": [
            "Group"
        ],
        "properties": [
            {
                "name": "courseId",
                "type": "Integer"
            },
            {
                "name": "courseName",
                "type": "String"
            },
            {
                "name": "courseType",
                "type": "String"
            }
        ]
    }
    ```

1. Create a new group with extended data

    ```
    POST https://graph.microsoft.com/beta/groups
    Content-type: application/json
    {
        "displayName": "New Managers March 2017",
        "description": "New Managers training course for March 2017",
        "groupTypes": ["Unified"],
        "mailEnabled": true,
        "mailNickname": "newMan201703",
        "securityEnabled": false,
        "graphlearn_courses": {
            "courseId":"123",
            "courseName":"New Managers",
            "courseType":"Online"
        }
    }
    ```

1. Add or update custom data to an existing group

    ```
    PATCH https://graph.microsoft.com/beta/groups/dfc8016f-db97-4c47-a582-49cb8f849355
    Content-type: application/json
    Content-length: 230
    {
        "graphlearn_courses":{
            "courseId":"123",
            "courseName":"New Managers",
            "courseType":"Online"
        }   
    }
    ```

1. Get a group and its extension data

    ```
    GET https://graph.microsoft.com/beta/groups/dfc8016f-db97-4c47-a582-49cb8f849355?$select=displayName,id,description,graphlearn_courses
    ```
