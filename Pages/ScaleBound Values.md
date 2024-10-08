# ScaleBound Values
When we define or calculate measurements in chart space when using Custom Indicators or Custom Drawing Tools, we need to consider the unit type that we are working with. We use two typical unit types when developing custom Tradovate tools:

- _px_: the _px_ unit is for pixels. When we want to define something with an absolute pixel unit value, we use _px_.
- _du_: the _du_ unit stands for _domain units_. In the X axis, domain units are the index of the bar, while in the Y axis they are the price of the asset.

We provide helper functions to work with the Scale Bound values expected by anything that requires a [Point](https://tradovate.github.io/custom-indicators/interfaces/canvas.point.html) type object. We can import those functions into our Custom Indicator or Custom Drawing file like so:
```
const { px, du, op, min, max } = require('./tools/graphics')
//...
```
Let's review each of these functions and their use.

- _px_ and _du_: Simply defines a Scale Bound value in either _px_ or _du_ unit spaces respectively.
- _op_: allows for operation between _px_ or _du_ Scale Bound values. This allows us to write code like so - _const myVal = op(px(16), '-', du(4525))_ - this would result in a value equal to 16 pixels above the 4525 price, given that this is used for a _y_ coordinate.
- _min_: chooses the lesser of a value. Ex. _min(px(d.index()*16), du(d.index())_ will choose the smaller of 16 pixels times the bar index, or the bar index as a domain unit.
- _max_: works exactly the same way as _min_, except it chooses the larger of the given values.

Anytime we need to define a point in chart space, we will need to provide ScaleBound values. Knowing how to use these functions will save you time and effort when it comes to creating custom tools. Some examples of ScaleBound values in the wild are the _x_ and _y_ fields of any [Point](https://tradovate.github.io/custom-indicators/interfaces/canvas.point.html) type object (so anchors in custom drawing tools, or the _point_ field of a Text type DisplayObject).