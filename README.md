# Overview

LockerDome applications are made to be extremely flexible and powerful. The primary way that an application interacts with LockerDome is by creating app content, and controlling the rendering of it.

## Creating an App

In order to create an app on LockerDome, a user account must first be created. Follow these steps:

1. Ensure that all cookies for lockerdome.com are cleared because these will interfere with the other steps
2. Visit [the GlobalHack 4 test page](http://globalhack4.test.lockerdome.com)
3. Click on the blue "Join" button and create an account
4. Visit [the app creation page](http://globalhack4.test.lockerdome.com/app/create)
5. Enter a name for your application
6. Enter a URL that will serve as the base URL of iframes’ src attribute that LockerDome will generate to hand off the rendering of your app’s content to your server. Specify something that can accept a query string that will be json and url encoded because this URL will serve as the master controller for your app.
7. Submit the form. Be sure to record your app id and secret, you'll need those below.

## Creating your first piece of app content

Content creation will be handled by a RESTful API. As explained below, all serverside APIs are in the form 

```http://api.globalhack4.test.lockerdome.com/<api call>?<JSON encoded arguments>```

In order to create your first piece of content, initiate an HTTP GET request with the following URL:

```
http://api.globalhack4.test.lockerdome.com/app_create_content?{"app_id":your_app_id,"app_secret":"your_app_secret","app_data":{"fun":"times"},"name":"Some App Content","text":"Short description of your content"}
```

The response you get back should contain an id and a status of true. If you visit your profile on the lockerdome staging environment, you should see a feed item with your created content. If you click on it, it will open an overlay that creates an iframe that calls over to the server hook you setup when you created your app.

## Handling the hook from the app

If your server that should be handling the iframe request isn’t set up to handle that path yet, then you may have an iframe that has an error message of some sort. No worries, you just need to get your server to respond. If your app path was `http://example-ld-app.com/hook` then you will need to handle `http://example-ld-app.com/hook?<data>` requests. The simplest way to do this is to just return some basic HTML to start with.

Obviously, as you improve your app, it should utilize the data stored in the app content to control what it shows. For example, if you created a poll, you might store the options in the app data. If you look at the example above, you’ll see that we passed `{"fun":"times"}` as our app_data. You can use anything you want there that is valid JSON. There is a size limit, but it is on the order of a few KB. If you need to store more than that, you should use a database on your server.

## Client Side APIs

Your IFRAME will get certain information from LockerDome about its context. This will include the account id and a special app specific login token that the app can use to create app content on behalf of the user. Additionally, the app may request an increase in the height of its iframe and get updates if the user logs in while the app’s iframe is open. See the example and include the script file http://api.globalhack4.test.lockerdome.com/globalhack_app_platform.js

## Server Side APIs

The server-sided app API calls currently consist of the following:

+ [app_create_content](#app_create_content)
+ [app_destroy_content](#app_destroy_content)
+ [app_fetch_content](#app_fetch_content)
+ [app_update_content](#app_update_content)
+ [app_fetch_user_content](#app_fetch_user_content)
+ [app_get_account_name_and_slug](#app_get_account_name_and_slug)
+ [app_fetch_batch_data](#app_fetch_batch_data)

All *app* API calls require the following parameters:

| Parameter   | Type    | Description                         |
|-------------|---------|-------------------------------------|
| app_id      | int     | Your app's `app_id`                 |
| app_secret  | String  | Your app's `app_secret`             |

For the sake of this tutorial let's assume the following values:

```javascript
{
  your_user_id: 0,
  app_id: 1337,
  app_secret: "bananas",
}
```

For the tutorial, we'll walk you through [app_create_content](#app_create_content), but you'll have to figure the rest out on your own.

###app_create_content

Allows your app to create and hang content.

This API call accepts the following parameters:

| Parameter   | Type    | Required  | Description                                 |
|-------------|---------|-----------|---------------------------------------------|
| name        | String  | No        | Your app content's title. Keep it concise   |
| thumb_url   | String  | No        | An optional (but recommended) thumbnail url |
| text        | String  | No        | An optional alt-text for your content       |
| app_data    | Object  | No        | Optional data passed to your app (unique)   |

Let's make a sample request. This sample will be nicely formatted and won't be url-encoded, however, actual requests need to be url-encoded.

```json
https://api.lockerdome.com/app_create_content?
{
  "app_id": 1337,
  "app_secret": "bananas",
  "name": "Fun App Content",
  "thumb_url": "http://yourapp.com/app_thumb.png",
  "text": "Isn't my app great?",
  "app_data": {
    "data":"some_data"
  }
}
```

If all goes well, we should receive something along the lines of the following.

```json
{
  "status": true,
  "result": {
    "app_data": {
      "data": "some_data"
    },
    "app_id": 1337,
    "created_by": 0,
    "id": 42,
    "name": "Fun App Content",
    "text": "Isn't my app great?",
    "thumb_url": "http://yourapp.com/app_thumb.png"
  }
}
```

Let's take a look at the response. The first field is the status of the transaction. Hopefully it's true, if it's not, see [Handling Errors](#handling-errors). The `result` object is what we're interested in here. With LockerDome API calls, `result` is the returned data. In our case, it's the object that we've just created. Let's take a look at the fields. We won't include fields that we passed in our request.  

| Field       | Type    | Description                                                                           |
|-------------|---------|---------------------------------------------------------------------------------------|
| created_by  | int     | The `id` of the user that created the content. This defaults to the owner of the app. |
| id          | int     | The `id` of your content. It wouldn't be a bad idea to store this somewhere.          |

###app_destroy_content

Allows your app to destroy content as long as it has permission.

##### Parameters:

| Parameter   | Type    | Required  | Description                                 |
|-------------|---------|-----------|---------------------------------------------|
| content_id  | int     | Yes       | The `id` of the content to be deleted       |

##### Sample request:
```json
https://api.lockerdome.com/app_create_content?
{
  "app_id": 1337,
  "app_secret": "bananas",
  "content_id": 42 
}
```

##### Sample response:
```json
{
  "status": true
}
```

###app_fetch_content

Allows your app to fetch information about any content posted on LockerDome.

##### Parameters:

| Parameter   | Type    | Required  | Description                                 |
|-------------|---------|-----------|---------------------------------------------|
| content_id  | int     | Yes       | The `id` of the content to be fetched       | 

##### Sample request:
```json
https://api.lockerdome.com/app_create_content?
{
  "app_id": 1337,
  "app_secret": "bananas",
  "content_id": 42
}
```

##### Sample response:
```json
{
  "status": true,
  "result": {
    "app_data": {
      "data": "some_data"
    },
    "app_id": 7738711164977218,
    "created_by": 6597154458173441,
    "id": 7738813707321411,
    "name": "Fun App Content",
    "text": "Isn't my app great?",
    "thumb_url": "http://yourapp.com/app_thumb.png",
    "type": "app_content"
  }
}
```

##### Response fields:


###app_update_content

| Parameter   | Type    | Required  | Description                                 |
|-------------|---------|-----------|---------------------------------------------|
| content_id  | int     | Yes       | The `id` of the content you're updating     |
| name        | String  | No        | A new name for your content                 |
| thumb_url   | String  | No        | A new thumbnail url for your content        |
| text        | String  | No        | New alt-text for your content               |
| app_data    | Object  | No        | Updated data for this app content           |

###app_fetch_user_content

| Parameter   | Type    | Required  | Description                                 |
|-------------|---------|-----------|---------------------------------------------|
| created_by  | int     | Yes       | The `id` of the user whose content to fetch |

###app_get_account_name_and_slug

| Parameter   | Type    | Required  | Description                                 |
|-------------|---------|-----------|---------------------------------------------|
| account_ids | int[]   | Yes       | An array of account `id`s to fetch          |

###app_fetch_batch_data

| Parameter   | Type    | Required  | Description                                 |
|-------------|---------|-----------|---------------------------------------------|
| content_ids | int[]   | Yes       | An array of content `id`s to fetch          |