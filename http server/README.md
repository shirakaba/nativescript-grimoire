# Run a HTTP server (`GCDWebserver`)

Note: requires installing the `nativescript-http-server` plugin, which has been taken down for some reason.

It used to be available from here: https://plugins.nativescript.rocks/plugin/nativescript-http-server

## JS

```js
/** Start up the web server **/
const ws = GCDWebServer.alloc().init();
ws.addGETHandlerForBasePathDirectoryPathIndexFilenameCacheAgeAllowRangeRequests(
    "/",
    NSURL.alloc().initWithString(
        NSBundle.mainBundle.pathForResourceOfTypeInDirectory(
            "index", "html", "tsserver"
        )
    ).URLByDeletingLastPathComponent.absoluteString,
    null,
    3600,
    true
);
ws.startWithPortBonjourName(8080, null);
/*****************************/

/** Navigate to the statically-served website **/
function makeWebView(bounds){
    const webView = WKWebView.alloc().initWithFrameConfiguration(bounds, WKWebViewConfiguration.alloc().init());
    webView.autoresizingMask = UIViewAutoresizing.FlexibleWidth | UIViewAutoresizing.FlexibleHeight;
    webView.translatesAutoresizingMaskIntoConstraints = true;
    webView.backgroundColor = UIColor.alloc().initWithRedGreenBlueAlpha(0,1,0,1);

    return webView;
}
const webView = makeWebView(design.ios.bounds);
design.ios.addSubview(webView);
webView.loadRequest(
    NSURLRequest.alloc().initWithURL(
        NSURL.alloc().initWithString("http://localhost:8080/index.html")
    )
);
/*****************************/
```
