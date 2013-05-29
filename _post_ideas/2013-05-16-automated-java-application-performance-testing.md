---
layout: post
title: "Automated Java Application Performance Testing"
tags : [java, load test, profiling ]
tweet: Take a look at this post on java performance test
hashtags: #java #performancetest
bitmark:
---
{% include JB/setup %}

Introduction
============
I am currently writing a library called [Gachette](http://github.com/emeka/gachette) that automatically caches
method invocation results and dependencies.  For example, if a method <code>getResult()</code> on object <code>A</code>
returns 100, the value 100 is cached and will be returned each time the same method is called without having to recalculate
the value.  In addition Gachette will keep track of the dependencies that where used to calculate the result and if any
of them changes, the cached result will be invalidated and the result recalculated on the next call to <code>getResult()</code>.

The initial implementation is naive and not optimized for performance.  Before starting any optimisation, I need
first to measure the performance (CPU, Memory).  Without measure, it would be foolish to "optimize" anything.

This small post contains the information I have gathered during my quest for proper java performance measurement.

Multiverse
==========

[Multiverse]()



References
==========

[1]: http://www.infoq.com/presentations/performance-testing-java "Performance Testing Java Applications"
[2]: http://metrics.codahale.com/ "Metrics"
[3]: http://www.jinspired.com/ "JInspired"
[4]: http://www.yourkit.com/ "YourKit"
[5]: https://visualvm.dev.java.net/ "Java VisualVM"


[6]: http://cobertura.sourceforge.net/ "Cobertura"
[7]: http://www.sonarsource.org/ "Sonar"

