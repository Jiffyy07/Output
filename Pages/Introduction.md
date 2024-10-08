
# Introduction
Please check the user guide on how to access custom indicators in Tradovate Trader [here](https://tradovate.zendesk.com/hc/en-us/articles/115011665727-How-do-I-use-custom-indicators-in-Tradovate)

[Here](https://community.tradovate.com/c/custom-indicators/6) you can find out some requests and discussions.

**Javascript**

Tradovate Trader is a cross-platform application: we run it as a standalone application on Windows, Mac OS, Android, iOS, and via regular modern web browsers. To achieve it, our decision was to leverage a cross-platform programming language and SDK that would be available on all these platforms with no (ok, almost no) changes. Javascript is the perfect fit for this idea.

When the app is based on Javascript, charting is supposed to be Javascript-based too. As well as built-in technical indicators that a must-have feature of trading application. The next step is just to expose the internal API to the public to make custom indicators possible.

In spite of old popular beliefs, modern Javascript is not scary unreliable some-glue-for-html slow stuff anymore like it was just several years ago. New standards like ES6 made Javascript friendlier for developers with an object-oriented background. If you are familiar with C#, C++, Java, or Scala, you can find out a lot of surprising similarities.

The range of open-source packages can satisfy (up to some degree, of course) any data science needs: you can find out as digital signal processing libraries as well as machine learning ones. The most advanced of them use your GPU even on a mobile phone behind the scene to speed up calculations.

There is a lot of educational material in Internet - starting from [W3 schools](https://www.w3schools.com/js/default.asp) up to [CodeAcademy](https://www.codecademy.com/catalog) and [Udacity](https://www.udacity.com/course/es6-javascript-improved--ud356). Any community library will definitely have a couple of paper books too.

Because there are so many high-quality materials about the language, we will not try to teach you how to program. Instead, we will focus on how to plug your algorithms to Tradovate Trader.