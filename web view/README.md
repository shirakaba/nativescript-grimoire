# `WKWebView`

Continuing on from the [`CFStringTokenizer` example](https://github.com/shirakaba/nativescript-grimoire/blob/master/text%20tokenisation/README.md) to provide sample data to load into a web view:

## JS

```js
// transliterate() function comes from the CFStringTokenizer example
const { originals, transliterations } = transliterate(
    "孫悟空是孕育自一顆位於東勝神州花果山上的靈石，牠吸納日月精華後破石而出，率領眾猴在花果山稱王，被敬拜為「美猴王」。牠因見到其它猴子老死而興起求得長生不死之法的念頭，便前往斜月三星洞拜菩提祖師為師。菩提祖師賜牠「孫悟空」之名，授他七十二般變化之法和觔斗雲。",
    "zh-Hant"
);

const htmlOpening = `<!DOCTYPE html><head><meta name="viewport" content="width=device-width, initial-scale=1"></head><body><html>`;

const rubyText = originals.reduce((acc, original, i) => {
    return acc + `<rb>${original}</rb><rt>${transliterations[i]}</rt>`
}, htmlOpening + "<p><ruby>") + "</ruby></p></body></html>";

function makeWebView(bounds){
    const webView = WKWebView.alloc().initWithFrameConfiguration(bounds, WKWebViewConfiguration.alloc().init());
    webView.autoresizingMask = UIViewAutoresizing.FlexibleWidth | UIViewAutoresizing.FlexibleHeight;
    webView.translatesAutoresizingMaskIntoConstraints = true;
    webView.backgroundColor = UIColor.alloc().initWithRedGreenBlueAlpha(0,1,0,1);

    return webView;
}

const webView = makeWebView(design.ios.bounds);
webView.loadHTMLStringBaseURL(rubyText, null);
design.ios.addSubview(webView);
```

### Swift

TODO
