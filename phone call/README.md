# Dialling a phone number

Note: Acts with the usual native-level restrictions, i.e. the user must accept a system prompt before the phone call will be initiated.

Video demo: https://twitter.com/LinguaBrowse/status/1102184387645505536

## JS

```js
function dialNumber(number){
    const url = NSURL.URLWithString(`tel://${number}`);
    if(url === null) return console.error("URL invalid.");
    if(UIApplication.sharedApplication.canOpenURL(url)){
        // WARNING: I assume that I'm targeting iOS 10 and above here.
        UIApplication.sharedApplication.openURLOptionsCompletionHandler(
            url,
            NSDictionary.alloc().init(),
            null
        );
    } else {
        return console.error("Can't open URL.");
    }
}

dialNumber("01234567890");
```

## Swift

```swift
func dialNumber(number: String) {
    guard let url = URL(string: "tel://\(number)") else { return print("URL invalid.") }
    guard UIApplication.shared.canOpenURL(url) else { return print("Can't open URL.") }

    if #available(iOS 10, *) {
        UIApplication.shared.open(url, options: [:], completionHandler:nil)
    } else {
        UIApplication.shared.openURL(url)
    }
}

dialNumber("01234567890")
```
