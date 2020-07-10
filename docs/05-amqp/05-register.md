---
layout: default
title: Register Attendees
description: Setup a REST endpoint that accepts attendees' registration
lang: en
parent: Advanced Message Queuing Protocol
nav_order: 5
permalink: docs/amqp/register/
---

# Register Attendees
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Objectives

- [ ] Attendees cannot register to an event that does not exist
- [ ] Attendees cannot register to an expired event (event that happened in the past)
- [ ] Attendees should receive a confirmation following a successful registration

## Description

We need to add a new REST endpoint, `/event/{event-id}/register`, that accepts POST requests to register attendees.  The event id, in the form of [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier), will be part of the path, as shown next.

```
/event/47705b9b-518b-4dc2-a517-3dbbcab13fe7/register
```

The POST request will also have a body similar to the following example.

{% include custom/note.html details="The event id is not part of the request body, but only part of the request endpoint." %}

```json
{
  "name": "Aden Attard",
  "foodPreference": "VEGETARIAN"
}
```

In the above example, _Aden Attard_ submitted his registration for the event with id `47705b9b-518b-4dc2-a517-3dbbcab13fe7` and picked `VEGETARIAN` as his food of choice.

Following is a complete example of a request, using [CURL](https://curl.haxx.se/), that can be made to the REST endpoint **once this is created**.

```bash
$ curl \
 -H "content-type:application/json" \
 -X POST -d'{"name":"Aden Attard","foodPreference":"VEGETARIAN"}' \
 http://localhost:8080/event/47705b9b-518b-4dc2-a517-3dbbcab13fe7/register
```

The attendee should either receive a [`404`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404), meaning that the registration failed as the event does not exist or has expired.  If the registration succeeds, then the attendee should receive the registration id, similar to the following example.

```json
{
  "id": "248ed31c-35e3-4bd8-a36e-a5c27d3eb999"
}
```

The registration id is another UUID which is generated by the registration process.

## High-level design

The controller will receive the POST request from the attendee, will capture the information received from both the path variable and the request body and pass this as a single object to the service, as shown in the following sequence diagram.

![Contact-Us-Controller-Service-DB-Sequence-Diagram.png]({{ '/assets/images/Contact-Us-Controller-Service-DB-Sequence-Diagram.png' | absolute_url }})

The service will process the registration and will reply to the controller.  The controller will then parse the service response and responds to the attendee, either a `404` or the confirmation id, based on the response received from the service.  The following diagram shown next shows the objects exchanged between the controller and the service.

![Contact-Us-Controller-Service-Exchange-Of-Objects.png]({{ '/assets/images/Contact-Us-Controller-Service-Exchange-Of-Objects.png' | absolute_url }})

The service will use a repository to interact with the database.  The object that is received from the controller is mapped to an entity, which is then passed to the repository.  The service needs to first verify that the event exists and is not expired.  In these two cases, the service should simply return an empty optional.

We have two entities.
* Events
* Attendees

One or more attendees may attend an event.  We need to represent these into two tables, as shown next.

![One-Event-Many-Attendees.png]({{ '/assets/images/One-Event-Many-Attendees.png' | absolute_url }})

We can take advantage of JPA and use one repository to persist both the events and attendees entities.

## Controller

We will start from the frontend, the controller, and we will work our way back.

1. Create new `event` package

   ```bash
   $ mkdir src/main/java/demo/boot/event
   $ mkdir src/test/java/demo/boot/event
   $ mkdir src/test-integration/java/demo/boot/event
   ```

1. Create the controller

   Create file: `src/main/java/demo/boot/event/EventRegistrationController.java`

   ```java
   package demo.boot.event;

   import org.springframework.web.bind.annotation.RestController;

   @RestController
   public class EventRegistrationController {

   }
   ```

1. Create the controller test

   Create file: `src/test/java/demo/boot/event/EventRegistrationControllerTest.java`

   ```java
   package demo.boot.event;

   import org.junit.jupiter.api.DisplayName;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;

   @DisplayName( "Registration controller" )
   @WebMvcTest( EventRegistrationController.class )
   public class EventRegistrationControllerTest {

   }
   ```

1. Test for registration for an event that does not exist or has expired

   {% include custom/note.html details="From the controller point-of-view expired events are treated the same as event not found.  The service will return an <a href='https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Optional.html#empty()'>empty optional</a> for both cases." %}

   Update file: `src/test/java/demo/boot/event/EventRegistrationControllerTest.java`

   {% include custom/dose_not_compile.html %}

   ```java
   package demo.boot.event;

   import com.fasterxml.jackson.databind.ObjectMapper;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
   import org.springframework.boot.test.mock.mockito.MockBean;
   import org.springframework.http.MediaType;
   import org.springframework.test.web.servlet.MockMvc;

   import java.nio.charset.StandardCharsets;
   import java.util.Optional;
   import java.util.UUID;

   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;
   import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

   @DisplayName( "Registration controller" )
   @WebMvcTest( EventRegistrationController.class )
   public class EventRegistrationControllerTest {

     @Autowired
     private MockMvc mockMvc;

     @MockBean
     private EventRegistrationService service;

     @Autowired
     private ObjectMapper jsonObjectMapper;

     @Test
     @DisplayName( "should return not found when registering for an event that does not exists" )
     public void shouldReturnNotFound() throws Exception {
       final UUID eventId = UUID.randomUUID();
       final String name = "Aden Attard";
       final FoodPreference foodPreference = FoodPreference.MEAT;
       final RegistrationRequest registrationRequest = new RegistrationRequest( name, foodPreference );
       final RegistrationDetails details = new RegistrationDetails( eventId, name, foodPreference );

       when( service.register( eq( details ) ) ).thenReturn( Optional.empty() );

       mockMvc
         .perform(
           post( "/event/{eventId}/register", eventId )
             .contentType( MediaType.APPLICATION_JSON )
             .characterEncoding( StandardCharsets.UTF_8.displayName() )
             .content( jsonObjectMapper.writeValueAsString( registrationRequest ) )
         )
         .andExpect( status().isNotFound() )
       ;

       verify( service, times( 1 ) ).register( details );
       verifyNoMoreInteractions( service );
     }
   }
   ```

   Make the test compile.

   1. Create file: `src/main/java/demo/boot/event/FoodPreference.java`

      ```java
      package demo.boot.event;

      public enum FoodPreference {
        NO_FOOD,
        VEGETARIAN,
        VEGAN,
        MEAT
      }
      ```

   1. Create file: `src/main/java/demo/boot/event/RegistrationDetails.java`

      ```java
      package demo.boot.event;

      import lombok.AllArgsConstructor;
      import lombok.Data;
      import lombok.NoArgsConstructor;

      import java.util.UUID;

      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class RegistrationDetails {

        private UUID eventId;
        private String name;
        private FoodPreference foodPreference;
      }
      ```

   1. Create file: `src/main/java/demo/boot/event/RegistrationConfirmation.java`

      ```java
      package demo.boot.event;

      import lombok.AllArgsConstructor;
      import lombok.Data;
      import lombok.NoArgsConstructor;

      import java.util.UUID;

      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class RegistrationConfirmation {

        private UUID id;
      }
      ```

   1. Create file: `src/main/java/demo/boot/event/EventRegistrationService.java`

      ```java
      package demo.boot.event;

      import java.util.Optional;

      public class EventRegistrationService {

        public Optional<RegistrationConfirmation> register( final RegistrationDetails registration ) {
          return Optional.empty();
        }
      }
      ```

   1. Create file: `src/main/java/demo/boot/event/RegistrationRequest.java`

      ```java
      package demo.boot.event;

      import lombok.AllArgsConstructor;
      import lombok.Data;
      import lombok.NoArgsConstructor;

      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class RegistrationRequest {

        private String name;
        private FoodPreference foodPreference;
      }
      ```

   The test should now compile.  Run the test.

   ```bash
   $ ./gradlew clean test

   ...

   Registration controller > should return not found when registering for an event that does not exists FAILED
       org.mockito.exceptions.verification.WantedButNotInvoked at EventRegistrationControllerTest.java:58

   ...

   BUILD FAILED in 9s
   5 actionable tasks: 5 executed
   ```

   The test will fail, as expected.

   {% include custom/note.html details="The test had received a 404, as expected but the controller did not interact with the mocks as expected" %}

   ```bash
   $ open "build/reports/tests/test/classes/demo.boot.event.EventRegistrationControllerTest.html"
   ```

   ![Event-Registration-Controller-Test-shouldReturnNotFound.png]({{ '/assets/images/Event-Registration-Controller-Test-shouldReturnNotFound.png' | absolute_url }})

1. Make the test pass

   Update file: `src/main/java/demo/boot/event/EventRegistrationController.java`

   ```java
   package demo.boot.event;

   import lombok.AllArgsConstructor;
   import org.springframework.http.ResponseEntity;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;

   import java.util.UUID;

   @RestController
   @AllArgsConstructor
   public class EventRegistrationController {

     private final EventRegistrationService service;

     @PostMapping( "/event/{eventId}/register" )
     public ResponseEntity<RegistrationConfirmation> register(
       @PathVariable( "eventId" ) final UUID eventId,
       @RequestBody final RegistrationRequest request
     ) {
       final RegistrationDetails details =
         new RegistrationDetails( eventId, request.getName(), request.getFoodPreference() );

       service.register( details );
       return ResponseEntity.notFound().build();
     }
   }
   ```

   {% include custom/note.html details="Annotations are polluting the <code>register()</code> method." %}

   ```java
     @PostMapping( "/event/{eventId}/register" )                /* a */
     public ResponseEntity<RegistrationConfirmation> register(
       @PathVariable( "eventId" ) final UUID eventId,           /* b */
       @RequestBody final RegistrationRequest request           /* c */
     ) {
   ```

   1. Maps the `register()` method to our POST requests and defined the `{eventId}` path variable
   1. Extracts the `{eventId}` path variable into the method parameter, `eventId`
   1. Parse the POST request JSON body into `RegistrationRequest`

   Run the tests.

   ```bash
   $ ./gradlew clean test

   ...

   BUILD SUCCESSFUL in 7s
   5 actionable tasks: 5 executed
   ```

   All tests should now pass.

1. Test for registration for an active event

   Update file: `src/test/java/demo/boot/event/EventRegistrationControllerTest.java`

   ```java
   package demo.boot.event;

   import com.fasterxml.jackson.databind.ObjectMapper;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
   import org.springframework.boot.test.mock.mockito.MockBean;
   import org.springframework.http.MediaType;
   import org.springframework.test.web.servlet.MockMvc;

   import java.nio.charset.StandardCharsets;
   import java.util.Optional;
   import java.util.UUID;

   import static org.hamcrest.Matchers.is;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;
   import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.header;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

   @DisplayName( "Registration controller" )
   @WebMvcTest( EventRegistrationController.class )
   public class EventRegistrationControllerTest {

     @Autowired
     private MockMvc mockMvc;

     @MockBean
     private EventRegistrationService service;

     @Autowired
     private ObjectMapper jsonObjectMapper;

     @Test
     @DisplayName( "should return not found when registering for an event that does not exists" )
     public void shouldReturnNotFound() throws Exception { /* ... */ }

     @Test
     @DisplayName( "should return the registration confirmation when registering for an existing event" )
     public void shouldReturnConfirmation() throws Exception {
       final UUID eventId = UUID.randomUUID();
       final UUID confirmationId = UUID.randomUUID();
       final String name = "Jade Attard";
       final FoodPreference foodPreference = FoodPreference.MEAT;
       final RegistrationRequest registrationRequest = new RegistrationRequest( name, foodPreference );
       final RegistrationDetails details = new RegistrationDetails( eventId, name, foodPreference );

       when( service.register( eq( details ) ) ).thenReturn( Optional.of( new RegistrationConfirmation( confirmationId ) ) );

       mockMvc
         .perform(
           post( "/event/{eventId}/register", eventId )
             .contentType( MediaType.APPLICATION_JSON )
             .characterEncoding( StandardCharsets.UTF_8.displayName() )
             .content( jsonObjectMapper.writeValueAsString( registrationRequest ) )
         )
         .andExpect( status().isCreated() )
         .andExpect( header().string( "Location", String.format( "/event/registration/%s", confirmationId ) ) )
         .andExpect( jsonPath( "$" ).isMap() )
         .andExpect( jsonPath( "$.id", is( confirmationId.toString() ) ) )
       ;

       verify( service, times( 1 ) ).register( details );
       verifyNoMoreInteractions( service );
     }
   }
   ```

   Run the tests.

   ```bash
   $ ./gradlew clean test

   ...

   Registration controller > should return the registration confirmation when registering for an existing event FAILED
       java.lang.AssertionError at EventRegistrationControllerTest.java:84

   BUILD FAILED in 7s
   5 actionable tasks: 5 executed
   ```

   The test should fail.

1. Make the test pass

   Update file: `src/main/java/demo/boot/event/EventRegistrationController.java`

   ```java
   package demo.boot.event;

   import lombok.AllArgsConstructor;
   import org.springframework.http.ResponseEntity;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;

   import java.net.URI;
   import java.util.UUID;

   @RestController
   @AllArgsConstructor
   public class EventRegistrationController {

     private final EventRegistrationService service;

     @PostMapping( "/event/{eventId}/register" )
     public ResponseEntity<RegistrationConfirmation> register(
       @PathVariable( "eventId" ) final UUID eventId,
       @RequestBody final RegistrationRequest request
     ) {
       final RegistrationDetails details =
         new RegistrationDetails( eventId, request.getName(), request.getFoodPreference() );

       return service
         .register( details )
         .map( confirmation -> ResponseEntity
           .created( URI.create( String.format( "/event/registration/%s", confirmation.getId() ) ) )
           .body( confirmation ) )
         .orElse( ResponseEntity.notFound().build() );
     }
   }
   ```

   Run the tests.

   ```bash
   $ ./gradlew clean test

   ...

   BUILD SUCCESSFUL in 8s
   5 actionable tasks: 5 executed
   ```

   The test should pass.

1. Refactor the controller

   Update file: `src/main/java/demo/boot/event/EventRegistrationController.java`

   ```java
   package demo.boot.event;

   import lombok.AllArgsConstructor;
   import org.springframework.http.ResponseEntity;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.PostMapping;
   import org.springframework.web.bind.annotation.RequestBody;
   import org.springframework.web.bind.annotation.RestController;

   import java.net.URI;
   import java.util.UUID;

   @RestController
   @AllArgsConstructor
   public class EventRegistrationController {

     private static final ResponseEntity<RegistrationConfirmation> NOT_FOUND = ResponseEntity.notFound().build();

     private final EventRegistrationService service;

     @PostMapping( "/event/{eventId}/register" )
     public ResponseEntity<RegistrationConfirmation> register(
       @PathVariable( "eventId" ) final UUID eventId,
       @RequestBody final RegistrationRequest request
     ) {
       final RegistrationDetails details =
         new RegistrationDetails( eventId, request.getName(), request.getFoodPreference() );

       return service
         .register( details )
         .map( this::mapToResponse )
         .orElse( NOT_FOUND );
     }

     private ResponseEntity<RegistrationConfirmation> mapToResponse( final RegistrationConfirmation confirmation ) {
       return ResponseEntity
         .created( createLocationHeader( confirmation ) )
         .body( confirmation );
     }

     private URI createLocationHeader( final RegistrationConfirmation confirmation ) {
       return URI.create( String.format( "/event/registration/%s", confirmation.getId() ) );
     }
   }
   ```

The controller is complete.  It parses the request into Java objects and invokes the service and then reply to the attendee based on the service's response.

## Service

{% include custom/pending.html %}

1. Create test class

   Create file: `src/test/java/demo/boot/event/EventRegistrationServiceTest.java`

   ```java
   package demo.boot.event;

   import org.junit.jupiter.api.DisplayName;

   @DisplayName( "Event registration service" )
   public class EventRegistrationServiceTest {

   }
   ```

1. Test for registration for an event that does not exist

   {% include custom/note.html details="The service needs two tests to cover events that do not exist and expired events.  In the latter, the repository will return an optional with the expired entity, while in the former case the repository will return an empty optional." %}

   Update file: `src/test/java/demo/boot/event/EventRegistrationServiceTest.java`

   {% include custom/dose_not_compile.html %}

   ```java
   package demo.boot.event;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.Optional;
   import java.util.UUID;

   import static org.junit.jupiter.api.Assertions.assertEquals;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "Event registration service" )
   public class EventRegistrationServiceTest {

     @Test
     @DisplayName( "should return Optional empty when registering to an non existing event" )
     public void shouldReturnOptionalEmptyWhenNotFound() {
       final EventRepository eventRepository = mock( EventRepository.class );

       final UUID eventId = UUID.randomUUID();
       final String name = "Albert Attard";
       final FoodPreference foodPreference = FoodPreference.MEAT;
       final RegistrationDetails details = new RegistrationDetails( eventId, name, foodPreference );

       when( eventRepository.findById( eq( eventId ) ) ).thenReturn( Optional.empty() );

       final EventRegistrationService service = new EventRegistrationService( eventRepository );
       final Optional<RegistrationConfirmation> confirmation = service.register( details );
       assertEquals( Optional.empty(), confirmation );

       verify( eventRepository, times( 1 ) ).findById( eventId );
       verifyNoMoreInteractions( eventRepository );
     }
   }
   ```

   Make the test compile.

   1. Create file: `src/main/java/demo/boot/event/EventRepository.java`

      {% include custom/note.html details="We are using an <code>Object</code> (<code>extends JpaRepository&lt;Object, UUID&gt;</code>) and not our entity, which is not yet created." %}

      ```java
      package demo.boot.event;

      import org.springframework.data.jpa.repository.JpaRepository;

      import java.util.UUID;

      public interface EventRepository extends JpaRepository<Object, UUID> {
      }
      ```

   1. Update file: `src/main/java/demo/boot/event/EventRegistrationService.java`

      ```java
      package demo.boot.event;

      import lombok.AllArgsConstructor;

      import java.util.Optional;

      @AllArgsConstructor
      public class EventRegistrationService {

        private final EventRepository repository;

        public Optional<RegistrationConfirmation> register( final RegistrationDetails registration ) {
          return Optional.empty();
        }
      }
      ```

   The test should now compile.  Run the test.

   ```bash
   $ ./gradlew clean test

   ...

   Event registration service > should return Optional empty when registering to an non existing event FAILED
       org.mockito.exceptions.verification.WantedButNotInvoked at EventRegistrationServiceTest.java:36

   ...

   BUILD FAILED in 9s
   5 actionable tasks: 5 executed
   ```

   The test will fail, as expected.

1. Make the test pass

   Update file: `src/main/java/demo/boot/event/EventRegistrationService.java`

   {% include custom/note.html details="The following example is just enough to make the test pass." %}

   ```java
   package demo.boot.event;

   import lombok.AllArgsConstructor;

   import java.util.Optional;

   @AllArgsConstructor
   public class EventRegistrationService {

     private final EventRepository repository;

     public Optional<RegistrationConfirmation> register( final RegistrationDetails registration ) {
       repository.findById( registration.getEventId() );
       return Optional.empty();
     }
   }
   ```

   Run the tests.

   ```bash
   ./gradlew clean test

   ...

   BUILD SUCCESSFUL in 7s
   5 actionable tasks: 5 executed
   ```

   All tests should now pass.

## Repository

{% include custom/pending.html %}