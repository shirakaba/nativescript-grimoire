# Assigning an action to a `UIButton`

Clicking the button will cause the text view to change its text.

## JS

```js
function makeTextView(bounds){
    const tv = UITextView.alloc().initWithFrame(bounds);
    tv.autoresizingMask = UIViewAutoresizing.FlexibleWidth | UIViewAutoresizing.FlexibleHeight;
    tv.translatesAutoresizingMaskIntoConstraints = true;
    tv.backgroundColor = UIColor.alloc().initWithRedGreenBlueAlpha(0,1,0,1);

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

const sections = 2;
const logTv = makeTextView(
    CGRectMake(designOrigin.x, designOrigin.y + 0 * (designSize.height / sections), designSize.width, designSize.height / sections)
);
logTv.text = "Log text view.";
const button = makeButton(
    CGRectMake(designOrigin.x, designOrigin.y + 1 * (designSize.height / sections), designSize.width, designSize.height / sections)
);
button.setTitleForState("Toggle recording", UIControlState.Normal);
const MyTapHandlerImpl = NSObject.extend(
    {
        get button() { return this._button; },
        set button(aButton) { this._button = aButton; },
        get tv() { return this._tv; },
        set tv(aTv) { this._tv = aTv; },
        tap: function (args){
            if(!this.button) return;
            if(!this.tv) return;
            this.tv.text = "BUTTON PRESSED";
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
const tapHandler = new MyTapHandlerImpl();
tapHandler.button = button;
tapHandler.tv = logTv;
button.addTargetActionForControlEvents(tapHandler, "tap", UIControlEvents.TouchUpInside);
design.ios.addSubview(logTv);
design.ios.addSubview(button);
```
