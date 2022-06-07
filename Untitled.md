My key takeaways from the entire conference: Learn more about Automerge, Datalog, Lenses, and Timely/Differential Dataflow. Lots of talks were based on these core concepts/patterns.

https://marketplace.visualstudio.com/items?itemName=humao.rest-client

 [https://www.benkuhn.net/progessays/](https://www.benkuhn.net/progessays/)

The Pragmatic Programmer   
Designing Data-Intensive Applications  
The Architecture of Open Source Applications  
Design Patterns: Elements of Reusable Object-Oriented Software (Classic but maybe too dated now)  
Domain Driven Design  
Site Reliability Engineering  
Refactoring  
Domain Specific Languages (Martin Fowler)  
Enterprise Integration Patterns  
Patterns of Enterprise Application Architecture

Since the data retention policy keeps gobbling up our historical discussion, I'm going to post some links here:

-   A Primer on Functional Architecture -> [https://increment.com/software-architecture/primer-on-functional-architecture/](https://increment.com/software-architecture/primer-on-functional-architecture/)
-   Functional Core, Imperative Shell -> [https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell)
-   Boundaries -> [https://www.destroyallsoftware.com/talks/boundaries](https://www.destroyallsoftware.com/talks/boundaries)
-   Parse, Don't Validate -> [https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
-   Railway Oriented Programming -> [https://vimeo.com/113707214](https://vimeo.com/113707214)
-   Integrated Tests are a Scam -> [https://blog.thecodewhisperer.com/permalink/integrated-tests-are-a-scam](https://blog.thecodewhisperer.com/permalink/integrated-tests-are-a-scam)
Concepts like "railway-oriented programming", "functional-core, imperative-shell", "functional architectures", "onion-architecture", "hexagonal architecture", "ports-and-adapters architectures", "value boundaries", "clean architecture" (I'm assuming) etcetera, etcetera, etcetera, are all ways at looking at system design from a very similar perspective - call it "functionally oriented". Specifically, how do we not muddy the domain code with infrastructure concerns (eg. apis, logging, storage, messaging, caching, etc) so that it is super testable and easy to reason on - AND how do we keep the infrastructure concerns as THIN as possible, so as much of the code as possible fits into this very clean, testable design approach.I think this is probably what "frameworkless" advocates are advocating for more so than just "not liking a framework" - it's that many of the popular frameworks often make it harder to apply this style of system design.  Technically, you can use a framework with this style - but this is not in the typical way frameworks are used - there is design-style inertia in frameworks that pushes you away from this "functional" style of design - and that inertia that is very hard to overcome. (edited)

[https://www.infoq.com/presentations/ddd-eric-evans/](https://www.infoq.com/presentations/ddd-eric-evans/)

[https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/](https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/)[https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/)[https://www.amazon.com/exec/obidos/ASIN/0321125215/domainlanguag-20](https://www.amazon.com/exec/obidos/ASIN/0321125215/domainlanguag-20) (ed

https://www.eventstorming.com