# `AVSpeechSynthesizer`

See: https://nshipster.com/avspeechsynthesizer/

Call the text-to-speech functions.

Video demo: https://twitter.com/LinguaBrowse/status/1101943350037544963

## JS

```js
/* This overload is also valid: */
// const utterance = AVSpeechUtterance.speechUtteranceWithString(“Hello, world!”);

const utterance = AVSpeechUtterance.alloc().initWithString(“Hello, world!”);
utterance.voice = AVSpeechSynthesisVoice.voiceWithLanguage(“en-GB”);

AVSpeechSynthesizer.alloc().init().speakUtterance(utterance);
```

## Swift

```swift
import AVFoundation

let utterance = AVSpeechUtterance(string: “Hello, world!”)
utterance.voice = AVSpeechSynthesisVoice(language: "en-GB")

AVSpeechSynthesizer().speakUtterance(utterance)
```

## JS (advanced)

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

## Swift (advanced)

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
