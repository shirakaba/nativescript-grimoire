# Monitoring the compass

We'll access the compass and print the orientation (in degrees), live.

CLLocationManager in the [NativeScript iOS runtime API declarations](https://github.com/NativeScript/NativeScript/blob/master/tns-platform-declarations/ios/objc-x86_64/objc!CoreLocation.d.ts#L327).

And the Apple [Objective-C docs](https://developer.apple.com/documentation/corelocation/cllocationmanager?language=objc):

## JS

```js
const MyCLLocationManagerDelegate = NSObject.extend(
    {
        get onLocationManagerDidUpdateHeading() {
            return this._onLocationManagerDidUpdateHeading;
        },
        set onLocationManagerDidUpdateHeading(cb) {
            this._onLocationManagerDidUpdateHeading = cb;
        },
        locationManagerDidUpdateHeading: function(manager, newHeading){
            if(!this.onLocationManagerDidUpdateHeading) return;
            this.onLocationManagerDidUpdateHeading(manager, newHeading);
        }
    },
    {
        name: "MyCLLocationManagerDelegate",
        protocols: [CLLocationManagerDelegate]
    }
);

function run(label){
    const lm = CLLocationManager.alloc().init();
    global.lm = lm;
    const delegate = new MyCLLocationManagerDelegate();
    delegate.onLocationManagerDidUpdateHeading = (manager, newHeading) => {
        label.text = "Compass position: " + newHeading.magneticHeading.toString();
    }
    lm.delegate = delegate;
    lm.startUpdatingHeading();
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


const sv = setUpBoundedVerticalStackView(design.ios.bounds);
const view = UIView.alloc().init();

const label = UILabel.alloc().init();
label.text = "Compass position: ";

const svx = setUpUnboundedHorizontalStackView();

run(label);

svx.addArrangedSubview(label);

sv.addArrangedSubview(svx);

design.ios.addSubview(sv);
```
