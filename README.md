# Background

## What is this?

These are snippets of JavaScript, assuming NativeScript bindings to the iOS native APIs, that allow one to invoke native iOS functionality. 📲

Equivalent (or similar) Swift code is provided where available. NativeScript is based on Obj-C interfaces, but I show Swift because I don't have any familiarity with Obj-C 🤷‍♂️

You can try these out by building a new NativeScript project. But I myself use [NS:IDE](https://github.com/shirakaba/nside), my open-source NativeScript run-time IDE.

## How do you discover these snippets?

First, I write the desired native code in Swift. Then I try rewriting that same code line-by-line in NS:IDE, paying attention to the auto-complete suggestions to determine the correct method name (in JS) for each call is. Referencing the official Apple API documentation (ideally with 'Objective-C' selected as the language, which maps better to the NativeScript bindings) helps a lot here.

I do also refer to the incredible [`tns-platform-declarations`](https://github.com/NativeScript/NativeScript/tree/master/tns-platform-declarations/ios/objc-x86_64) when I get stuck. One day, it would be great if NS:IDE could have a TypeScript service to ease this process.

You can also start a TypeScript project of your own and use those platform declarations to get Intellisense. I just use NS:IDE to skip the source-rebuilding step (NS:IDE allows you to test the code at run-time).

# Examples

## Dialling a phone number

Note: Acts with the usual native-level restrictions, i.e. the user must accept a system prompt before the phone call will be initiated.

### JS

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

### Swift

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

## `AVCaptureDevice`

Toggle the torch to 'on' (if off), or vice versa. Requires the device to have a torch of course, so won't do anything in the simulator.

Video here: https://twitter.com/LinguaBrowse/status/1088548651247513602

### JS

```js
function toggleTorch(){
    const device = AVCaptureDevice.defaultDeviceWithMediaType(AVMediaTypeVideo);
    if(device === null) return;
    if(!device.hasTorch) return;
    
    try {
        device.lockForConfiguration();

        if(device.torchMode == AVCaptureTorchMode.On){
            device.torchMode = AVCaptureTorchMode.Off;
        } else {
            try {
                device.setTorchModeOnWithLevelError(1.0);
            } catch(error){
                console.error("Could not set torch mode to 'on'.", error);
            }
        }
        device.unlockForConfiguration();
    } catch(error) {
        console.error("Torch could not be used.", error);
    }
}

toggleTorch();
```

### Swift

From: https://stackoverflow.com/questions/27207278/how-to-turn-flashlight-on-and-off-in-swift

```swift
import AVFoundation

func toggleTorch() {
    guard let device = AVCaptureDevice.default(for: AVMediaType.video) else { return }
    guard device.hasTorch else { return }

    do {
        try device.lockForConfiguration()

        if device.torchMode == .on {
            device.torchMode = .off
        } else {
            do {
                try device.setTorchModeOn(level: 1.0)
            } catch {
                print(error)
            }
        }

        device.unlockForConfiguration()
    } catch {
        print("Torch could not be used")
    }
}

toggleTorch()
```

## `AVSpeechSynthesizer`

See: https://nshipster.com/avspeechsynthesizer/

Call the text-to-speech functions.

### JS

```js
/* This overload is also valid: */
// const utterance = AVSpeechUtterance.speechUtteranceWithString(“Hello, world!”);

const utterance = AVSpeechUtterance.alloc().initWithString(“Hello, world!”);
utterance.voice = AVSpeechSynthesisVoice.voiceWithLanguage(“en-GB”);

AVSpeechSynthesizer.alloc().init().speakUtterance(utterance);
```

### Swift

```swift
import AVFoundation

let utterance = AVSpeechUtterance(string: “Hello, world!”)
utterance.voice = AVSpeechSynthesisVoice(language: "en-GB")

AVSpeechSynthesizer().speakUtterance(utterance)
```

### JS (advanced)

We'll live-highlight the words of a UILabel (inserted into the native view wrapped by a NativeScript View in NS:IDE: `design`) as they're being read out by text-to-speech.

Under construction. Currently uses a lot of globals while I figure out the minimum amount of JS-retained references to native objects needed to get this working with the delegate.

```js
function makeLabel(text, bounds){
    const label = UILabel.alloc().initWithFrame(bounds);
    label.text = text;
    label.numberOfLines = 0;
    label.adjustsFontSizeToFitWidth = false;
    label.autoresizingMask = UIViewAutoresizing.FlexibleWidth | UIViewAutoresizing.FlexibleHeight;
    label.translatesAutoresizingMaskIntoConstraints = true;
    label.backgroundColor = UIColor.alloc().initWithRedGreenBlueAlpha(0,1,0,1);

    return label;
}

function speak(text, language, delegate){
    const nsstr = NSString.alloc().initWithUTF8String(text);
    const utterance = AVSpeechUtterance.alloc().initWithString(nsstr);
    utterance.voice = AVSpeechSynthesisVoice.voiceWithLanguage(language);
    const synthesizer = AVSpeechSynthesizer.alloc().init();
    synthesizer.delegate = delegate;
    synthesizer.speakUtterance(utterance);
}

// ES5-compatible class declaration
const MyAVSpeechSynthesizerDelegate = NSObject.extend(
    {
        /* I'd prefer to assign these in a constructor, but I'm unsure how to write it! */
        get label() {
            return this._label ? this._label.get() : undefined;
        },
        set label(aLabel) {
            /* Note: I don't guarantee that WeakRef is necessary/appropriate here! */
            this._label = new WeakRef(aLabel);
        },
        speechSynthesizerWillSpeakRangeOfSpeechStringUtterance: function (
            synthesizer,
            characterRange,
            utterance
        ){
            if(!this.label) return;

            const mut = NSMutableAttributedString.alloc().initWithString(utterance.speechString);
            const dict = {};
            dict[NSForegroundColorAttributeName] = UIColor.alloc().initWithRedGreenBlueAlpha(1,1,0,1);
            mut.setAttributesRange(dict, characterRange);
            this.label.attributedText = mut;
        },

        speechSynthesizerDidFinishSpeechUtterance: function (synthesizer, utterance){
            if(!this.label) return;

            this.label.attributedText = NSAttributedString.alloc().initWithString(utterance.speechString);
        }
    },
    {
        name: "MyAVSpeechSynthesizerDelegate",
        protocols: [AVSpeechSynthesizerDelegate]
    }
);

const text = "孫悟空是孕育自一顆位於東勝神州花果山上的靈石，牠吸納日月精華後破石而出，率領眾猴在花果山稱王，被敬拜為「美猴王」。";
const label = makeLabel(text, design.ios.bounds);
/* CAUTION: you may need to assign the UILabel to global to retain it (the delegate keeps a WeakRef)! */
// global.label = label;
design.ios.addSubview(label);
const delegate = new MyAVSpeechSynthesizerDelegate();
delegate.label = label;
/* CAUTION: you may need to assign the delegate to global to retain it! */
// global.myAVSpeechSynthesizerDelegate = delegate;
speak(label.text, "zh-CN", delegate);
```

### Swift (advanced)

```swift
import AVFoundation

let text = "It's pronounced 'tomato'"

let mutableAttributedString = NSMutableAttributedString(string: text)
let range = NSString(string: text).range(of: "tomato")
let pronunciationKey = NSAttributedString.Key(rawValue: AVSpeechSynthesisIPANotationAttribute)

// en-US pronunciation is /tə.ˈme͡ɪ.do͡ʊ/
mutableAttributedString.setAttributes([pronunciationKey: "tə.ˈme͡ɪ.do͡ʊ"], range: range)

let utterance = AVSpeechUtterance(attributedString: mutableAttributedString)

// en-GB pronunciation is /tə.ˈmɑ.to͡ʊ/... but too bad!
utterance.voice = AVSpeechSynthesisVoice(language: "en-GB")

let synthesizer = AVSpeechSynthesizer()
synthesizer.speak(utterance)

// TODO: UILabel.
// See: https://www.hackingwithswift.com/example-code/media/how-to-highlight-text-to-speech-words-being-read-using-avspeechsynthesizer
```

## `NLTokenizer`

https://developer.apple.com/documentation/naturallanguage/nltokenizer

Simple text segmentation. Segment text into its constituent tokens and analyse the attributes of those tokens (far less powerful than `CFStringTokenizer` below).

### JS

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

### Swift

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

## `CFStringTokenizer`

Perform powerful text tokenisation of many possible languages, including transliteration of non-Latin-based scripts.

### JS

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

### Swift

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

## `WKWebView`

Continuing on from the previous `CFStringTokenizer` example:

### JS

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

const webView = makeWebView(design.ios.bounds);
webView.loadHTMLStringBaseURL(rubyText, null);
design.ios.addSubview(webView);
```

### Swift

TODO

## `CMMotionManager`

Access the accelerometer. Here, I update UIProgressViews to display the values.

### JS

```js
function groove(xProg, yProg, zProg){
    const mm = CMMotionManager.alloc().init();
    mm.startAccelerometerUpdates();
    setInterval(() => {
        const data = mm.accelerometerData;
        if(data === null) return “no data”;
        const acceleration = data.acceleration;
        if(acceleration === null) return “no accel”;
        // Range is -1 to +1
        xProg.setProgressAnimated((acceleration.x + 1) / 2, true);
        yProg.setProgressAnimated((acceleration.y + 1) / 2, true);
        zProg.setProgressAnimated((acceleration.z + 1) / 2, true);
    }, 100);
}

function setUpBoundedVerticalStackView(bounds){
    const sv = UIStackView.alloc().initWithFrame(
        CGRectMake(
            bounds.origin.x + 8,
            bounds.origin.y + 8,
            bounds.size.width - 16,
            bounds.size.height - 16
        )
    );
    sv.autoresizingMask = UIViewAutoresizing.FlexibleWidth | UIViewAutoresizing.FlexibleHeight;
    sv.translatesAutoresizingMaskIntoConstraints = true;
    sv.axis = UILayoutConstraintAxis.Vertical;
    sv.alignment = UIStackViewAlignment.Center;
    sv.distribution = UIStackViewDistribution.EqualSpacing;
    sv.spacing = 8.0;
    sv.isLayoutMarginsRelativeArrangement = true;
    return sv;
}

function setUpUnboundedHorizontalStackView(){
    const sv = UIStackView.alloc().init();
    sv.axis = UILayoutConstraintAxis.Horizontal;
    sv.alignment = UIStackViewAlignment.Leading;
    sv.distribution = UIStackViewDistribution.EqualSpacing;
    sv.spacing = 8.0;
    sv.layoutMargins = UIEdgeInsetsMake(8, 8, 8, 8);
    sv.isLayoutMarginsRelativeArrangement = true;
    return sv;
}

function setUpProgressView(w, h){
    const prog = UIProgressView.alloc().initWithProgressViewStyle(UIProgressViewStyle.Bar);
    prog.frame = CGRectMake(0, 0, w, h);
    prog.progressTintColor = UIColor.alloc().initWithRedGreenBlueAlpha(1,1,1,1);
    prog.trackTintColor = UIColor.alloc().initWithRedGreenBlueAlpha(0,0,0,1);
    prog.widthAnchor.constraintEqualToConstant(w).active = true;
    prog.heightAnchor.constraintEqualToConstant(h).active = true;
    return prog;
}


const sv = setUpBoundedVerticalStackView(design.ios.bounds);
const labelX = UILabel.alloc().init();
labelX.text = "X:";
const labelY = UILabel.alloc().init();
labelY.text = "Y:";
const labelZ = UILabel.alloc().init();
labelZ.text = "Z:";

const svx = setUpUnboundedHorizontalStackView();
const svy = setUpUnboundedHorizontalStackView();
const svz = setUpUnboundedHorizontalStackView();
const xProg = setUpProgressView(200, 10);
const yProg = setUpProgressView(200, 10);
const zProg = setUpProgressView(200, 10);

groove(xProg, yProg, zProg);

svx.addArrangedSubview(labelX);
svx.addArrangedSubview(xProg);
svy.addArrangedSubview(labelY);
svy.addArrangedSubview(yProg);
svz.addArrangedSubview(labelZ);
svz.addArrangedSubview(zProg);

sv.addArrangedSubview(svx);
sv.addArrangedSubview(svy);
sv.addArrangedSubview(svz);

design.ios.addSubview(sv);
```

### Swift

TODO
