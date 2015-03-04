# youtube-video-api [![Build Status](https://api.travis-ci.org/h2non/youtube-video-api.svg?branch=master)][travis] [![NPM version](https://img.shields.io/npm/v/youtube-video-api.svg)][npm]

Simple **node.js/io.js programmatic interface to** easily **upload and delete videos** from **YouTube** using [OAuth2](https://developers.google.com/accounts/docs/OAuth2) and [Google API v3](https://developers.google.com/youtube/v3/docs/videos)

It was designed to provide a reliable server-to-server automation solution, with additional support for  Google API OAuth2 token automatically retrieval using [PhantomJS](http://phantomjs.org)

**Note**: this is still a beta implementation, don't use it for serious stuff

## Installation

```bash
npm install youtube-video-api --save
```

## Configuration

If you already have a valid OAuth2 token, you can create in your working directory a file called `.google-oauth2-credentials.json` to store it.

This will avoid you to define your Google account email and password to automatically obtain a valid OAuth2 token using [PhantomJS](http://phantomjs.org), however if you want to get a fresh OAuth2 token automatically on-the-fly read [this](#google-oauth)

Example file:
```json
{
  "access_token": "xx99.xxxxxxxxx-xxxxxxxxxxx_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "token_type": "Bearer",
  "refresh_token": "1/xxxxxxxxxxxxxxxxxx_xxxxx-xxxxxxxxxx_xxxxxxxxx_xxxxxxxxx",
  "expiry_date": 1425349408683
}
```

Only `access_token` and `refresh_token` fields are mandatory.



## Usage

```js
var Youtube = require('youtube-video-api')
```

```js
var youtube = Youtube({ 
  video: {
    part: 'status,snippet' 
  }
})

youtube.on('auth:success', uploadVideo)

youtube.authenticate('my-client-id', 'my-client-secret')

function uploadVideo() {
  var params = {
    resource: {
      snippet: {
        title: 'test video',
        description: 'This is a test video uploaded via the YouTube API'
      },
      status: {
        privacyStatus: 'private'
      }
    }
  }

  youtube.upload('path/to/video.mp4', params, function (err, video) {
    if (err) {
      return console.error('Cannot upload video:', err)
    }

    console.log('Video was uploaded with ID:', video.id)
  
    // this is just a test! delete it
    youtube.delete(video.id, function (err) {
      if (!err) console.log('Video was deleted')
    })
  })
}
```

<!--
.on('progress', function (percent) {
  console.log('% uploaded:', percent)
})
-->
 
## API

### youtube([ options ])

Youtube Videos API constructor. Returns an evented API based on `EventEmitter` subscribers

Supported options:

- **saveTokens** `boolean` - Save OAuth tokens in `.google-oauth2-credentials.json`. Default `true`
- **video** `object` - Default video options to send to the API. Documentation [here](https://developers.google.com/youtube/v3/docs/videos)
- **email** `string` - Optional. Google Account email login required obtain a valid OAuth2 token. You can pass it as env variable `GOOGLE_LOGIN_EMAIL`
- **password** `string` - Optional. Google Account password login required to obtain a valid OAuth2 token. You can pass it as env variable `GOOGLE_LOGIN_PASSWORD`

#### youtube#upload(video [, callback ])
Alias: `insert`

Upload a new video with custom metadata

You can see all the allowed params [here](https://developers.google.com/youtube/v3/docs/videos/insert)

#### youtube#delete(id [, callback ])

Delete a video, passing its ID as `string`

#### youtube#authenticate(clientId, clientSecret [, token ])

Authorize the client to perform read/write API operations. You **must call this method** on each new Youtube client 

#### youtube#on(event, fn)

Dispached events:

- **error** `error` - Fired when some error happend
- **auth:success** `token` - Fired when the client was authorized successfully

## Google OAuth

### Automatically obtain a valid OAuth2 token

Be sure you have a project and a Web Application credentials with a Client ID and Client Secret 
from the [Google API Console][console] > `API & Auth` > `Credentials`

Then you must add the following URI as allowed redirects (without final slash):
```
http://localhost:8488
```

Then you should see something like:

<img src="http://oi59.tinypic.com/2w3udmd.jpg" />

Example getting a valid OAuth token on-the-fly:
```js
var youtube = Youtube({ 
  video: {
    part: 'status,snippet'
  },
  email: 'john@gmail.com',
  password: 'svp3r_p@s$p0rd'
})

youtube.on('auth:success', function (err) {
  if (!err) {
    youtube.upload('path/to/video.mp4', {}, function (err, video) {
      if (!err) console.log('Video was uploaded:', video.id)
    })
  }
})

youtube.authenticate('my-client-id', 'my-client-secret')
```

## License

[MIT](http://opensource.org/licenses/MIT) © Tomas Aparicio

[console]: https://code.google.com/apis/console/
[travis]: https://travis-ci.org/h2non/youtube-video-api
[npm]: http://npmjs.org/package/youtube-video-api
