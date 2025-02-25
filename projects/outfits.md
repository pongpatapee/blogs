# Outfits (Fitcheck)

Outfits (fitcheck) is a social media project idea created by a good friend -
Kenneth Wong, which I helped work on.

The project was inspired by social medias like BeReal and Pinterest. The core
idea was for people and communities to post their outfits of the day, get outfit
inspirations, and have an easy way to get the outfit's information.

From a technical perspective, the Outfits platform would tag individual items
(clothing, accessories, shoes, etc.), which enable users to have their _closet_
where they can mix & match their outfits. This would allow for _recommendation
systems_ and the ability to easily find information on individual items, which
had business implications if the product ever took off.

## Design Considerations

A lot of work for the design was put in by Kenneth, so all credit to him. I was
pretty involved in the backend & API setup, so here are the design decisions
from my understanding and perspective.

### Backend & API Design

#### API Versioning

`/api/v1/endpoint`

- Backwards compatibility
- Evolving flexibility

API versioning allows for **backwards compatibility** and **flexibility** as you
evolve your API. We could continue to work and iterate on our API design while
serving a previous version to our users.

#### Pagination

Pagination is a standard practice in implementing APIs, especially for larger
datasets. It improves performance as retrieving and processing smaller chunks of
data reduces response time, network bandwidth, and minimizes the load on the
servers/DB, improving the overall efficiency.

#### Security

Security is extremely important in API design, we don't want to expose users
data or allow unauthorized requests. Here are some security measures implemented
in our app.

- **JWT**: Every request is made in the context of the current user.

- **Salted hashed passwords**: Never store user passwords in plain text.

#### Layered Architecture

To separate concerns in our application, we implemented 3 layers.

- **API Layer**: Handles HTTP request and response logic.
- **Service Layer**: Handles business logic and converts DB models (SQLAlchemy) -> response
  models (Pydantic).
- **Data Access (DAO) Layer**: Handles database interactions.

Initially, we only implemented the API and DAO layer, but as the application
grew and our API endpoints are getting messy (especially with model
conversions), we needed to separate concerns further. The 3 layered architecture
is quite common and standard, which is what we went with.

#### SQL ORM (SQLAlchemy)

We used SQLAlchemy an Object Relational Mapper (ORM) library in Python. We chose
this over writing raw SQL code for the following reasons:

- **Database Agnostic**: Easily swap out to a different DB seamlessly. For
  example swapping in SQLite for testing or migrating to a different DB.

- **Isolated Transaction/Connection**: Every request is yielded a new
  session/connection in the connection pool, which creates thread safe requests,
  and allows us to isolate it to its own transaction, which allows for rollback
  on any failures.

### Scalability (Performance Considerations)

We primarily focused on optimizing for read speeds. Which were done in two ways: fanout-on-write timeline and server-sided cache with Redis.

#### Fanout-On-Write Timeline

When designing a user timeline for a social media app.

#### Server-Sided Cache (Redis)

One of the standard ways to improve read performance in a system design is to introduce a cache. Redis is the most popular server-sided cache and is used as a layer between the server and the DB.

There are quite a few caching strategies, such as _cache-aside_, _read-through_, and _write_through_. We chose to use the **cache-aside** strategy also known as the _lazy-loading_ strategy.

The **cache-aside** strategy reads from the cache, on a cache miss event, the application reads from the database and updates the cache. It's very similar to the _read-through_ strategy, except instead of the cache querying the db and updating the itself, the application handles the db query and cache update logic.

The reason we chose this strategy is because to only cache _hot_ or frequently read data. There is no need to update the cache on a write event, because that write event could belong to a user with very few followers, not many reads will come through for their posts. We want to **maximize** our **cache hits** so we only cache data users are currently actively engaging with.



### Project setup

- PDM

  - reproducibility
  - manage and document dependency
  - standardize pyproject setup for distribution/builds

- Docker/Docker compose
- pre-commit
  - keep code structure consistent
  - auto formatting/linting
  - No need to fight about nit picks in code

## Lessons Learned

- Testing. We did not have enough upfront unit tests. Made it difficult to
  refactor later. Manual testing endpoints were also time consuming (e.g.,
  testing image uploads)

- Configuration. We tried to do a single global setting files, but were drifted
  for certain "temporary" setups, resulting in some settings and configs being
  outside the main setting/config file. Making navigating the code more
  difficult

- Model conversion. Translating between DB models and Pydantic models was a big
  challenge.



<!-- TODO: add graphics -->