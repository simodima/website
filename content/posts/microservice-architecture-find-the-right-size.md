+++
title = 'Microservices 101: balancing the size'
date = 2024-09-20T15:25:11+02:00
draft = false
+++

## Balancing the size of Microservices

One of the most common challenges developers and architects face when designing a microservice architecture is determining the optimal size of each service. How "micro" should a microservice be? In this article, we explore the various factors—acting as driving forces—that influence decisions regarding service size.
<!--more-->

> This post is the result of my experience as a staff developer and my study of how the software industry addresses well-known software architecture problems (specifically, the book [Software Architecture the hard parts](https://www.amazon.com/Software-Architecture-Trade-Off-Distributed-Architectures/dp/1492086894/)).

Designing microservice-based systems involves a constant balancing act, as architectural choices are influenced by evolving requirements, performance considerations, and security concerns. Rather than making static decisions, architects must continuously re-evaluate and adjust their designs to adapt the system to the new requirements.

These factors can be divided into the following categories:

1. Technical factors
    1. Performance & Scalability
    2. Security
2. Domain factors
    1. Code Change Frequency
    2. Multi-purpose services
3. Developer experience factors
    1. Over fragmentation
    2. Optimize for the development process

## Technical factors

**Performance and Scalability Needs**

Diverging performance requirements can also drive the decision to split services. When a single service has components with drastically different performance demands, separating them into distinct microservices allows you to fine-tune each independently.

For instance, if your team is the payment system owner and the credit card payment component receives 1000x more traffic than other payment providers, scaling the entire payment service together might lead to inefficiencies. Instead, of having one single payment service, it can be worth splitting it into separate microservices (one for each provider);

In this way, the new architectural shape enables the team to:
- scale each service independently based on actual traffic patterns, ensuring more efficient use of resources.
- independently deploy each payment provider, reducing the risk of bugs in the entire payment system.

**Security Requirements**

Security is another critical factor when deciding whether to split services or not.
Services that handle sensitive data, such as payment details or personally identifiable information (PII), should be isolated from those with lower security requirements to reduce potential vulnerabilities.

Imagine an e-commerce platform where a service handles user preferences (such as theme settings or notification options) and sensitive payment information. If both functionalities are housed in the same service, the attack surface is broader, and the risk of data leakage increases. By splitting these services, you can apply different security protocols, such as more stringent access controls or encryption, to the payment service while maintaining simpler security for non-sensitive data.

## Domain factors

**Multi-Purpose Services**

One of the key indicators that a service may need to be split is when it serves more than one purpose or encompasses too many distinct responsibilities. In microservice design, it's best to align services with single, well-defined bounded contexts. When a service straddles multiple contexts, it becomes more difficult to maintain, test, and scale.

For example, if a service exposes numerous REST API endpoints that serve different subdomains of your application, this is a signal that it may be too large and complex. Splitting it into more focused, context-driven services can improve maintainability and scalability. By clearly defining each subdomain, you can ensure that each service remains manageable and performs optimally.

**Code Change Frequency**

Another factor that justifies splitting a service is when certain parts of the codebase change far more frequently than others. Services with uneven change frequencies are harder to manage, as constant updates in one section can introduce risks across the entire service.

To get back to the e-commerce example, consider a payment service that handles multiple providers: a PayPal provider, a credit card provider and a "buy now, pay later" provider. The latter is a brand new emerging payment gateway and its API is evolving rapidly.
The code change frequency in the "buy now, pay later" components is very high due to the API updates. In this scenario, separating each provider into distinct microservices could minimize deployment risks, isolate changes to the "buy now, pay later" service, and reduce the testing surface area. By isolating frequently changing components, you can achieve greater stability in the rest of your system.


## Developer experience factor

**Over-Fragmentation**

Splitting services into smaller ones and having more focused components has several advantages, but some forces drive the opposite direction, merging services. The goal is always to avoid the trap of excessive microservice fragmentation, which can introduce complexity and inefficiency.
One of the main risks in a microservice architecture is creating too many services, leading to what is often referred to as "over-fragmentation.".
When services are too fine-grained, the teams start facing lots of serious difficulties:
- communication overhead between services increases because lots of data need to be transferred via API or events
- managing the entire system becomes more complex
- [distributed transactions](https://developers.redhat.com/articles/2021/09/21/distributed-transaction-patterns-microservices-compared) needs to be implemented 

This can result in higher latency, difficulties with deployment, and increased operational costs.

Merging services may become necessary when the overhead of managing multiple microservices outweighs the benefits.
If you fall in the following scenario:
- two services are tightly coupled and need to communicate frequently
- two or more services have business critical requirements in strong writing consistency

Merging them into a single service could reduce the complexity of inter-service communication and improve performance.

**Optimize for development process**

Another reason to merge services is to simplify the development and testing process. If multiple microservices are contributing to the same feature and require coordinated releases, it may be more efficient to combine them into a single unit. This can reduce the coordination effort between teams, simplify CI/CD pipelines, and lead to faster release cycles.

### Conclusion

Finding the right balance in microservice size is about managing competing forces. On one hand, you want to split services to improve scalability, maintainability, and security. On the other hand, you must avoid over-fragmenting the architecture, which can introduce unnecessary complexity and operational challenges.
My advice is to "almost" always start with a modular macro-service (unless you have a clear understanding of the future requirements). Keep the focus on designing well-isolated bounded context in a single service.
Do not design API only for your client (a JS or mobile application), build instead, a consistent API without shortcuts on interface design; The API is the first layer that will allow you to change the underlying implementation easily when your system is mature enough; isolating critical components into dedicated service will be easier.

In conclusion, microservice granularity can be trivial to define when strong drivers are pushing for one or another way (security, performance, data consistency, etc..) but can also be less clear when you start designing a new system, in that case, do not over-engineer the system and let the requirements emerge.