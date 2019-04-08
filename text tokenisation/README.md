# `NLTokenizer`

https://developer.apple.com/documentation/naturallanguage/nltokenizer

Simple text segmentation. Segment text into its constituent tokens and analyse the attributes of those tokens (far less powerful than `CFStringTokenizer` below).

## JS

```js
function segmentAndAnalyse(input, localeIdentifier){
    const tokenizer = NLTokenizer.alloc().initWithUnit(NLTokenUnit.Word);

    /* NLLanguage.h provides a non-exhaustive set of supported constants,
     * but any BCP-47 language tag, as a string value, is a valid input. */
    // tokenizer.setLanguage(NLLanguageSimplifiedChinese);
    tokenizer.setLanguage(localeIdentifier);
    const text = NSString.alloc().initWithUTF8String(input);
    tokenizer.string = text;
    const tokens = [];
    const attributesArr = [];

    tokenizer.enumerateTokensInRangeUsingBlock(
        NSMakeRange(0, text.length),
        function(tokenRange, attributes){
            const token = text.substringWithRange(tokenRange);
            tokens.push(token);
            attributesArr.push(attributes);
            return true;
        }
    );

    const printable = attributesArr.map((attrib) => {
        switch(attrib){
            case NLTokenizerAttributes.Numeric:
                return "numeric";
            case NLTokenizerAttributes.Symbolic:
                return "symbolic";
            case NLTokenizerAttributes.Emoji:
                return "emoji";
            default:
                return null;
        }
    });

    return {
        tokens: tokens,
        attributes: printable
    };
}

segmentAndAnalyse("I will soon be 75 🦄", "en");
```

## Swift

```swift
import NaturalLanguage

func segmentAndAnalyse(_ input: String, _ localeIdentifier: String) -> Dictionary<String, Any> {
    let tokenizer = NLTokenizer(unit: .word)
    // tokenizer.setLanguage(NLLanguage.simplifiedChinese)
    tokenizer.setLanguage(NLLanguage(rawValue: localeIdentifier))
    tokenizer.string = input
    
    var tokens: [Substring] = []
    var attributesArr: [NLTokenizer.Attributes] = []
    tokenizer.enumerateTokens(in: input.startIndex..<input.endIndex) { tokenRange, attributes in
        let token: Substring = input[tokenRange]
        tokens.append(token)
        attributesArr.append(attributes)
        return true
    }
    
    let printable: [String?] = attributesArr.map({
        switch($0){
        case NLTokenizer.Attributes.numeric:
            return "numeric"
        case NLTokenizer.Attributes.symbolic:
            return "symbolic"
        case NLTokenizer.Attributes.emoji:
            return "emoji"
        default:
            return nil
        }
    })
    
    return [
        "tokens": tokens,
        "attributes": printable
    ]
}

segmentAndAnalyse("I will soon be 75 🦄", "en")
```

# `CFStringTokenizer`

Perform powerful text tokenisation of many possible languages, including transliteration of non-Latin-based scripts.

Video demo: https://twitter.com/LinguaBrowse/status/1102202042620223488

## JS

```js
function transliterate(input, localeIdentifier){
    const inputText = NSString.alloc().initWithUTF8String(input);
    const range = __CFRangeMake(0, inputText.length);
    const targetLocale = NSLocale.alloc().initWithLocaleIdentifier(localeIdentifier);
    const tokenizer = CFStringTokenizerCreate(
        kCFAllocatorDefault,
        inputText,
        range,
        kCFStringTokenizerUnitWordBoundary,
        targetLocale
    );

    const originals = [];
    const transliterations = [];

    let tokenType = CFStringTokenizerGoToTokenAtIndex(tokenizer, 0);

    while(tokenType !== kCFStringTokenizerTokenNone){
        const attribute = CFStringTokenizerCopyCurrentTokenAttribute(
            tokenizer,
            kCFStringTokenizerAttributeLatinTranscription
        );

        if(attribute === null) break;

        const original = CFStringCreateWithSubstring(
            kCFAllocatorDefault,
            inputText,
            CFStringTokenizerGetCurrentTokenRange(tokenizer)
        );

        originals.push(original);
        transliterations.push(attribute);

        tokenType = CFStringTokenizerAdvanceToNextToken(tokenizer);
    }

    return {
        originals: originals,
        transliterations: transliterations
    };
}

transliterate(
    "孫悟空是孕育自一顆位於東勝神州花果山上的靈石，牠吸納日月精華後破石而出，率領眾猴在花果山稱王，被敬拜為「美猴王」。牠因見到其它猴子老死而興起求得長生不死之法的念頭，便前往斜月三星洞拜菩提祖師為師。菩提祖師賜牠「孫悟空」之名，授他七十二般變化之法和觔斗雲。",
    "zh-Hant"
)
```

## Swift

```swift
import CoreFoundation

func transliterate(_ input: String, _ localeIdentifier: String) -> [String] {
    let inputText: NSString = input as NSString
    let range: CFRange = CFRangeMake(0, inputText.length)
    let targetLocale: NSLocale = NSLocale(localeIdentifier: localeIdentifier)
    let tokenizer: CFStringTokenizer = CFStringTokenizerCreate(kCFAllocatorDefault, inputText as CFString, range, kCFStringTokenizerUnitWordBoundary, targetLocale)
    var originals = [String]()
    var transliterations = [String]()
    
    var tokenType: CFStringTokenizerTokenType = CFStringTokenizerGoToTokenAtIndex(tokenizer, 0)
    
    while (tokenType != .none) {
        guard let attribute: CFTypeRef = CFStringTokenizerCopyCurrentTokenAttribute(tokenizer, kCFStringTokenizerAttributeLatinTranscription) else { break }
        let original: String = CFStringCreateWithSubstring(kCFAllocatorDefault, inputText as CFString, CFStringTokenizerGetCurrentTokenRange(tokenizer)) as String
        originals.append(original)
        let transliteration: String = attribute as! String
        transliterations.append(transliteration)
        tokenType = CFStringTokenizerAdvanceToNextToken(tokenizer)
    }
    
    return transliterations
}

transliterate(
    "孫悟空是孕育自一顆位於東勝神州花果山上的靈石，牠吸納日月精華後破石而出，率領眾猴在花果山稱王，被敬拜為「美猴王」。牠因見到其它猴子老死而興起求得長生不死之法的念頭，便前往斜月三星洞拜菩提祖師為師。菩提祖師賜牠「孫悟空」之名，授他七十二般變化之法和觔斗雲。",
    "zh-Hant"
)
```
