# Live speech recognition via `Speech.framework`

We'll transcribe speech live using `SFSpeechRecognizer`.

Video demo: https://twitter.com/LinguaBrowse/status/1108447428326408192

## JS

```js
class Transcriber {
    constructor(localeIdentifier){
        this.speechRecognizer = SFSpeechRecognizer.alloc().initWithLocale(NSLocale.alloc().initWithLocaleIdentifier(localeIdentifier));
        this.audioEngine = AVAudioEngine.alloc().init();
        this.request = SFSpeechAudioBufferRecognitionRequest.alloc().init();
        this.mostRecentlyProcessedSegmentDuration = 0;
        this.started = false;

        // Customisable
        this.onTranscriptionUpdate = (text) => {};
        this.onSegment = (text) => {};
        this.onLog = (text) => {};
        this.onError = (error) => {};
    }

    launchRecordingRequest(){
        SFSpeechRecognizer.requestAuthorization((authStatus) => {
            switch(authStatus){
                case SFSpeechRecognizerAuthorizationStatus.Authorized:
                    this.startRecording();
                    break;
                case SFSpeechRecognizerAuthorizationStatus.Denied:
                case SFSpeechRecognizerAuthorizationStatus.Restricted:
                case SFSpeechRecognizerAuthorizationStatus.NotDetermined:
                    break;
            }
        });
    }

    startRecording(){
        this.onLog("startRecording.");
        this.mostRecentlyProcessedSegmentDuration = 0;
        this.onTranscriptionUpdate("");

        // 1
        const node = this.audioEngine.inputNode;
        node.installTapOnBusBufferSizeFormatBlock(
            0,
            1024,
            node.outputFormatForBus(0),
            (buffer, time) => {
                this.request.appendAudioPCMBuffer(buffer);
            }
        );

        // 3
        this.onLog("Preparing audio engine.");
        this.audioEngine.prepare();

        this.onLog("Starting recording.");
        try {
	    /* Interestingly, any timeout placed here seems to never get called; or at least it appears to have no effect. */
            // setTimeout(
            //     () => {
            //         this.onLog("Stopping recording (outside of stopRecording()).");
            //         this.stopRecording();
            //     },
            //     3000
            // );
            this.audioEngine.startAndReturnError();
        } catch(error){
            // Problem starting recording.
            this.onError(error);
            return;
        }

        this.started = true;

        this.onLog("Launching recognition task.");
        this.recognitionTask = this.speechRecognizer.recognitionTaskWithRequestResultHandler(
            this.request,
            (result, error) => {
                if(error !== null){
                    this.onError(error);
                    return;
                }
                const transcription = result.bestTranscription;
                if(transcription !== null){
                    this.updateUIWithTranscription(transcription);
                }
            }
        );
    }

    stopRecording(){
        this.onLog("Stopping recording.");
        this.audioEngine.stop();
        this.audioEngine = null;
        this.request.endAudio();
        this.request = null;
        if(this.recognitionTask){
            this.recognitionTask.cancel();
            this.recognitionTask = null;
        }
        this.started = false;
    }
    
    updateUIWithTranscription(transcription){
        // self.transcriptionOutputLabel.text = transcription.formattedString
        this.onTranscriptionUpdate(transcription.formattedString);

        const lastSegment = transcription.segments.lastObject;
        if(lastSegment && lastSegment.duration > this.mostRecentlyProcessedSegmentDuration){
            this.mostRecentlyProcessedSegmentDuration = lastSegment.duration;
            this.onSegment(lastSegment.substring);
        }
    }
}

function makeTextView(bounds){
    const tv = UITextView.alloc().initWithFrame(bounds);
    tv.autoresizingMask = UIViewAutoresizing.FlexibleWidth | UIViewAutoresizing.FlexibleHeight;
    tv.translatesAutoresizingMaskIntoConstraints = true;
    tv.backgroundColor = UIColor.alloc().initWithRedGreenBlueAlpha(0,1,0,1);

    return tv;
}

// NEW
function makeTextField(bounds){
    const tv = UITextField.alloc().initWithFrame(bounds);
    tv.autoresizingMask = UIViewAutoresizing.FlexibleWidth | UIViewAutoresizing.FlexibleHeight;
    tv.translatesAutoresizingMaskIntoConstraints = true;
    tv.backgroundColor = UIColor.alloc().initWithRedGreenBlueAlpha(1,1,1,1);

    return tv;
}

function makeButton(bounds){
    // const button = UIButton.alloc().initWithFrame(bounds);
    const button = UIButton.buttonWithType(UIButtonType.System);
    button.frame = bounds;
    button.autoresizingMask = UIViewAutoresizing.FlexibleWidth | UIViewAutoresizing.FlexibleHeight;
    button.translatesAutoresizingMaskIntoConstraints = true;
    button.backgroundColor = UIColor.alloc().initWithRedGreenBlueAlpha(0,1,0,1);

    return button;
}

const designOrigin = design.ios.bounds.origin;
const designSize = design.ios.bounds.size;
const sections = 6;
const transcriptionTv = makeTextView(
    CGRectMake(designOrigin.x, designOrigin.y, designSize.width, designSize.height / sections)
);
const segmentTv = makeTextView(
    CGRectMake(designOrigin.x, designOrigin.y + designSize.height / sections, designSize.width, designSize.height / sections)
);
const errorTv = makeTextView(
    CGRectMake(designOrigin.x, designOrigin.y + 2 * (designSize.height / sections), designSize.width, designSize.height / sections)
);
const logTv = makeTextView(
    CGRectMake(designOrigin.x, designOrigin.y + 3 * (designSize.height / sections), designSize.width, designSize.height / sections)
);
const tf = makeTextField(
    CGRectMake(designOrigin.x, designOrigin.y + 4 * (designSize.height / sections), designSize.width, designSize.height / sections)
);
tf.text = “ja”;
global.tf = tf;
const button = makeButton(
    CGRectMake(designOrigin.x, designOrigin.y + 5 * (designSize.height / sections), designSize.width, designSize.height / sections)
);
const MyTapHandlerImpl = NSObject.extend(
    {
        get button() { return this._button; },
        set button(x) { this._button = x; },
        get tv() { return this._tv; },
        set tv(x) { this._tv = x; },
        get cb() { return this._cb; },
        set cb(x) { this._cb = x; },
        tap: function(responder){
            if(!this.button) return;
            if(!this.tv) return;
            if(!this.cb) return;
            this.cb(responder);
        }
    },
    {
        name: "MyTapHandlerImpl",
        protocols: [],
        exposedMethods: {
            tap: { returns: interop.types.void, params: [UIButton] }
        }
    }
);


transcriptionTv.text = "[Transcriptions]";
segmentTv.text = "[Segments]";
errorTv.text = "[Errors]";
logTv.text = "[Logs]";
button.setTitleForState("Start recording", UIControlState.Normal);

function initialiseTranscriber(localeId){
const transcriber = new Transcriber(localeId);
transcriber.onTranscriptionUpdate = (text) => {
    transcriptionTv.text = "[Transcriptions] " + text;
};
transcriber.onSegment = (text) => {
    segmentTv.text = "[Segments] " + text;
};
transcriber.onLog = (text) => {
    logTv.text = "[Logs] " + text;
};
transcriber.onError = (error) => {
    if(error instanceof NSError){
        // Ignore the error that is purely due to the recording being forcibly stopped.
        if(error.localizedDescription.indexOf("216") > -1) return;
        errorTv.text = "[Errors] " + error.localizedDescription;
    } else {
        errorTv.text = "[Errors] " + error.toString();
    }
};

return transcriber;
}

const tapHandler = new MyTapHandlerImpl();
tapHandler.button = button;
tapHandler.tv = logTv;
tapHandler.cb = (responder) => {
    if(!global.transcriber){
        global.transcriber = initialiseTranscriber(global.tf.text);
    }
    if(transcriber.started){
        logTv.text = "[Logs] " + "Recording stopped.";
        transcriber.stopRecording();
        global.transcriber = null;
        button.setTitleForState("Start recording", UIControlState.Normal);
    } else {
        logTv.text = "[Logs] " + "Recording...";
        button.setTitleForState("Stop recording", UIControlState.Normal);
        transcriber.launchRecordingRequest();
    }
};
button.addTargetActionForControlEvents(tapHandler, "tap", UIControlEvents.TouchUpInside);

design.ios.addSubview(transcriptionTv);
design.ios.addSubview(segmentTv);
design.ios.addSubview(errorTv);
design.ios.addSubview(logTv);
design.ios.addSubview(tf);
design.ios.addSubview(button);
```

## Swift

Adapted from the Ray Wenderlich [Speech Recognition Tutorial for iOS](https://www.raywenderlich.com/573-speech-recognition-tutorial-for-ios).
