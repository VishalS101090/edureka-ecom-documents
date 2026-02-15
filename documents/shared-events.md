# Shared Events / DTOs

Purpose: central guidance for event DTOs shared between services (Kafka producers/consumers).

Recommendations:

- Create a small Maven module (for example `shared-events`) containing event DTOs used across services.
- Publish it to your internal artifact repository (or include it as a multi-module build) so all services depend on the exact same classes.
- Keep DTOs simple POJOs with Lombok annotations where appropriate.
- Ensure Kafka serialization/deserialization settings match across services (use Spring's `JsonSerializer` and `JsonDeserializer`).
- Consumers should set `spring.kafka.consumer.properties.spring.json.trusted.packages` to include the package where shared DTOs live.

Example DTO (Java):

```java
package com.edureka.shared.event;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class OrderPlacedEvent {
    private String orderNumber;
    private String email;
}
```

Consumer properties hint (application.properties):

```
spring.kafka.consumer.properties.spring.json.trusted.packages=com.edureka.shared.event
```

Topic naming:
- Keep topic names stable and document them here (e.g., `notificationTopic`).

Versioning:
- If DTOs change in incompatible ways, create a new event type or version the topic payload to avoid deserialization issues.
