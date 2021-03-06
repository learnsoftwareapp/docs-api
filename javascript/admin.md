---
title: "Admin API Client Library"
date: "2019-01-09"
meta_title: "Javascript Admin API Client Libarary – Ghost"
meta_description: "Server-side client library for working with the Ghost Admin API. Publish your content from anywhere. Read more on Ghost Docs 👉"
keywords:
    - "headless cms"
    - "javascript"
    - "ghost api"
sidebar: "javascript"
---

Admin API keys should remain secret, and therefore this promise-based JavaScript library is designed for server-side usage only. This library handles all the details of generating correctly formed urls and tokens, authenticating and making requests.

## Working Example

```javascript
const api = new GhostAdminAPI({
  url: 'http://localhost:2368',
  key: 'YOUR_ADMIN_API_KEY',
  version: 'v2'
});

api.posts.add({
    title: 'My first draft API post',
    mobiledoc: '{\"version\":\"0.3.1\",\"atoms\":[],\"cards\":[],\"markups\":[],\"sections\":[[1,\"p\",[[0,[],0,\"My post content. Work in progress...\"]]]]}'
});
```

## Authentication

The client requires the host address of your Ghost API, an Admin API key, and a version string in order to authenticate.

- `url` - API domain, must not end in a trailing slash.
- `key` - string copied from the "Integrations" screen in Ghost Admin
- `version` - should be set to 'v2'

The `url` and `key` values can be obtained by creating a new `Custom Integration` under the Integrations screen in Ghost Admin.

![Get Ghost Admin API credentials](/images/apikey.png)

See the documentation on [Admin API authentication](/api/admin/#authentication) for more explanation.

## Endpoints

All endpoints & parameters provided to integrations by the [Admin API](/api/admin/) are supported.

```javascript

// [Stability: stable]

// Browsing posts returns Promise([Post...]);
// The resolved array will have a meta property
api.posts.browse();
api.posts.read({id: 'abcd1234'});
api.posts.add({title: 'My first API post'});
api.pages.edit({id: 'abcd1234', title: 'Renamed my post'});
api.posts.delete({id: 'abcd1234'});

// Browsing pages returns Promise([Page...])
// The resolved array will have a meta property
api.pages.browse({limit: 2});
api.pages.read({id: 'abcd1234'});
api.pages.add({title: 'My first API page'})
api.pages.edit({id: 'abcd1234', title: 'Renamed my page'})
api.pages.delete({id: 'abcd1234'});

// Uploading images returns Promise([Image...])
api.images.upload({file: '/path/to/local/file'});
```

## Publishing Example

A bare minimum example of how to create a post from HTML content, including extracting and uploading images first.

```JavaScript
const GhostAdminAPI = require('@tryghost/admin-api');
const path = require('path');

// Your API config
const api = new GhostAdminAPI({
    url: 'http://localhost:2368',
    version: 'v2',
    key: 'YOUR_ADMIN_API_KEY'
});

// Utility function to find and upload any images in an HTML string
function processImagesInHTML(html) {
    // Find images that Ghost Upload supports
    let imageRegex = /="([^"]*?(?:\.jpg|\.jpeg|\.gif|\.png|\.svg|\.sgvz))"/gmi;
    let imagePromises = [];

    while((result = imageRegex.exec(html)) !== null) {
        let file = result[1];
            // Upload the image, using the original matched filename as a reference
            imagePromises.push(api.images.upload({
                ref: file,
                file: path.resolve(file)
            }));
    }
    
    return Promise
        .all(imagePromises)
        .then(images => {
            images.forEach(image => html = html.replace(image.ref, image.url));        
            return html;            
        });
}

// Your content
let html = '<p>My test post content.</p><figure><img src="/path/to/my/image.jpg" /><figcaption>My awesome photo</figcaption></figure>';

return processImagesInHTML(html)
    .then(html => {        
        return api.posts        
            .add(
                {title: 'My Test Post', html},             
                {source: 'html'} // Tell the API to use HTML as the content source, instead of mobiledoc
            )
            .then(res => console.log(JSON.stringify(res)))
            .catch(err => console.log(err));

    })
    .catch(err => console.log(err));
```


## Installation

`yarn add @tryghost/admin-api`

`npm install @tryghost/admin-api`


### Usage

ES modules:

```javascript
import GhostAdminAPI from '@tryghost/admin-api'
```

Node.js:

```javascript
const GhostAdminAPI = require('@tryghost/admin-api');
```
