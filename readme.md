# Service Principles

- [Service Principles](#service-principles)
  - [Creation](#creation)
    - [Discuss the organization of your overall system first](#discuss-the-organization-of-your-overall-system-first)
    - [Check whether you can add your feature to an existing service](#check-whether-you-can-add-your-feature-to-an-existing-service)
    - [Consider whether your feature is better suited to a library](#consider-whether-your-feature-is-better-suited-to-a-library)
    - [Services are curated by teams, not individuals](#services-are-curated-by-teams-not-individuals)
    - [Services are a long-term commitment](#services-are-a-long-term-commitment)
    - [Factor in the overhead of deploying a distributed system](#factor-in-the-overhead-of-deploying-a-distributed-system)
    - [Prefer larger services](#prefer-larger-services)
    - [Minimize the depth of the service call-graph](#minimize-the-depth-of-the-service-call-graph)
    - [Minimize the number of services owned by your team](#minimize-the-number-of-services-owned-by-your-team)
  - [Interfaces](#interfaces)
    - [Interfaces should be easy to understand](#interfaces-should-be-easy-to-understand)
    - [Interfaces should be robust](#interfaces-should-be-robust)
    - [Changes to interfaces should be backwards compatible](#changes-to-interfaces-should-be-backwards-compatible)
  - [Testing](#testing)
    - [Any changes to your service should be able to be tested automatically](#any-changes-to-your-service-should-be-able-to-be-tested-automatically)
    - [Your interface is the most important thing to test](#your-interface-is-the-most-important-thing-to-test)
  - [Operations](#operations)
    - [You are responsible for running your service](#you-are-responsible-for-running-your-service)
    - [Guide your clients’ expectations](#guide-your-clients’-expectations)
    - [Plan for failure](#plan-for-failure)
  - [Additional Reading](#additional-reading)

## Creation

### Discuss the organization of your overall system first

As you start thinking about designing your service, talk with your teammates and other service experts before writing any code or even design documents. Think about the features you want to add in addition to how they fit in with any existing features, products, and services. Think about the most sensible way to organize the entire feature set. Sometimes adding new features warrants a reorganization of existing components.

We want to avoid "append-only" service architecture where development only happens in the form of new services.

### Check whether you can add your feature to an existing service

Before writing a new service, you should check that your feature doesn’t belong in any existing service. It may overlap with existing functionality, deal with the same information, or be a natural progression of an existing service’s scope. On the other hand, if adding the new functionality to an existing service would be surprising and confusing to users of that service, it probably doesn't belong there.

### Consider whether your feature is better suited to a library

Even though this is a document about services, it's important to point out that some features are better suited to libraries. To help you make the right call for your feature, we describe some of the tradeoffs between libraries and services:

***Upgrade speed*** Libraries are best suited to situations where it's acceptable for users to upgrade over long periods of time (normally weeks to months, sometimes years). Typically, this requires that your feature is relatively small and self-contained, resulting in a low rate of change.  In contrast, if you anticipate lots of ongoing development, or you may need to quickly roll out a bugfix to all users, then your feature may be better suited to a service. This will tend to occur with more complicated features, potentially with external dependencies.

***Performance and reliability*** Libraries live in the same address space as the calling process, whereas services live on the other end of a network cable. All other things being equal, it's going to be faster and more reliable to access your feature as a library. However, if your feature has high resource requirements (e.g. CPU, memory or disk) then implementing it as a service can allow clients to run more efficiently and allow the service hardware to be scaled independently of the client hardware.

***Technology independence*** For the most part, Yelp is a Python shop, with a few Java systems on the backend. If your feature does need to be accessed by both Python and Java clients, then writing it as a service is going to allow you to avoid writing two different library implementations (one for each of Python and Java).

### Services are curated by teams, not individuals

Each service should be curated by a team instead of an individual. This prevents situations where only one developer knows how to fix the service. In practice, this means that every service should be developed by more than one person from the very beginning, and operational responsibilities must be shared.

### Services are a long-term commitment

When you write any code, you are committing your team to providing ongoing maintenance and operational support. Over time you will build up a set of consumers of your service that will depend on it to stay operational. You need on-going monitoring for performance, consistency, and brown-outs. You may also need to respond to failures in downstream systems.

### Factor in the overhead of deploying a distributed system

The initial rollout of a service often takes longer than you might think. Sometimes a lot longer. This could be due to unexpected interactions between other services or difficulties in setting up the right monitoring. Load testing is rarely perfect so plan on going through several stages and iterations before being 100% deployed.

### Prefer larger services 

Aim to place related functionality into a single, larger service instead of multiple, smaller services. Note that your larger service should be logically cohesive i.e. you should still be able to concisely describe its behavior. The rationale for this guideline is as follows:

* In-process calls are more reliable and faster than inter-service calls.
* It is often easier to change just a single service instead of coordinating changes across multiple services. In particular, it is relatively expensive to change service interfaces.
* At the operations level, it tends to be easier to keep a smaller number of more uniform processes correctly running.
* There are some maintenance tasks which must be separately performed on each service e.g. upgrading to new versions of libraries. It is easier to do this if there are fewer services.

### Minimize the depth of the service call-graph

When architecting services, prefer a shallow inter-service call-graph (service ``A`` calls services ``B``, ``C`` and ``D``) to a deep one (``A`` calls ``B`` calls ``C`` calls ``D``). The rationale for this principle is:

* It is often easier to reason about shallow call-graphs; in the shallow case, B, C and D have no external service dependencies, whereas in the deep case both B and C depend on another service. Note that this fits naturally with the [three-tier architecture](http://en.wikipedia.org/wiki/Multitier_architecture#Three-tier_architecture).
* It tends to increase performance; in the shallow case, the service calls can be executed in parallel (as long as they are independent), whereas in the deep case all code runs serially.

### Minimize the number of services owned by your team

Your team is [hopefully] organized to deliver some product or related set of products. The services you develop should be aligned with that focus. If you have too many services, it means either your team is spread too thinly across dissimilar systems, or you're not unifying your services enough.

If your team is already responsible for a large number of critical services (ex: more services than developers), and the new feature does not fit into any of the existing services, you may need to re-distribute responsibility before your team is able to take on the task of building another service. You should also avoid sharing responsibility for services across more than one team.

## Interfaces

The interface to your service may be more than just its public [REST](http://en.wikipedia.org/wiki/Representational_state_transfer) interface. Logs, data dumps and streams that are consumed by other services should be considered. Your interface is the sum of all the parts that are usable by clients.

> For examples of good interface design we try to follow, see the [GitHub API v3](https://developer.github.com/v3/) and the [PayPal REST API](https://developer.paypal.com/docs/api/)

### Interfaces should be easy to understand

When designing an interface, follow best-practices such as:

* Use self-describing names. Be consistent internally and externally.
* Have domain experts review your interface.
* Have one obvious way of performing each operation.
* When porting to a service your existing functions won’t necessarily make the best network endpoints. Remote execution changes the nature of consistency, reliability, and performance. Design your service boundary to be loosely coupled with other systems.
* Separate the interfaces that read from those that update. (See [CQRS](http://martinfowler.com/bliki/CQRS.html) as an example)
* Minimise and simplify the interface as much as possible. This will also aid in testing. 
* Document any areas where confusion may arise.
* Ask yourself: "Could a new-hire understand your interface without having to talk to you?"

### Interfaces should be robust

Design your interface as if it is going to be exposed to the wider Internet. Only expose information [that is strictly required by clients](http://en.wikipedia.org/wiki/Law_of_Demeter). Where possible, avoid providing unsafe or expensive operations.
	 	
Differentiate between read-only methods vs those that change state. Ideally your read-only methods are idempotent and cacheable.

### Changes to interfaces should be backwards compatible

Your interfaces should have some versioning mechanism. When you change a service interface, pre-existing clients will still be invoking the previous version of your interface. You must not break those clients. Plan on continuing to support these older clients, or to be spending additional time coordinating with them about upgrades. It’s acceptable to have several versions of an interface in use at the same time.

## Testing

### Any changes to your service should be able to be tested automatically

Yelp doesn't employ separate QA engineers. Instead, we rely on computers to do our validation. You are responsible for maintaining a good test suite for your service. Your tests should run quickly and reliably in both dev and test environments.

A good test suite is like a pyramid. At the base are your unit tests: many fast, small tests that validate individual implementation details of your code. In the middle are your integration tests, in which you validate how various components of your service interact. At the top is a small but critical set of acceptance tests that validate your service works correctly with dependent services.

### Your interface is the most important thing to test

Your interface is an important part of your contract. The interface is what your clients use and interact with. If you change your interface and break or change how it behaves you're affecting all clients of your service. This can have widespread impact. 

This is why your interface is the most vital component to test. Your interface tests will tell you what your client actually sees, while your remaining tests will inform you on how to ensure your clients see those results. Ensure that all active versions of your interface perform consistently. Write these tests as early as possible, since the desired behavior from a blackbox testing perspective should be driving your interface design.

## Operations

### You are responsible for running your service

Your team must be committed to the on-going maintenance of your service. Your friendly Operations team is the first line of defense when there are overall site problems, but they're mostly doing triage when services are involved. It's your responsibility to monitor the health of your service, have meaningful alerts, and have a plan of action in the event of problems. You are the one who knows best how your service works, so you're best positioned to identify and fix problems when they arise.

Write a runbook for your service. Fill it with common operational cases (such as deployment, monitoring, and troubleshooting). Document known errors.

### Guide your clients’ expectations

You need to be able to clearly communicate to clients the operational characteristics of your service. We suggest that you actively monitor and performance test your service in order to understand these characteristics and that you commit to certain thresholds that the service should maintain.

For example, if I tell my clients that "99% of requests return in under 100ms with 99.99% uptime", it means I should be actively monitoring the performance and availability of my service to ensure that I'm sticking to my guarantee.

It's okay for different services to have different operational guarantees. The uptime of a "cat picture of the day" service that's only used internally by your team for laughs may not even be actively monitored or alerted. A service that's production-facing and part of the critical path to major endpoints should have a much higher commitment to uptime, performance, and any problems that arise.

### Plan for failure

One of the major defining features of a Service Oriented Architecture is a vast increase in the number of possible failure scenarios in your distributed system. Operationally, this means your service must strive to be honest about the collaborating services and datastores it requires to operate, and those whose downtime can be gracefully handled.

Plan from the start of your service to identify external dependencies and document your relationship to them. Place metrics on external links, and put in place both passive and developer-operated protections so you can protect your service against external failures. Finally, make sure you actively monitor these potential weak points, and alert your on-call team appropriately depending on their severity.

In addition, your service will likely be running across multiple machines, racks, and datacenters, each with their own possibility of failure. Understand how your service will behave under each scenario.

Well-designed services consider all and every failure points and make a conscious decision on how to degrade in every single one of these.

## Additional Reading

* [Martin Fowler on Microservices](http://martinfowler.com/articles/microservices.html)
* [Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter)
* [Steve Yegge's Google Platforms Rant](http://steverant.pen.io/)
* [The Fallacies of Distributed Computing](http://en.wikipedia.org/wiki/Fallacies_of_distributed_computing)
* [Designing Poetic APIs](http://pyvideo.org/video/2647/designing-poetic-apis)
