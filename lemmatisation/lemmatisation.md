## JS

```js
function lemmatise(text, localeIdentifier) {
    // Not sure what the consequences of inputting a (UTF-16) JS string, but is necessary for setting orthography...
    const textAsNSString = NSString.alloc().initWithUTF8String(text);
    const tagSchemes = NSLinguisticTagger.availableTagSchemesForLanguage(localeIdentifier);
    if(!tagSchemes.containsObject(NSLinguisticTagSchemeLemma)){
        throw new Error("Lemmatisation not supported for language");
    }
    
    const scheme = NSLinguisticTagSchemeLemma;
    // 10 bytes as UTF-16; 5 bytes as UTF-8. tagger.setOrthographyRange() explodes in UTF16.
    const range = { location: 0, length: textAsNSString.lengthOfBytesUsingEncoding(NSUTF8StringEncoding) };

    const options = NSLinguisticTaggerOptions.OmitWhitespace | NSLinguisticTaggerOptions.OmitPunctuation;

    const tagger = NSLinguisticTagger.alloc().initWithTagSchemesOptions(
        [NSLinguisticTagSchemeLemma],
        options,
    );

    tagger.string = textAsNSString
    tagger.setOrthographyRange(
        NSOrthography.defaultOrthographyForLanguage(localeIdentifier),
        range
    );
    
    /* Not every word will have a lemma available */
    const lemmas = [];
    const surfaces = [];

    tagger.enumerateTagsInRangeSchemeOptionsUsingBlock(
        range,
        scheme,
        options,
        function(tag, tokenRange, sentenceRange, stop){
            if(tag){
                lemmas.push(
                    NSString.alloc().initWithUTF8String(tag).lowercaseStringWithLocale(
                        NSLocale.alloc().initWithLocaleIdentifier(localeIdentifier)
                    )
                );
            } else {
                /* Most NSString methods return a JS string, so unfortunately we can't chain them. */
                const token = textAsNSString.substringWithRange(tokenRange);
                const lowercasedToken = NSString.alloc().initWithUTF8String(token).lowercaseStringWithLocale(
                    NSLocale.alloc().initWithLocaleIdentifier(localeIdentifier)
                );

                surfaces.push(lowercasedToken);
            }
        }
    );
    
    return { lemmas, surfaces };
}

lemmatise("the quick brown fox jumps over the lazy Zalgo", "en");
```
