# Graphics
Custom Indicators now has a Graphics module. We can leverage this in our own indicators to create more complex graphics. To enable graphics in your indicator, modify the object return value of _map_.
```
//...
    map(d) {
        return {
            graphics: {
                items: [
                    //GraphicsObjects go here.
                ]
            }
        }
    }
//...
```
The _items_ field within graphics tells the renderer what graphics to display. Each item must adhere to the [GraphicsObject](https://tradovate.github.io/custom-indicators/modules/graphics_graphicsresponse.html#graphicsobject) interface.

Let's imagine we want to note when a substantial gain has occured at an index on the chart. To do so we must track the last known value. Let's write a tool to help us do this:
```
function MyTracker(magicNumber) {
    function tracker(value) {
        return tracker.push(value)
    }

    tracker.push = (value) => {
        let result = value - tracker.state.last >= magicNumber
        tracker.state.last = value
        return result
    }

    tracker.reset = () => {
        tracker.state = {
            last: -1
        }
    }

    tracker.reset()

    return tracker
}
```
On push, the _tracker_ tool will compare the difference of incoming value and the last known value against a _magicNumber_ that will be set by UI parameters. The _push_ function will return true if the difference is greater than or equal to the magic number, indicating a 'substantial gain', as defined by the end user. We will use boolean result to determine whether or not to render a graphic at the point of 'substantial gain'.

Now let's build our calculator - the class that will ultimately represent our indicator.
```
const {px, du, op} = require('./tools/graphics')

class SubstantialGain {
    init() {
        this.tracker = MyTracker(this.props.magicNumber)
    }

    map(d) {
        const shouldDraw = this.tracker(d.value())
        return {
            graphics: shouldDraw && {
                items: [
                    {
                        tag: "Text",
                        key: "ex",
                        point: {
                            x: op(du(d.index()), '-', px(2)),
                            y: op(du(d.value()), '-', px(58)),
                        },
                        text: "!",
                        style: { fontSize: 18, fontWeight: "bold", fill: "#5c5" },
                        textAlignment: "centerMiddle"
                    },
                ]
            }
        }
    }

module.exports = {
    name: "substantialGain",
    description: "SubstantialGain",
    calculator: SubstantialGain,
    params: {
        magicNumber: predef.paramSpecs.number(1.00, 0.25, 0.25)
    },
    tags: ["Drawings"],
}
```
We first must initialize our tracker tool. We do so in the _init_ function of our calculator, leveraging our module's _props_. In the _map_ function, we use our tracker tool to get a boolean value for the bar currently being drawn. Finally in the return object of our _map_, we can add our first graphics item. This particular object is a [Text](https://tradovate.github.io/custom-indicators/interfaces/graphics_displayobject.text.html) object. Also, look closely at the _point_ field of our Text graphics item. We have to declare the point as a ScaleBound value, so we will use the special helper operators _op_, _du_, and _px_. _px_ allows us to declare a value in the pixel unit-space, while _du_ utilizes Domain Units. In the X axis, this is the bar index. In the Y axis, domain units are the price of the stock. The _op_ function is special - it allows us to operate on _px_ and _du_ values interchangeably. So we can do things like add absolute pixel values to domain unit values if we want (and we have done exactly that in the example code!). If we were to select our Substantial Gains indicator and supply it with a (reasonable) magic number, we should see some greenish exclamation points rendered over some of our chart's bars.

That's a great start, but we have substantially more power at our disposal. Let's start by making our graphic a bit more complex. Within our _items_ array, let's add another [GraphicsObject](https://tradovate.github.io/custom-indicators/modules/graphics_graphicsresponse.html#graphicsobject).
```
//...
{
    tag: "Shapes",
    key: 'circs',
    primitives: [
        {
            tag: 'Circle',
            radius: 10,
            center: {
                x: op(du(d.index()), '-', px(2)),
                y: op(du(d.value()), '-', px(60)),
            },
        },

    ],
    fillStyle: {
        color: "#5c5"
    }
},
//...
```
This is another type of [GraphicsObject](https://tradovate.github.io/custom-indicators/modules/graphics_graphicsresponse.html#graphicsobject), the [ContourShapes](https://tradovate.github.io/custom-indicators/interfaces/graphics_displayobject.contourshapes.html) type. This object describes one or more [Shape](https://tradovate.github.io/custom-indicators/modules/graphics_displayobject.html#shape) objects to apply a border stroke to, as defined by its _lineStyle_ field. Now we should see a circled exclamation point on bars that have had 'substantial gains'. Now that we have more than one logically grouped element, we should consider containing them as a unit. To do so, we can use the [Container](https://tradovate.github.io/custom-indicators/interfaces/graphics_displayobject.container.html) type. Replace your _items_ code with this single [GraphicsObject](https://tradovate.github.io/custom-indicators/modules/graphics_graphicsresponse.html#graphicsobject) object.
```
//...
{
    tag: 'Container',
    key: 'mediumContainer',
    children: [
        {
            tag: "Text",
            key: "ex",
            point: {
                x: op(du(d.index()), '-', px(2)),
                y: op(du(d.value()), '-', px(58)),
            },
            text: "!",
            style: { fontSize: 18, fontWeight: "bold", fill: "#5c5" },
            textAlignment: "centerMiddle"
        },
        {
            tag: "ContourShapes",
            key: 'circs',
            primitives: [
                {
                    tag: 'Circle',
                    radius: 10,
                    center: {
                        x: op(du(d.index()), '-', px(2)),
                        y: op(du(d.value()), '-', px(60)),
                    },
                },

            ],
            lineStyle: {
                lineWidth: 2,
                color: "#5c5"
            }
        },
    ],
},
//...
```
The container has one unique field, _children_, which is an array of [GraphicsObjects](https://tradovate.github.io/custom-indicators/modules/graphics_graphicsresponse.html#graphicsobject). You can leverage this grouping to apply transformations that keep these contained objects in relative space. That way you can move the whole container instead of each object. The other benefit of this is that we can leverage [VisibilityConditions](https://tradovate.github.io/custom-indicators/modules/graphics_graphicsresponse.html#visibilityconditions) over whole groups. Here's what we should see when we render this indicator:
![Visabilityconditions](https://tradovate.github.io/custom-indicators/media/Graphics1.png)
To understand our next step, let's do an experiment. First, zoom way out in the X axis - you can do so by scrolling the mouse wheel. Those circled exclamations don't scale down to fit the new bar size, instead they get clumped up and look bad. Now zoom way in on the X axis. Now because we haven't scaled up they look somewhat lacking and small. We can fix all of this by creating a variety of groups to describe each breakpoint and give them a _conditions_ field. These are the [VisibilityConditions](https://tradovate.github.io/custom-indicators/modules/graphics_graphicsresponse.html#visibilityconditions) for the object being rendered.

[VisibilityConditions](https://tradovate.github.io/custom-indicators/modules/graphics_graphicsresponse.html#visibilityconditions) consist of scalar ranges in the X or Y axis. These are measured in pixels-per-domain-unit. This works out to roughly the pixel distance between bars in the X axis, and the tick-size in pixels of an index for the given contract being calculated. What this allows us to do is render something different based on breakpoints we define. Here's an example:
```
//...
{
    tag: 'Container',
    key: 'mediumContainer',
    conditions: { //<== Added this field!
        scaleRangeX: { min: 10 },
    },
    children: [
        //... Same as defined before
    ],
},
{
    tag: 'Container',
    key: 'tinyContainer',
    conditions: {
        scaleRangeX: { min: 0, max: 10 },
    },
    children: [
        {
            tag: 'Dots',
            key: 'dots',
            dots: [
                { 
                    point: {
                        x: du(d.index()), 
                        y: op(du(d.value()), '-', px(48))
                    }, 
                    color: {
                        r: .25, 
                        g: .8, 
                        b: .25
                    } 
                }
            ],
            style: {
                lineWidth: 6,
                color: '#5c5'
            }
        },
    ],
},
//...
```
Now when the bars are tiny, and have only between 0 and 10 pixels between them, we will render a tiny dot instead of the circled exclamation.
![Customindicators](https://tradovate.github.io/custom-indicators/media/Graphics2.png)
This introduces another [GraphicsObject](https://tradovate.github.io/custom-indicators/modules/graphics_graphicsresponse.html#graphicsobject) as well - the [Dots](https://tradovate.github.io/custom-indicators/interfaces/plotter.dots.html) object. It describes one or more WebGL dots to render. At 10 pixels of space per bar or greater we draw our standard circled exclamation. Note that the _scaleRangeX_, _scaleRangeY_, and their _min_ and _max_ fields are actually optional. If omitted, we simply won't apply a bound in the ignored direction or on the ignored axis. We can of course apply something extra for large pixel distances as well.

I'd like to display the amount of the 'substantial gain' at high pixel-per-domain-unit distances. We will need to modify our tracker slightly to accomodate this - we currently discard our last value on each push. We will could provide the difference in the two values as part of the return value of our tracker's _push_ function.
```
//... in tracker, we simply change push a tiny bit
    tracker.push = (value) => {
        let number = value - tracker.state.last
        let result = number >= magicNumber
        tracker.state.last = value
        return [result, number]
    }

//... in SubstantialGains class we change our tracker's expected return to 2 destructured variables
    map(d) {
        const [shouldDraw, difference] = this.tracker(d.value());

        return {
            graphics: shouldDraw && {
                items: [
                    //We only want to add a single extra element - no need for another container
                    {
                        conditions: {
                            scaleRangeX: { min: 30 }
                        },
                        tag: "Text",
                        key: "bigText",
                        text: `+ ${difference}`,
                        point: {
                            x: op(du(d.index()), '-', px(4)),
                            y: op(du(d.value()), '-', px(84))
                        },
                        style: { fontSize: 20, fontWeight: "bold", fill: "#0bf" },
                        textAlignment: "centerMiddle"
                    },
                    //...
                ]
            }
        }
    }            
//...
```
Rendering our indicator should yield this result at big pixel-to-domain ratios:
![CustomIndicators2](https://tradovate.github.io/custom-indicators/media/Graphics3.png)
