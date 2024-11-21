---
title: 'Microservices 101: API Maturity level'
date: 2024-11-16T15:25:11+02:00
draft: false
tags:
    - software architecture
    - microservices
    - API
    - swagger
    - OpenAPI

---

## Communication between Microservices

In a microservices architecture, services need to communicate with each other frequently. This communication can be broadly categorized into two types: *East-West* and *North-South*.
**East-West** traffic is the internal communication between microservices. North-South traffic is the communication between the services and external systems or clients.
An effective communication is vital for a microservices app's lifecycle.

### East-West traffic

As said, East-West communication involves interactions within the microservices ecosystem. This internal communication often uses RESTful APIs, so if your team is embracing such architecture it's critical to implement consistent, available, and resilient APIs. They ensure that microservices can communicate and be reliable as the system evolves.

### The Domain Evolution

Implementing API it's not trivial. You must distil the business requirements and constraints into a bunch of URL and request/response objects.
Every set of these entities represents a part of the domain model in the API layer.

As the world evolves, the domain model evolves too. We should adopt a process to easily support changing requirements. This way, our team won't be slowed down by the new API and its maintenance. This often means evolving API schemas and changing their behaviour, but also means deprecating old versions and creating new ones. All this must ensure compatibility and minimal disruption to service consumers.

## API Maturity levels

The API maturity levels is the result of tools, processes, and best practices that a team can agree on and apply on their API development. The processes can be more or less strict. It depends on the project stage and the maturity of the engineering culture.
This is a four-level strength management process that ranges from 0, to an highly structured and robust API governance.
As the system grows more sophisticated, so will the team. They will need to advance through the maturity levels to operate with ease.

Let's dive to the levels to see how we can incrementally improve team API workflows.

### Level Zero: Cowboy 🤠

At this stage, you have very primordial API management practices. Most communication between teams is verbal or informal, like email threads or ad-hoc meetings. Inconsistent API contracts cause confusion and slow communication and development.

There's no agreement on data structures, endpoints, or documentation; resulting on fragile and prone to breaking APIs.

*Example with Alice and Bob*

Alice and Bob are working on both sides of the API. Alice is consuming it from a browser JS app, while Bob is exposing it from a backend service.

The two teammates agreed on the API behaviour and models as follows.

```
PATCH /users/me/settings
{
	"email": {
		"marketing": "ON",
		"subscription": "ON",
		"mentions": "OFF"
	}
}

{"status": "ok"}
``` 

Once implemented the logic (both in the server and the client) all the knowledge of data semantics, request/response validation, and edge cases is lost. The only place where the knowledge is, is the code.
This isn't a problem per se, but, with only the code, recreating a clear picture of the service API layer can be time-consuming and error-prone.

As another developer joins the team (or, if another client needs to use the same API) then, all the specification must be grasped again from the code.

### Level One: From chaos leaning to documentation 📜

The team, now aware of their needs, creates a shared source of truth for the API semantics.
They start putting down in documentation all the API knowledge distilled from:

- the code
- the verbal and written past communications
- test cases
- code comments and examples
- The team's implicit knowledge (like "If you call this endpoint more than once, all subsequent calls will fail")

It's a great step forward for the project; the team have a place where they can get the API documentation at any time. It will be easy to onboard new team members and make the consumers aware of the API offered by the team.
But, this is not a free lunch. The docs must be aligned with the code. For each new change, the devs have to find and update the relevant API docs.

This can be error-prone, easy to forget and in general it's an additional development cost.

### Level Two: The raise of contracts 🤝

At this stage, the team starts using strict API contracts. They will use standards [IDL](https://en.wikipedia.org/wiki/Interface_description_language) like [Open API specifications](https://www.openapis.org/what-is-openapi). The API spec must be reviewed and approved via well defined processes, like Pull Requests.

Implementing a strict API contract is not free of overhead, especially for brownfield projects.
Adopting a formal API definition on legacy projects requires some work. The team must write all the specs for the currently implemented API, that can be a very long work if they have a large API surface.
To mitigate that and start adopting best practices, They can also choose to start writing specs for new implementations and then fill in the old API definition over time.

On the other hand, adopting a contract-first approach to a new project can have some overhead too, but it has also some nice side effects and the benefits far outweigh the cost.
Having an API contract:

- It unblocks API consumers on developing their logic. You can define the contract first and then iterate on both sides: the server and the consumer.
- It sets guardrails to design the interface and API semantics before coding. It guides developers to use a top-down design mindset. Focusing on the API first it helps on implementing things with all the requirements in mind.
- It helps on setting best practices and maintaining a consistent API layers; for example by defining specs linting rules (it has benefits especially in large organizations)

### Level Three: The glory of API management 🏆

The team want to massively automate their API lifecycle. So, on top of the previous best practices (like a well-defined API contract, a review process, and linted API specs), they want to deeply integrate the tools into the software development lifecycle through automation.

Many open-source tools, like [openapi-generator tech](https://openapi-generator.tech), can be easily integrated into CI pipelines like Jenkins or GitHub Actions.

Relying on tooling and automation opens to myriad of possibilities:

**Server can be automatically aligned with the API specs:** The team leverage on automated tool to generate server stubs (requests & responses) and routing definitions.
This can reduce bugs caused by misalignment between the API definition and the actual server implementation by an order of magnitude.

**Client code can be produced by the tools:** Automated tools can generate client code from the API specs. This reduces manual work for consumers. It allows the provider to offer an out-of-the-box SDK.

**Human readable documentation can be produced by tools:** Automated tools can be easily integrated to the CI to produce a human readable documentation. There are plenty of them (like [docusaurus openapi-docs plugin](https://github.com/PaloAltoNetworks/docusaurus-openapi-docs) or even [openapi-generator tech](https://openapi-generator.tech))

---

My two cents are that every API maturity level has its benefits and challenges. Organizations must find a balance that fits their size, engineering culture, and goals in their microservices journey.
