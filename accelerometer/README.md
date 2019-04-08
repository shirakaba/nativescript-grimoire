# `CMMotionManager`

Access the accelerometer. Here, I update UIProgressViews to display the values.

Video demo: https://twitter.com/LinguaBrowse/status/1102272013845057536

## JS

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

## Swift

TODO
