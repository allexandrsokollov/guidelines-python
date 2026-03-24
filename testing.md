Here is the English translation:

**Core Principles**

**1. Test behavior, not internal implementation**
Tests should verify not that one method called another, but that the system as a whole behaved as expected.

For example, instead of checking whether a specific function was called, it is more useful to verify:

* what HTTP response the endpoint returned;
* what data was saved in the database;
* whether a message was published to the broker;
* whether the state of a business entity changed;
* whether the expected side effect occurred.

Such tests are much more resilient to refactoring. If the internal implementation changes but the system’s behavior remains the same, the tests should not break.

**2. Use test containers**
One of the key elements of this approach is the use of test containers to spin up real dependencies: databases, cache, message broker, and other services required by the application.

This is important for several reasons:

* we avoid a significant portion of mocks;
* we use real mechanisms for creating and reading objects;
* we verify data serialization and deserialization;
* we verify object mapping;
* we test the handling of incoming and outgoing messages;
* we verify the application’s interaction with the infrastructure it actually runs on.

As a result, tests become closer to the production environment and help uncover errors that cannot be detected with fully isolated testing.

At the same time, it is important to remember that the closer tests are to the real environment, the more important determinism becomes: state must be isolated, data must be controlled, and results must be reproducible.

**3. Minimize mocks, but do not abandon them entirely**
The main focus should be on testing with real internal components of the system, but a complete rejection of mocks is not always justified.

Mocks are still useful when dealing with external boundaries of our area of responsibility:

* third-party HTTP APIs;
* payment systems;
* email and SMS providers;
* unstable external integrations;
* expensive or slow external calls;
* hard-to-reproduce error-case scenarios.

So the principle can be formulated as follows:
everything related to our internal logic and infrastructure should, whenever possible, be tested for real; everything outside our control may be mocked.

**4. Use factories for test data**
For convenient and scalable creation of test data, it is useful to use factories, for example based on factory_boy.

Once factories for the main entities have been defined, we get the following benefits:

* faster writing of new tests;
* less duplicated setup code;
* more understandable test structure;
* a consistent way of creating objects;
* reduced cognitive load when reading tests.

Factories are especially useful in combination with integration tests because they allow you to quickly assemble full scenarios with minimal noise in the code.

**5. Endpoint-based testing**
To verify application functionality, it makes sense to call endpoints directly, since the endpoint is the actual point of interaction between the client and the system.

This approach allows multiple layers to be tested at once:

* routing;
* input validation;
* business logic;
* response format and structure;
* database interaction;
* side effects;
* interaction with queues, cache, and other components.

The advantage of this approach is that we test not a single function “in a vacuum,” but a complete scenario that a real user or another service actually works with.

Combined with test containers, this makes it possible to bring the test environment as close as possible to production and catch more errors during development.

**6. Test not only the happy path**
One of the common problems with test suites is an excessive focus on successful scenarios. In practice, however, a significant portion of errors arise precisely in edge cases and negative scenarios.

Therefore, in addition to the happy path, tests should cover:

* invalid input data;
* missing required fields;
* state conflicts;
* duplicates;
* constraint violations;
* authorization errors and insufficient permissions;
* repeated message delivery;
* empty or partially filled data;
* non-standard state transitions.

Tests for such scenarios often provide more value than a large number of superficial checks of internal calls.

**7. Verify not only the response, but also the side effects**
If a test calls an endpoint, it is not enough to check only the HTTP status code and the response body. Often, the main value of the scenario lies in what happened after the call.

For example, it is important to additionally verify:

* whether a record was created in the database;
* whether the object’s status changed;
* whether an event was sent to the broker;
* whether the cache was updated;
* whether an audit log was created;
* whether a background task was queued.

This makes tests more complete and allows them to truly verify business functionality, not just the external response contract.

**8. Protect system contracts**
If the system interacts with other services via HTTP APIs or a message broker, tests must protect not only internal logic but also contracts.

This may include verifying:

* the structure of JSON responses;
* required fields;
* data types;
* the format of serialized messages;
* contract compatibility when changes are introduced.

This is especially important in distributed systems, where a change in the response or event format can break other services even if the internal logic continues to work correctly.

**9. Use the AAA structure**
To improve readability and consistency of tests, it is useful to follow the AAA structure (Arrange, Act, Assert):

* **Arrange** — prepare the data and environment;
* **Act** — perform the action being tested;
* **Assert** — verify the result.

This approach helps make tests more structured, understandable, and maintainable. It works especially well in combination with factories and endpoint-based testing, where it is important not to mix scenario setup, execution, and assertions into one hard-to-read sequence of actions.

I can also make it sound more natural and article-ready in English, not just directly translated.
