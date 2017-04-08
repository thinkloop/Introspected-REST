# Introspected REST
(Sliding away from Roy Fielding's `REST` model)

There has been a great confusion of what a `REST` API is.
Most people think that `REST` API is just a CRUD over HTTP.
Or a CRUD with some links.
Or a nicely formatted, a sophisticated CRUD.

In this _manifesto_, we will give a specific definition of what `REST` is, according to Roy,
and see the majority of APIs and API specs ([JSONAPI](http://jsonapi.org/format), [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08) etc) fail to follow this model.
Then, we will propose a new model that brings into the table the same things,
yet it's much simpler to implement while at the same time being backwards compatible with any current (sane) API.

## 1. Definitions
First some definitions, that we will use through the text:

* `REST`, `RESTfull`: The model that Roy defined in his [thesis](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) (along with his blog post [REST APIs must be hypertext-driven](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)).
* `RESTly`: APIs that follows all parts `REST` model, except HATEOAS in which they support mostly links (specs like [JSONAPI](http://jsonapi.org/format), [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08) etc)
* `RESTless`: APIs that have a plain JSON API without any links (follows `REST` model other than HATEOAS)
* `Introspected REST`: APIs that follow the definition of the model we provide in this _manifesto_

We will use the term APIs and networked APIs interchangeably.

## 2. Introduction
`REST` defined by Roy was a magnificent piece of work, much ahead of its time
which took us 10+ years to understand what its capabilities are.
However, now, almost 20 years later `REST` model shows its age. It's unflexibile,
difficult to implement, difficult to test, with performance and implementation issues.
But most importantly, any implementation of `REST` model is _very_ complex.

Now, one could say that, most APIs are not build with mind to last for decades and maybe
that's the reason that this model hasn't seen much adoption.

The former is true but even if the latter is also true could it mean that this model is
not suitable for short-term APIs?

We firmly believe that `REST` is much better than any API that does not follow `REST` principles
(like `RESTly` APIs), even for short-term APIs.
Networked services have very peculiar characteristics which, until now, only `REST` has fully addressed them.
Being able to evolve your API without breaking the clients is critical.

Imagine the following scenario: you have built an Online Social Network and an iOS app that talks the API on your backend.
Now imagine that, after a company meeting, your CEO needs you to make tiny yet important change in the signup page: require the user
to fill in her age, a field in the signup form you didn't have before.
Essentially, this means, in API terms, add an extra field in the accepted object and require it from the client to be filled in
by the user before sending it over.

If your API is `RESTly` and not `REST`, this means that you need to fix the code in the iOS side, test it and send a new iOS app to Apple store.
It takes roughly 1 week for Apple to review your app and if your app won't be rejected for some reason, your
tiny change will take action at least a week later after requested.
If your API _was_ `REST` that would mean a change on the server's response denoting which fields are required to submit the form.
You would have the change deployed 10 minutes later.

Roy notes in his thesis:

>  A system intending to be as long-lived as the Web must be prepared for change
>
> --- Roy Fielding
>

Let me rephrase that in terms you will sound familiar to you:

>  If you want to move fast, you should build a change-first API.
>
>
An API that can change the state of the client without needing the latter to change.

Given that, how can we have a simpler model than `REST`, yet have the same functionality of
`REST`?

As we will show, `Introspected REST` is an API architectural style that solves that.
An architectural style is not an implementation and it's not a spec either.
As Roy notes:

>  An architectural style is a coordinated set of architectural constraints that restricts
>  the roles/features of architectural elements and the allowed relationships among those
>  elements within any architecture that conforms to that style.
>
> --- Roy Fielding

`Introspected REST` is based on Roy's initial model but removes the need for runtime HATEOAS.
Instead, the client derives the state using instrospection.

Eventually this brings the same advantages as Roy's model while being it's much simpler,
much more flexible and backwarde compatible with any Restfull API.

But first let's discuss about Networked Services.

## 3. Networked Services and APIs
Nowadays JSON has become so popular that developers almost forget that there is whole bunch of
protocols below it.
Developers also forget that JSON is just a specification in the message level, like XML.
It's not the only one and definitely it's not the best we could use.
Nevertheless it's simple and simplicity is a virtue.

When describing networks we follow the OSI model description, with TCP/IP following 5 of OSI's 7 layers,
but in our case we will make a more API-specific description

When we want to request a resource from a networked hypermedia-based API, we _roughly_
have the following levels:

### 3.1. Application level
In the application level, the client starts content negotiation (or content selection), usually asking
for only one Media Type.

In the HTTP this is achieved by using the `Accept` header while server responds with `Content-Type` header.

`application/json` is a Media Type that denotes that the data format of the requested
representation is in JSON data format.
Specifically the type of this Media Type is `application` while the subtype is `json`.
**JSON itself is not a Media Type but a message format**.

Media Types can be a bit more complex as well: `application/vnd.api+json`, the media type of [JSONAPI](http://jsonapi.org/format) spec, (roughly) means that
* the main type is `application`
* the subtype is `vnd.api` which _roughly_ denotes the Media Type name
* the underlying structure follows JSON semantics

In theory, [JSONAPI](http://jsonapi.org/format) spec spemantics could also be applied using XML as the data format (like in the case of [HAL](https://tools.ietf.org/html/draft-kelly-json-hal-08)),
or even YAML, however in practice we tend to forget that and we treat all Media Types as single and not composite.

However, it should also be noted that the Media Types and the content negotiation in general, are
not restricted to HTTP only.
Although HTTP is one of the most popular network protocols today, the same logics could be applied
in other (mostly text-based) protocols like SIP, CoAP, QUIC etc.

To sum up, the application level semantics are not coupled tight to the semantics of the
message level (like JSON) or the underlying protocol level (like HTTP).

### 3.2. Message level
In the message level we find the format that is used for the actual representation.
Nowadays we have almost mixed the message level with JSON but in practice other
formats could successfully be used: XML, YAML, TOML to name a few.

Usually the message format that is used is described by the Media Type's suffix.

### 3.3. Protocol level
In the protcol level, the requests are usually sent using the HTTP.
After all, nowadays most of the development happens around the Web and
HTTP is the only protocol that browsers officially support.

Nonetheless, there are other protocols as well.
QUIC is a HTTP alternative protocol that is targeted for low latency and uses UDP
underneath.
CoAP is targeted in the IoT and also uses UDP underneath (full TCP/IP stack is quite heavy for constrainted devices).
SIP is also a text-based protocol with the same semantics as HTTP and is used in VoIP.

### 3.4. Network level
Finally (well for the scope of this manifesto, in networks the lowest protocols are the one found in the Physical level
which deal with the wire signals), in the network level, the browser (or any other non-browser client) sends the networked request
in one of the TCP, UDP, etc

The actual protocol depends on the protocol used by the protocol level.


## 4. Roy's `REST` model
Roy's `REST` model is an arhictectural style which is not tight to any spec, protocol or format of the
aforementioned levels.

> a RESTful API is just a website for users with a limited vocabulary (machine to machine interaction)
>
> --- Roy Fielding


When Roy talks about `REST` he mentions 5 crucial properties of a `REST` model:

### 4.1. Access methods have the same semantics for all resources
> induces visible, scalable, available through layered system, cacheable, and shared caches

Failure to provide consistency on access would imply that you don't provide a generic interface but instead
you have resource-specific or even object-specific interfaces.

Actually a common interface is one of the most crucial parts of REST: without a common uniform interface
it would be impossible to derive REST:

> The central feature that distinguishes the REST architectural style from other
> network-based styles is its emphasis on a uniform interface between components.
> By applying the software engineering principle of generality to the component interface,
> the overall system architecture is simplified and the visibility of interactions is improved.
> Implementations are decoupled from the services they provide, which encourages independent
> evolvability. The trade-off, though, is that a uniform interface degrades efficiency,
> since information is transferred in a standardized form rather than one which is specific
> to an application's needs. The REST interface is designed to be efficient for
> large-grain hypermedia data transfer, optimizing for the common case of the Web,
> but resulting in an interface that is not optimal for other forms of architectural interaction.
>
> In order to obtain a uniform interface, multiple architectural constraints are needed to guide the behavior of components.
> REST is defined by four interface constraints: identification of resources; manipulation of resources through
> representations; self-descriptive messages; and, hypermedia as the engine of application state.
>
> --- Roy Fielding
>

### 4.2. All important resources are identifed by one resource identifer mechanism
> induces simple, visible, reusable, stateless communication

Roy explains that very well in his thesis:
> A resource is a conceptual mapping to a set of entities, not the entity that corresponds to the mapping at any particular point in time
>
> More precisely, a resource R is a temporally varying membership function M<sub>R</sub>(t),
> which for time t maps to a set of entities, or values, which are equivalent.
> The values in the set may be resource representations and/or resource identifiers. [...]
> Some resources are static in the sense that, when examined at any time after their creation,
> they always correspond to the same value set.
> Others have a high degree of variance in their value over time.
> The only thing that is required to be static for a resource is the semantics of the mapping,
> since the semantics is what distinguishes one resource from another.
>
> --- Roy Fielding
>


### 4.3. Resources are manipulated through the exchange of representations
> induces simple, visible, reusable, cacheable, and evolvable (information hiding)

The respresentation that you expose from your public API could be totally different from
your implementation internally (usually in your db).
It could also be the same.
Nevertheless the client expects and is expected to manipulate any resource using the representation
you expose.

### 4.4. Representations are exchanged via self-descriptive messages
> induces visible, scalable, available through layered system, cacheable, and shared caches
> induces evolvable via extensible communication

> Interaction is stateless between requests, standard methods and Media Types are used to
> indicate semantics and exchange information, and responses explicitly indicate cacheability.
>
> --- Roy Fielding
>

This would mean that the data of the response should follow the Media Type that the client
requested and unrestands.
Given that the client negotiated for that Media Type, it should be able to parse and understand any part of the response.

If you modify the response that breaks the semantics of the Media Type that was agreed upon, the client
would need an out-of-band information because the response wouldn't be self descriptive.

### 4.5. Hypertext as the engine of application state (HATEOAS)
> induces simple, visible, reusable, and cacheable through data-oriented integration
> induces evolvable (loose coupling) via late binding of application transitions

This is one of the most misunderstood parts of Roy's REST model. The idea here is that,
once the client and server have reached a concensus on the Media Type after the negotiation,
the server should strictly provide all the available options for the client to select.

As Roy notes:

> A REST API should be entered with no prior knowledge beyond the initial URI (bookmark)
> and set of standardized Media Types that are appropriate for the intended audience
> (i.e., expected to be understood by any client that might use the API).
>
> From that point on, all application state transitions must be driven by client
> selection of server-provided choices that are present in the received representations
> or implied by the user’s manipulation of those representations.
>
> The transitions may be determined (or limited by) the client’s knowledge of media
> types and resource communication mechanisms, both of which may be improved
> on-the-fly (e.g., code-on-demand).
>
> --- Roy Fielding
>

However, **one of the requirements for HATEOAS to work is that the Media Type itself _must_ allow
to its vocubulary hypermedia and linking.**
For instance, with `application/json` Media Type this wouldn't work as JSON itself
(`application/json` Media Type is nothing more than a JSON) does not provide any of those mechanisms.

Instead, the server and client must agree on a format that provide such mechanisms.

In practice however, we put `application/json` in our Content-Type header denoting
that the response type follows that Media Type and then inside the response we add
semantics regarding hypermedia. Then we hand off out-of-band information to the client,
like documentation, and demand to check them before identifying parsing and using the hypermedia
semantics of our API.

#### 4.5.1. The Human interaction principle
There are 2 types of human involvement:
* 1-fold: Programming the client only once to parse the Media Type correctly and let the
client work for any API that follows that Media Type even when APIs evolve (given that they adhere in the Media Type)
* multi-fold: Programming the client once to parse the Media Type correctly and then
oftenly, when that the server adds/removes resources or evolves the API in general.
The human involvement during the latter phase could be extensive or limited but in any case
the client cannot cope with the new API versions by itself.

#### 4.5.2. The case of API Versioning
Strictly speaking, an API that follows the `REST` model and is evolvable without the need
of human interaction does not need any versioning in the URL.
Instead, given that the  Media Type describes the API's capabilities, it's the right place to
put the versions there.

## 5. REST Applied in a modern API
REST model is built for machine-to-machine communication, of any type.
However, as this form of communication is getting more and more common,
clients are requesting and having more and more requirements from a server response.
It's not enough to just request and get the resource but you should be able to specify
to the server what transformations you need.

Nowadays we have been using networked APIs so much that now we essentially have to
provide an ORM to the client over the HTTP (or any other protocol).

We provide here a list of features (we call them capabilities) that we think should be built in a modern networked API,
in 2017.

### 5.1. Requirements from a modern REST API
We feel that a modern API should at least provide the following features.

#### 5.1.1. Sparse fields (collection/resource)
The client should be able to ask and get specific attributes (i.e. a subset) of the resource representation.
Also related, we should note that a representation of a resource could have completely different set of
attributes for different clients, usually depending on the client's permissions or user role that it represents.

#### 5.1.2. Associations on demand (collection/resource)
The client should be able to ask related associations to the main initial resource, in the same request.

What deffirintiates an association from an attribute is that the former has
a dedicated identification which can be used in order to be retrieved by itself.

#### 5.1.3. Sorting & pagination (collection only)
The client should be able to sort based on one or more attributes and paginate the collection
based on the page, page size and possibly an offset.

#### 5.1.4. Filtering collections (collection only)
The client should be able to run any sort of collection filtering, as long as it does not pose
any security thread or slows down the API performance.

#### 5.1.5. Aggregation queries (collection only)
The client should be able to run any sort of aggregation queries, as long as it does not pose
any security thread or slows down the API performance.

#### 5.1.6. Data types !
The client should know the data types of the attributes of the requested representation of a resource.
Message formats provide some data types but they are pretty basic.
For instance, JSON defines `String`, `Boolean`, `Number`, `Array`, and `null`.
Anything more than that we need to define it in the documentations.

We feel that these 5 data types that JSON provides are just a joke for modern APIs and that we should
have a much larger list of options to select from.
What is more, we should be able to provide custom types in an easy way, for instance, a field is `String` but
has maximum length of 255 characters, it follows a specific regex etc.

### 5.2. Media Types vs HATOEAS
Now the reader could be wondering: where is the appropriate place to describe those capabilities,
in our API's Media Type or using HATEOAS ?
What goes where?

#### 5.2.1. Defining a new Media Type is not easy and should be avoided
Creating a new Media Type for our API is genrally considered bad practice.
Create a new Media Type only if you are sure that none of the already published
Media Types can fit in your API design.

Also, extending an existing Media Type or adding a complementing Media Type to an
existing one (like `application/vnd.api+json+my_custom_data_types`) wouldn't work.
Not only the existing Media Type specification does not provide any extensibility principles,
but also, the main reason is that **the client _must_ understand the Media Type before hand**.
As a result, if we would like to use some _new_ custom types in our (already deployed) API, we would have to publish
the Media Type before hand and let humans implement code to fully parse API responses that
follow this Media Type or API responses that their media type also include this new media type.


#### 5.2.2. HATOEAS can get pretty heavy
Imagine if you have to describe in a resource, all the available actions along with the available API
capabilities _in that specific resource_.
Your API response would just explode in terms of size while making your API super complex.

#### 5.2.3. Balancing between Media Types and HATOEAS
The idea is that Media Types descibe the generic capabilities while HATOEAS
describe the resource-specific capabilities.

However we should note that **Media Types are not parsed by the client** (there was never such intention anyway)
which means that the client must be programmed by a human before hand in order to support that Media Type.

As a result, the Media Type can't be very restrictive because that would mean it would restrict the API designer's freedom
to design the API the way she wants.

For instance, in pagination, most RESTy APIs use a `page` and a `per_page` parameter in the URL.
If the Media Type describes how to do pagination using, say, a URL template on the resource path (like `/{resource}?page={page}&per_page={per_page}&offset={offset}`)
this would mean that all APIs following this Media Type should have the pagination following that URL template.
The level of restriction becomes more obvious when describing more complex capabilities.

On the other hand, if everyone follows that Media Type then it's easier to program our clients.
Specifically, especially when having a restrictive Media Type, if we create a client that parses responses using that Media Type
then it's easy to "configure" it for another API which also follows that Media Type.

HATEOAS should describe on a per-resource basis if the pagination is supported, what is the maximum `per_page` etc.
Essentially, HATOEAS should provide any details missing from the Media Type for the client to work.


#### 5.2.4. An alternative architecture
We feel that the current Media Type specification and use is dated.
If Software Engineering has learned us something, is that composition can enforce Single Responsibilty Principle if used correctly.
Inspired by that, later, we will suggest a new concept,  Microtypes, small composable modules that combined together can form a Media Type.

As a result, clients should be able to even negotiate parts of the Media Type and not the Media Type as a whole.


## 6. API Specs Today
Now that we defined what REST is, according to Roy, and what capabilities modern APIs should provide,
let's see the specs for REST(y) APIs available as today, April 2017, and what they provide.

### 6.1. Our use case
In our use case we will follow the aforementioned points of the `REST` model.

Our use case is a minature of yet another Social App.
Specifically, in our API domain, we have a `User` resource which has other, associated resources, like `Micropost`, `Like`, etc

For our message format, we will use JSON as it's the most popular but it could be anything like XML, YAML etc.

* `User`
  * `id`, a String, never empty or NULL, primary ID of the resource
  * `email`, a String, never empty or NULL, with maximum length up to 255 characters, email format
  * `name`, a String, with maximum length up to 150 characters
  * `birth_date`, a String, representing a Date according to `iso8601`, in `2017-04-01` format.
  * `created_at`, a String, never empty or NULL, representing a DateTime according to `is8601`, in UTC
  * `microposts_count` an Integer

So given the `REST` model properties we _could_ have the following routes:
* `Users` resource (`/users`):
  * List users (`GET /users`): Gets a collection of `User` resources
  * Create a new user (`/users`): Creates a new `User` with the specified attributes.

* `User` resource (`/users/{id}`):
  * Get a user (`GET /users/{id}`): Gets the attributes of the specified `User`
  * Update a user `PATCH /users/{id}`: Updates a `User` with the specified attributes
  * Delete a user `DELETE /users/{id}`: Updates a `User` with the specified attributes

_`Users` and `User` are 2 distinct resources which are often, mistankingly, missthought as a single, one, resource_

As we mentioned, `User` resource has also some associations (or relations/relationships if you prefer).

In plain JSON the a User resource would look like:
```json
{
  "user": {
    "id":"685",
    "email":"vasilakisfil@gmail.com",
    "name":"Filippos Vasilakis",
    "birth_date": "1988-12-12",
    "created_at": "2014-01-06T20:46:55Z",
    "microposts_count":50
  }
}
```

while a collection of `User` resources, the `Users` resource, would look like:

```json
{
  "users": [{
    "id":"685",
    "email":"vasilakisfil@gmail.com",
    "name":"Filippos Vasilakis",
    "birth_date": "1988-12-12",
    "created_at": "2014-01-06T20:46:55Z",
    "microposts_count":50
  }, {
    "id":"9124",
    "email": "robert.clarsson@gmail.com",
    "name": "Robert Clarsson",
    "birth_date": "1940-11-10",
    "created-at": "2016-10-06T16:01:24Z",
    "microposts-count": 17,
  }]
}
```


Now that we defined the scope of our little API, let's see how this would be implemented
in the specs for REST(y) APIs currently available. We feel that most current APIs
have a lot of similarities with the following specs, namely the structure and the HATEOAS part (regarding
linking), and as a result by comparing those specs with our model would be sufficient.

### 6.2. [JSONAPI](http://jsonapi.org)
JSONAPI was originally created by [Yehuda Katz](http://yehudakatz.com/), as part of Ember's ember-data library.
Since then a lot of people have contributed and has rised as one of the most supported
API specs as of 2017 in terms of tools and libraries.

##### User resource
```json
{
  "data":{
    "id":"1",
    "type":"users",
    "attributes":{
      "email":"vasilakisfil@gmail.com",
      "name":"Filippos Vasilakis",
      "birth_date":"1988-12-12",
      "created-at":"2014-01-06T20:46:55Z",
      "microposts-count":50
    },
    "relationships":{
      "microposts":{
        "links":{
          "related":"/api/v1/microposts?user_id=1"
        }
      },
      "likes":{
        "links":{
          "related":"/api/v1/likes?user_id=1"
        }
      }
    }
  }
}
```

##### Users resource (a collection of User resources)

```json
{
  "data":[
    {
      "id":"1",
      "type":"users",
      "attributes":{
        "email":"vasilakisfil@gmail.com",
        "name":"Filippos Vasilakis",
        "birth_date":"1988-12-12",
        "created-at":"2014-01-06T20:46:55Z",
        "microposts-count":50
      },
      "relationships":{
        "microposts":{
          "links":{
            "related":"/api/v1/microposts?user_id=1"
          }
        },
        "likes":{
          "links":{
            "related":"/api/v1/likes?user_id=1"
          }
        }
      }
    },
    {
      "id":"9124",
      "type":"users",
      "attributes":{
        "email":"robert.clarsson@gmail.com",
        "name":"Robert Clarsson",
        "birth_date":"1940-11-10",
        "created-at":"2016-10-06T16:01:24Z",
        "microposts-count":17
      },
      "relationships":{
        "microposts":{
          "links":{
            "related":"/api/v1/microposts?user_id=9124"
          }
        },
        "likes":{
          "links":{
            "related":"/api/v1/likes?user_id=9124"
          }
        }
      }
    }
  ],
  "links":{
    "self":"/api/v1/users?page=1&per_page=10",
    "next":"/api/v1/users?page=2&per_page=10",
    "last":"/api/v1/users?page=3&per_page=10"
  }
}
```

While the spec makes a great effort describing the structure of the document, we see some
notable issues. Namely:
 * Limited links (no URI templates, treats the client as totally stupid)
 * No actions
 * No info on available attributes
 * No info on data types
 * No attributes description

To sum up, it requires documentation and multi-fold human interaction.

### 6.3. HAL
* [specifications](https://tools.ietf.org/html/draft-kelly-json-hal-08)

```json
{
    "_links": {
        "self": {
            "href": "/api/v1/users/{id}"
        },
        "microposts": {
            "href": "/api/v1/microposts/user_id={id}",
            "templated": true
        },
        "likes": {
            "href": "/api/v1/likes/user_id={id}",
            "templated": true
        }
    },
    "id": "1",
    "name": "Filippos Vasilakis",
    "email": "vasilakisfil@gmail.com",
    "createdAt": "2014-01-06T20:46:55Z",
    "micropostsCount": 50,
}
```

```json
{
   "_links":{
      "self":{
         "href":"/api/v1/users"
      },
      "curries":[
         {
            "name":"ea",
            "href":"http://example.com/docs/rels/{rel}",
            "templated":true
         }
      ]
   },
   "_embedded":{
      "users":[
         {
            "_links":{
              "self":{
                "href":"/api/v1/users/{id}",
                "templated": true
              },
              "microposts":{
                "href":"/api/v1/microposts?user_id={id}",
                "templated": true
              },
              "likes": {
                "href": "/api/v1/likes/user_id={id}",
                "templated": true
              }
            },
            "id": 9123,
            "name": "Filippos Vasilakis",
            "email": "vasilakisfil@gmail.com",
            "createdAt": "2014-01-06T20:46:55Z",
            "micropostsCount": 50
         }, {
            "_links":{
              "self":{
                "href":"/api/v1/users/{id}",
                "templated": true
              },
              "microposts":{
                "href":"/api/v1/microposts?user_id={id}",
                "templated": true
              },
              "likes": {
                "href": "/api/v1/likes/user_id={id}",
                "templated": true
              }
            },
            "id": 9123,
            "name": "Robert Clarsson",
            "email": "robert.clarsson@gmail.com",
            "created-at": "2016-10-06T16:01:24Z",
            "microposts-count": 50,
         }
      ]
   }
}
```

Goods:
 * Links

Problems with this spec:
 * No actions
 * No info on available attributes
 * No info on data types
 * No attributes description, requires documentation (however it does provide a link to documentation)


To sum up, it requires documentation and involvment of human interaction (curries facilitate that).

### 6.4. Siren
```json
{
  "class": [ "user" ],
  "properties": {
    "name": "Filippos Vasilakis",
    "email": "vasilakisfil@gmail.com",
    "createdAt": "2014-01-06T20:46:55Z",
    "micropostsCount": 50,
  },
  "actions": [
    {
      "name": "get-user",
      "title": "Get User",
      "method": "GET",
      "href": "https://example.com/api/v1/users/1",
      "type": "application/json",
    },
    {
      "name": "update-user",
      "title": "Update User",
      "method": "PUT",
      "href": "https://example.com/api/v1/users/1",
      "type": "application/json",
      "fields": [
        { "name": "name", "type": "text" },
      ]
    },
    {
      "name": "delete-user",
      "title": "Get User",
      "method": "GET",
      "href": "https://example.com/api/v1/users/1",
      "type": "application/json",
    }
  ],
  "links":[
    { "rel":["self"], "href":"https://example.com//api/v1/users/1" },
    { "rel":["microposts"], "href":"/api/v1/microposts?user_id=1" }
    { "rel":["likes"], "href":"/api/v1/likes?user_id=1" }
  ]
}
```

```json
{
  "class":["users"],
  "properties":null,
  "entities":[
    {
      "class":["user"],
      "rel":["https://example.com/v1/users/1"],
      "href":"https://example.com/v1/users/1",
      "properties":{
        "name": "Filippos Vasilakis",
        "email": "vasilakisfil@gmail.com",
        "createdAt": "2014-01-06T20:46:55Z",
        "micropostsCount": 50,
      },
      "links":[
        { "rel":["self"], "href":"https://example.com//api/v1/users/1" },
        { "rel":["microposts"], "href":"/api/v1/microposts?user_id=1" }
      ]
    },
    {
      "class":["user"],
      "rel":["https://example.com/v1/users/9124"],
      "href":"https://example.com/v1/users/9124",
      "properties":{
        "email": "robert.clarsson@gmail.com",
        "name": "Robert Clarsson",
        "birth_date": "1940-11-10",
        "created-at": "2016-10-06T16:01:24Z",
        "microposts-count": 17,
      },
      "links":[
        { "rel":["self"], "href":"https://example.com/api/v1/users/9124" },
        { "rel":["microposts"], "href":"https://example.com/api/v1/microposts?user_id=9124" }
        { "rel":["likes"], "href":"https://example.com/api/v1/likes?user_id=9124" }
      ]
    }
  ],
  "actions":[
    {
      "name":"create-user",
      "title":"Create User",
      "method":"POST",
      "href":"https://example.com/v1/users/",
      "type":"application/json",
      "fields": [
        { "name": "name", "type": "text" },
        { "name": "email", "type": "text" },
        { "name": "birth_date", "type": "date" },
      ]
    }
  ],
  "links":[
    {"rel":["self"], "href":"https://example.com.api/v1/users"},
    {"rel":["next"], "href":"https://example.com.api/v1/users?page=2"}
  ]
}
```


### 6.5. JSON-LD
```json
{
  "@context": {
    "id": "@id",
    "foaf": "http://xmlns.com/foaf/0.1/",
    "firstName": "foaf:firstName",
    "lastName": "foaf:lastName",
    "workplaceHomepage": "foaf:workplaceHomepage"
  },
  "@type": "foaf:Person",
  "id": "http://thomashoppe.me",
  "firstName": "Thomas",
  "lastName": "Hoppe",
  "workplaceHomepage": "http://www.n-fuse.de"
}
```

### 6.5. Hydra



## 7. Ideal `REST` API
**How many years these specs could sustain in terms of evolvability ? Are they built with a lifespan of 2-3 years or are they
built with a life span of 50 years?**

### 7.1. UI-based REST API

### 7.1. The case of versioning

### 7.2. General purpose REST API

In an ideal REST API, we should be able to tell the client, at a given time:
* About each resource returned from the API to the client:
  * default attributes and available attributes of the resource, based on the user's permissions
    * default attributes is a subset of the available attributes
  * data types for each attribute in the resource or any embedded association
  * Sorting/pagination, filtering and aggregation queries availability
  * data type of each attribute
  * Ideally some description targeted for humans
  * default embedded associations and available associations to embed
    * recusrively apply the same information for each association available for embedding
* About each resource sent to the API from the client
  * available actions on the resource
  * attributes, per action, the client can modify, based on the user's permissions
  * required attributes of a resource (attributes a resource _must_ before sending it over)
  * data types of the attributes (could be different from the resource found in the response)
  * associations that are required or can be embedded to the initial request
    * recusrively apply the same information for each association available for embedding

We should note that the reason we don't mention anything about the headers that are required, or, the status codes
is because we feel that these belong to the Protocol level and not in the Application level.
Any changes on this level imply that the API breaks the protocol.

However, we are pragmatic and we understand that an API designer could want to _add_ (not change)
a status code or a header in a given request/response and as a result, ideally, this should also be possible to be described.

Now to the reader, it should be obvious that even if we manage to offload some of the aforementioned information
to the Media Type, we would still have a _very_ complex, massive, response from the server for the HATEOAS.

In our experience, such responses are very hard to implement correctly, test, be performant and even debug. After all,
a human will sit down and write the initial code and debugging the code by the eye is important.
Simplicity is crucial.

Another thing: Some clients might not be interested in hypermedia and evolvability at all. Only the data.
However such APIs force the clients to deal with it.

Ideally we would like to give the option to the client to decide the extend of the hypermedia that it
will support and follow, without taking on defaults. Some clients might want to follow 100% the HATEOAS part
of the API (and as a result be evolvable) some other clients might want the 50%, some clients might be interested
only in data.

By outputing a whole bunch of hypermedia-related information to the clients that, after all, might never use
them is a bad practice.

### 7.1 I miss my good old API
In plain JSON the User resource looks is:
```json
{
  "user": {
    "id":"685",
    "email":"vasilakisfil@gmail.com",
    "name":"Filippos Vasilakis",
    "birth_date": "1988-12-12",
    "created_at": "2014-01-06T20:46:55Z",
    "microposts_count":50
  }
}
```

As simple as that.

A JSON-based API built around 2006 would return just data. No hypermedia, no HATOEAS, only data.
Compared with a HATOEAS-ed response it's simple as hell, obvious, easy to debug and understand by a human (and a client).
And I miss that.

Is it possible to build an API that is simple as that, be Hypermedia driven and give the client the option to decide
the level of HATEOAS it will follow?

## Introspected APIs
>  Simple things should be simple and complex things should be possible.
>
> --- Alan Kay
>
>
In the following we will describe the architecture of the Introspected APIs through
a proposed implementation.
The reader though should not confuse the proposed implementation details with the actual
architecture style.


### Introduction

### Separating Hypermedia from the actual data
JSON Hyper Schemas + HTTP OPTIONS on the endpoint

### Automating the documentation generation
documentation generation could have extra stuff, by assigining a param in the url.





## Related Work
### GraphQL

### Linked Data and Semantic Web

### RESTful API Description Languages


## Future Work
It is obvious that after this Manifesto freezes people will start researching more on the introspected-based.

We would like to give some guidelines towards that direction.

### Microtypes: modules composing a Media Type
> Imagine how poor the Web would have been if we had limited HTML to what was
> needed by an FTP client. That's what most JSON APIs are today.
>
> --- Roy Fielding
>

Currently media types serve as a big monolith. Instead we believe that Media Types should be broken in smaller
media types, each describing very carefully a specific functionality of a modern API.

The Content-Type header is limited up to 128 characters so we might need another header for that.
Content-Type could describe the overall Media Type while Foo header could describe sub-media-types used to produce that Media Type.

Clients and Server should still do the regular negotiation flow even for those sub-media-types.

The reasoning is that, in our experience, we have seen that different API specs define the same functionalities in different ways.
Common foobar should allow us to interchange each of them lorem ipsum.

We will need a microtype for describing each of following:

* querying language over url (filters, aggregations, pagination and sorting)
* requesting specific attributes/associations of a resource and its associations
* linking other resources or associations
  * figure out when links should be placed on the introspection and when in Links header defined by RFC5988
* semantic data of a response
* actions supported by the resource
  * required fields, available fields
* resource data types

## Conclusion
We see that people fail to understand the full extend of Roy's initial `REST` model and what is happening is that
some elements of that REST are applied, some other not and eventually we have a model that has the downsides of both worlds.

In this _manifest_ we showed that we need to separate actual data from resource metadata and hypermedia. Documentation
should be generated.

No you don't need GraphQL

We want to embrace even the simplest APIs and allow them to provide the elements of REST that need, yet being easy to impelement
and backwards compatible.
The key thing here is backwards compatibility, because it allows you to incrementally add REST HATOAS incrementally.

In this Manifesto we will try to kill Roy's model.
It gave us great insights but let's be pragmatic: it will never work out.

We need to **be brave enough and move on**: Roy's HATOEAS-based REST model can be declared as deprecated.

Introspected REST is an alternative backwards compatible API. No breaking changes are needed.

##############################################################
If you want to build the next Introspected-REST spec, you can follow the following reasoning.
Note that this reasoning is message-agnostic, meaning that we use here JSON just because we know it better
but your spec could use anything, yaml, xml etc.

* Start with some SANE defaults and the axion: The simpler your API (and the lesser it deviates from defaults), the simpler the introspection-meta-data should be
* Reach a concencus on a Introspection spec using already defined specs like JSON-Schemas.
* Reach a concencus on a querying language over url (filter + aggregation + pagination)
* Reach a concencus on an URL-API for attribute/association inclusion
* Reach a concencus on linking
* Reach a concencus on denoting linked/semantic data
* Reach a concencus on document structure (root element, meta attributes which should appear in the simple response as well etc)
Each of those could be a separate Media Type

So we keep 80% of the REST constraints and while we understand the benefits of other 20% we switch it with an on-demand alternative that makes the final thing
more flexible and powerful while keeping the final data simple.

There are 3 kinds of criticizers of REST model.
1. The ones who understand what REST is and feel that due to its complexity, they prefer loosing some features and deliver something
simpler, yet easier to implement and test and deliver a RESTfull approach
2. The ones who understand what REST brings on the table but given that they control the client as well,
why should they bother with the whole HATOAS thing?
3. The ones who don't understand REST and just want a plain JSON because it's simple enough


Introspected REST model is flexible enough to cover all those user cases.
It's not a model that is black or white: your API is either Introspected-REST-compliant or it isn't, like REST.



It should be noted this model is not something we conceived in a lab. Some [people]()
have already tried to implement something similar, probably without really knowing
what they were doing.

You see, the shadow of Roy Fielding is above any API developer:
we are afraid to deprecate Roy's REST model and as a result what we are doing is that
we take some elements of Roy's model, apply them, and name our API or spec as RESTSful.
Eventually however, the final result is even worse. It doesn't have Roy's key elements for
a Markov-chain-like client (we still have offline contracts) yet we have added complexity
to our API for little result.


Probably Roy won't like that. He will either:
* declare that Introspected REST is a stupid manifesto that has nothing to do with his REST or
* he will declare the Introspected REST is just yet another REST as he defined it and we never
read his dissertation to actually see that we are defining yet another REST style.

In either case, given that very few has really implemented a Roy-compliant REST API means
that Roy himself failed to explain his model correctly.



Are we sliding a lot from Roy's initial model? No, we modernize it a little bit.


##### Roadmap to json-specific defined Introspected REST specs

#### Why this document
This document describes an architectural style.
It does not describe a specific spec and that's the reason it does not follow the IETF draft standards.

It's a manigesto if you wish.

_Anyone can contribute in this manifesto. Just open a pull request._


The way the introspection is made is up to the API designer, or better, up to the spec. Here we use HTTP OPTIONS as we feel that it's an approriate way.





Roy has done great work on initial HTTP spec and REST definition.
Unfortunately, very few people have truly understood the unique characteristics of networked
APIs which inspired Roy to define the REST.

The idioms of Networked Services are very peculiar.
When we have a client talking over the wire to a server,
neither the client developer nor the server developer has access to the other machine.

This means that if the client needs a specific resource, it must not have an offline contract
on how to retrieve this resource because that would mean
* changes on the server's side is difficult.
* automated API clients are not possible without human interaction.

Instead, the server would help (drive if you will) the client exactly where is needed to.

That's the main reason why Roy is against the version on the URL: because it means that
you take as de-facto that there will be breaking changes at some point.
Instead, a truly REST API should be able to apply changes on the resources without breaking any client
because the client.


* The simpler the API, the simpler the API description.



Good question.
Let's let Roy answer it.

We should describe them somehow though in our API without relying in offline contracts (like documentation)


#### 5.3. An alternative architectural style maybe?
Most of those Media Types specifications would not be needed if the APIs were built
with introspection in mind.

Imagine that we have a Media Type that allows us to describe new media types, called `generic_media_type`.
Then the clients would only need to understand and parse this `generic_media_type` and derive the other
Media Types.
Of course, this scenario is more difficult than it sounds and the goal of this _manifesto_ is not
to provide a generic Media Type.
Nevertheless, API introspection, as we will see, can provide us with information on API's
capabilities along with hypermedia in a much flexible and cleaner way, _without having data and hypermedia (representation metadata) tangled together in 
the representation_.



JSON-LD + Collection+JSON ? https://sookocheff.com/post/api/on-choosing-a-hypermedia-format/


WRITE ABOUT VERSIONING IN THE URL


RFC5988: A means of indicating the relationships between resources on the Web,
   as well as indicating the type of those relationships, has been
   available for some time in HTML [W3C.REC-html401-19991224], and more
   recently in Atom [RFC4287].  These mechanisms, although conceptually
   similar, are separately specified.  However, links between resources
   need not be format specific; it can be useful to have typed links
   that are independent of their serialisation, especially when a
   resource has representations in multiple formats.
Oh wait..we just figured out that having the links in there might not be the ideal.


Vale kai auta tou Roy pou exei sta presentations tou


>Versioning an interface is just a polite way to kill deployed applications
>
> Roy Fielding --- 2013 on interview foobar
>


Are you sure you want an architectural style and not an architecture ? (probably yes)
