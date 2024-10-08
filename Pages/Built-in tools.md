# Built-in tools
A major part of the indicator can be reused by other indicators. The application includes [tools](https://github.com/tradovate/custom-indicators/tree/master/tools) folder with a set of reusable classes and functions. Indicators can import them via _require_ construction of Javascript.

For example, our EMA can be rewritten as this one:
```
const predef = require("./tools/predef");
const EMA = require("./tools/EMA");

class ema {
   init() {
       this.emaAlgo = EMA(this.props.period);
   }

   map(d) {
       return this.emaAlgo(d.value());
   }
}

module.exports = {
   name: "exampleEma",
   description: "My EMA",
   calculator: ema,
   params: {
       period: predef.paramSpecs.period(10)
   },
   tags: ["My Indicators", predef.tags.MovingAverage],
   schemeStyles: predef.styles.solidLine("red")
};
```
Source codes of the _tools_ folder are available via the Code Explorer module of the application.