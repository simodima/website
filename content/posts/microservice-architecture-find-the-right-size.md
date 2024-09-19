+++
title = 'How to determine the right microservice size'
date = 2021-11-12T15:25:11+02:00
draft = true
+++

## Balancing the size of Microservices

One of the most common challenges developers and architects face when designing a microservice architecture is determining the optimal size of each service. How "micro" should a microservice really be? In this article, we explore the various factors—acting as driving forces—that influence decisions regarding service size.
<!--more-->

> This post is the result of my experience as a staff developer and my study of how the software industry addresses well-known software architecture problems (specifically, the book [Software Architecture the hard parts](https://www.amazon.com/Software-Architecture-Trade-Off-Distributed-Architectures/dp/1492086894/)).


Designing microservice-based systems involves a constant balancing act, as architectural choices are influenced by evolving requirements, performance considerations, and security concerns. Rather than making static decisions, architects must continuously re-evaluate and adjust their designs to adapt to these factors.

These factors can be divided into the following categories

1. Technical factos
	1. Performance & Scalability
	2. Security (https://www.fedramp.gov/, [SOC2](https://www.akamai.com/glossary/what-is-soc2) )
2. Domain factors
	1. Code Change Frequency
	2. Multi-purpose services
3. Organizational
	1. Over fragmentation
	2. Optimize for development process

## Technical factors

**Performance and Scalability Needs**

Diverging performance requirements can also drive the decision to split services. When a single service has components with drastically different performance demands, separating them into distinct microservices allows you to fine-tune each one independently.

For instance, if a credit card payment component receives 1000x more traffic than other payment providers, scaling the entire payment service together might lead to inefficiencies. Instead, splitting the payment service into separate microservices—one for each provider—enables you to scale each independently based on actual traffic patterns, ensuring more efficient use of resources.

**Security Requirements**

Security is another critical factor when deciding whether to split services. Services that handle sensitive data, such as payment details or personally identifiable information (PII), should be isolated from those with lower security requirements to reduce potential vulnerabilities.

Imagine an e-commerce platform where a customer service handles both user preferences (such as theme settings or notification options) and sensitive payment information. If both functionalities are housed in the same service, the attack surface is broader, and the risk of data leakage increases. By splitting these services, you can apply different security protocols, such as more stringent access controls or encryption, to the payment service while maintaining simpler security for non-sensitive data.

## Domain factors

**Multi-Purpose Services**

One of the key indicators that a service may need to be split is when it serves more than one purpose or encompasses too many distinct responsibilities. In microservice design, it's best to align services with single, well-defined bounded contexts. When a service straddles multiple contexts, it becomes more difficult to maintain, test, and scale independently.

For example, if a service exposes numerous REST API endpoints that serve different subdomains of your application, this is a strong signal that it may be too large and complex. Splitting it into more focused, context-driven services can improve maintainability and scalability. By clearly defining each subdomain, you can ensure that each service remains manageable and performs optimally.

**Code Change Frequency**

Another factor that justifies splitting a service is when certain parts of the codebase change far more frequently than others. Services with uneven change frequencies are harder to manage, as constant updates in one section can introduce risks across the entire service.

Consider a payment service that handles multiple providers—PayPal, credit card, and a "buy now, pay later" option. The latter might be a new payment method, and its APIs are evolving rapidly. In this scenario, separating each provider into distinct microservices could minimize deployment risks, isolate changes to the "buy now, pay later" service, and reduce the testing surface area. By isolating frequently changing components, you can achieve greater stability in the rest of your system.


### Organizational factors

**Over-Fragmentation**

Splitting services into smaller ones to have more focused components has its advantages, there are also forces that drive the opposite direction, merging services. The goal is always to avoid the trap of excessive microservice fragmentation, which can introduce complexity and inefficiency.
One of the main risks in a microservice architecture is creating too many services, leading to what is often referred to as "over-fragmentation." When services are too fine grained, communication overhead between services increases, and managing the entire system becomes more complex. This can result in higher latency, difficulties with deployment, and increased operational costs.

Merging services may become necessary when you find that the overhead of managing multiple microservices outweighs the benefits. For example, if two services are tightly coupled and need to communicate frequently, merging them into a single service could reduce the complexity of inter-service communication and improve performance.

**Optimize for development process**

Another reason to merge services is to simplify the development and testing process. If multiple microservices are contributing to the same feature and require coordinated releases, it may be more efficient to combine them into a single unit. This can reduce the coordination effort between teams, simplify CI/CD pipelines, and lead to faster release cycles.

### Conclusion

Finding the right balance in microservice size is about managing competing forces. On one hand, you want to split services to improve scalability, maintainability, and security. On the other hand, you must avoid over-fragmenting the architecture, which can introduce unnecessary complexity and operational challenges.

Ultimately, the decision on microservice size should be guided by your application’s specific needs, and as these needs evolve, so too should your architecture. Constantly reevaluating the size and boundaries of your services is key to maintaining an efficient, secure, and scalable microservice ecosystem.