# String encoding

## JS

```js
function UTF8toBase64(utf8text){
    const safeString = unescape(encodeURIComponent(utf8text));
    const base64 = NSString.alloc().initWithUTF8String(safeString).dataUsingEncoding(NSUTF8StringEncoding)
    .base64EncodedStringWithOptions(NSDataBase64EncodingOptions.Encoding64CharacterLineLength);
    return base64;
}
```
