class: center, middle

# Data integration patterns 
## Cooperation is hard

[live presentation](https://alonisser.github.io/integration-patterns-messaging) <br/>
[twitter](alonisser@twitter.com), [medium](https://medium.com/@alonisser/)

---

# The dream of One Integrated system

Once men dreamt of one integrated system, answering all possible needs.

--

Of course we know how that ends

--

"One Ring to rule them all, One Ring to find them,
 One Ring to bring them all and in the darkness bind them"

.img-half-container[![the one ring](./one-ring.png)]

---

# Integration

* Applications rarely live in isolation

* business functionality is spread between cutting edge microservices, ageing monoliths  and the Ancient Dinosaurs in the basement.

--

* Various third parties services provide different business functions like accounting, HR, sales management,affiliation, analytics,
 billing, tracking, customer relations, etc (hooray for SAAS).

Constant build/buy decisions make this more complicated by the day 

--

Users/Stakeholders of course want to access and change the data without ever knowing any of this complexity. 

---

# Integration challenges

* Networks are unreliable (And slow)

--

* Consistency is hard ([CAP theorem](https://dzone.com/articles/understanding-the-cap-theorem))

--

* Data is constantly changing and sometimes even deleted, Schema changes

--

* Applications are not trustworthy (Don't get me started about devs).
 You can never know if the other side would be available **when** you need it.

--

* No two applications are the same, different technology, different business domain.

--

* Change is inevitable: Technology changes, Business requirements change, Environments change, User expectations change

---

# Integration challenges

* Integrators have limited organizational authority over various enterprise apps. (We won't adapt our app to your needs) Might also be a third party we buy something from.

* Even with authority, Integrators have limited resources to change existing applications (Might not be even possible with "over the shelf" software products)

---
# Integration patters - Strategic design

* The big problem of integration is working with different mental models. DDD suggests several strategic patterns for this kind of cooperation:
Bounded contexts, shared kernel, conformist, producer/consumer teams and more

--

* This is out of scope for this talk.. we'll be focusing on the technical aspect of integrating data.

---
# Data Integration question: Stateless vs Statefull

A main question we need to answer when building an integration - Do we need to trigger a "one time" action based on the input. Or do we need to use existing state.

* Stateless; for example an ML model categorizing an input.

* Statefull: A service answering a question based on the last 1000 items (Anomaly detection)

---

# Integration meta pattens: replication vs single datasource

Should we replicate or should we query the source of truth.

1. Replication: Similar to "denormalize" data in a sql scheme. the same data exists in different places,
 together with some data which is unique for each place

2. Single datasource: where everyone is querying from: Similar to "Joining" in a sql scheme. we use foreign keys to get the data stored in another table.    

*Thanks Erik Ashepa from fiverr for this analogy*

---
class: center, middle

# Integration patterns

### Common approaches to integrating different applications

--

 Applications: Might be microservices, third party SAAS providers, in house code applications (notebooks)

---

# Integration pattern: File transfer

The pattern: One application writes a file periodically (or by request) to a place where another application can consume

--

 Pros:

* Suitable perhaps when moving lots of "tabular" data

* Can be Easier to implement 

--


 Cons:

* Locking

* "Heavy updates" => longer "out of sync" periods.

* knowing when something has been updated and the files needs to be consumers.

* When multiple apps needs the data, who handles fetching, syncing state, etc

* Ensuring no missed data (If a consumer was down)

---
# Integration pattern: File transfer

A better variation on this theme - is file transfer on push 

* Example: crm sends daily email with issues to integration partner.

---

# Integration pattern: Shared Db
The pattern: Two apps share a db as way to share data, perhaps one only reads and one reads and writes ("the owner") 

--

 Pros:
* Fast and consistent updates

--


 Cons:
* Deep coupling between different apps with different bounded contexts and thus different data models.

Example: Accounting concept of a client would have lot's of tax related fields and a formal accounting entity. 
But App concept of a client might be a geographical area. Or one service has a categorization of items by 80 categories.
 And another needs only to categorize to false/true according to specific criteria.
---

# Integration pattern: Shared Db
The pattern: Two apps share a db as way to share data, perhaps one only reads and one reads and writes ("the owner") 


Cons continued:
* Who "owns" the scheme? what happens when one app changes the scheme and breaks the other. Or change persistence type?
 Sometimes you don't control the schema. But a prepackaged third party (Integrating straight to a db of a CRM provider).

* Does not scale way with geographically remote apps (Would probably need to setup expensive and fragile replication and lose lots of the benefits) 

--

Note! This isn't the same pattern as applying event stream of db changes from the db. - which is more similar to messaging

---

# Integration pattern: Invoking a remote blocking Api (Remote procedure invocation)
The pattern: Invoking a remote api directly in synchronous way 

--

 Pros:
* Easier to implement

* Might be easier to reason about ordered behavior (instead of async sagas)

--


 Cons:
* Remote procedures are disguised as local procedures and hiding the big differences between the cases
 (See the fail of lots of desktop accounting apps migrated to "cloud server")

* Demand synchronous existence of both parties. Both need to be up and working during the period of the transaction

* Coupled - One app needs to know a lot about the other collaborators (or have an orchestrator knowing all that) 

---

# Integration pattern: Invoking a remote blocking Api (Remote procedure invocation)
The pattern: Invoking a remote api directly in synchronous way 

Cons continued:

* Harder to implement "transactional" behaviors on error scenarios

What happens when in a transactional path (think user creation) a http request has a transient failure.
 Especially when the failing request is "after" some steps (like save user to db)
 
---

# Integration pattern: remote blocking api wrapped by a middleware (Service mesh!)

A variant of the blocking pattern, but some of the http api problem handling is moved to the "network" level
 (Handling retries and transient errors for example).

Additional pros:
* More robust, less transient errors (think the occasional 502 from a loaded microservice)

---
# Messaging

First, A Question: who prefers a phone call over email/Whatsapp/Telegram?

--

Phonecall vs Email demonstrate the core difference between http api and a messaging api: sync vs async conversions.
 Transient vs Persistent conversations 
 
 *(Yes I'm aware that somewhere they also record our calls, but it's not available for me)*  

---

# Messaging

* (quite) reliable, (Optionally) persistent, (Usually) ordered, directional channels, decoupled from applications. Delivering messages with data and metadata.

* Messaging capabilities are typically provided by a s separate software system.
 
* A messaging system "moves" messages from a sender to a receiver over the unreliable network 

* Common Message Systems implementations: RabbitMQ, AWS SQS, Google pub/sub, Azure EventHub

---

# Integration pattern: Messaging

Pros:
* Async sending a message does not require both systems to be up and ready at the same time.
 Also variable timing for producing/consuming

* Throttling, Scaling

* Reliability, Order.

* When using common messaging systems we abstract technical communication code out of our app, and gain polyglot language support. 

* Furthermore, thinking about the communication in an asynchronous manner forces developers to recognize that working with a remote application is slower,
 which encourages design of components with high cohesion (lots of work locally) and low adhesion (selective work remotely).
from *Enterprise integration patterns*

---

# Integration pattern: Messaging

Cons:
* Complex (At least perceived as), harder to reason about. 

* Sequence handling

* Scaling while maintaing all the benefits is harder (Especially scaling while maintaing order)

* Handling Synchronous scenarios

* Vendor lock in (Less so today)
---

# Problematic assumptions

* Data would exist when we ask it:

>>We might get an event that  references a client that doesn't exist any more 


---

# Problematic assumptions

* Data won't change after being loaded in the service

>>Asking another service data we need when we start the service, or need the data isn't enough, We might request data from a service on a client, when we first encounter that.
But after that some client data might change, and if the microservice isn't being updated on change, it goes out of sync.

---

# Integration Dangers - The distributed monolith

* The [distributed monolith](https://sebiwi.github.io/comics/distributed-monolith/).



---

# Integration Dangers - The distributed monolith
How to detect:

* Overly chatty

* Needs to sync deploys

* Scales together

* Breaks together

---
# Integration Antidote

* Bounded context

**"total unification of the domain model for a large system will not be feasible or cost-effective"**

>>Bounded Contexts have both unrelated concepts (such as a support ticket only existing in a customer support context)
 but also share concepts (such as products and customers). Different contexts may have completely different models of common concepts with mechanisms
> to map between these polysemic concepts for integration. Several DDD patterns explore alternative relationships between contexts.[Martin fowler](https://martinfowler.com/bliki/BoundedContext.html)

 *DDD=>Domain driven design by Erik evans*
---
# Integration Antidote
* Well Defined apis

* Being strict about having a single source of truth (A bounded context Smell detector)

---

# Read some more

* [Enterprise Integration patterns](http://www.enterpriseintegrationpatterns.com/) A site dedicated to the book, with more discussions, modern code examples

---

class: center, middle

#Open source rocks!

---

class: center, middle

#Thanks for listening!

---
