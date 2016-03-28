# Hlsjs-wrapper

This module wraps an instance of hls.js to interface it with streamroot-p2p

** IMPORTANT: ** it's better to use babel-runtime when building this module. It makes use of Object.assign, and IE11 reports error due to the use of Symbol, although we don't make use of them

# Usage

### API

##### Constructor

The constructor expects the constructor of the Streamroot module as argument.

The instance will have the following properties:

##### P2Ploader

This is the constructor you need to override `loader`in `hls.config`.

##### createSRModule(p2pConfig, hls, Events)

Use this method to actually instantiate the p2p module, on `Hls.Events.MANIFEST_LOADING`.

parameter | description
----------|--------------
p2pConfig | Your p2p module configuration object. Check out the doc [here](https://streamroot.readme.io/docs/p2p-config)
hls       | Your instance of hls.js
Events | The Hls.Events enum
content | _(optionnal)_ a string identifying your content. Do not use character `?` as we remove the query strings from urls. **TODO: finalize spec and update doc**


### Example

```javascript
import HlsjsWrapper from "hlsjs-wrapper";
import Hls from "hls.js";
import StreamrootDownloader from "streamroot-p2p-dist";

// ...
// ...
// ...

var hlsConfig = {},
    p2pConfig = {
        streamrootKey: 'your-streamroot-key'
    };


var hlsjsWrapper = new HlsjsWrapper(StreamrootDownloader);

hlsConfig.loader = hlsjsWrapper.P2PLoader;

//Set buffer configuration params, unless they're specified
hlsConfig.maxBufferSize = hlsConfig.maxBufferSize || 0;
hlsConfig.maxBufferLength = hlsConfig.maxBufferLength || 30;

var hls = new Hls(hlsConfig);

hls.on(Hls.Events.MANIFEST_LOADING, (event, data) => {
    hlsjsWrapper.createSRModule(p2pConfig, hls, Hls.Events);
});
```

# Important notes

### xhrSetup, cookies and headers

`config.xhrSetup` is broken by this wrapper. The reason is that we override the loader for fragments, and this loader does not use xhr directly. Thus we throw if `xhrSetup` is defined.

However, we think that the overwhelming majority of the xhr configuration developpers need to do is:
- enable use of cookies
- set custom headers

We introduced a custom object in hls.js configuration object for that purpose:

```javascript
var hlsjsConfig = {
  // ... ,
  request: {
    withCredentials: true, // true | false.
    headers: [ ["X-CUSTOM-HEADER-1", value1], ["X-CUSTOM-HEADER-2", value2] ] // List of headers you want to set for your requests
  },
  // ... ,
}

```
