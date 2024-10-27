Here’s a full code implementation for a **Publish-Subscribe** system in Java that includes the following requirements:

1. **Publisher and Subscriber** classes with the ability to support multiple subscribers.
2. **Event Manager** to handle event publication and manage subscriptions.
3. **Concurrency Handling** to allow safe multi-threaded access.
4. **Exception Handling** for cases like invalid/unregistered subscribers.
5. **High Performance** using non-blocking data structures (e.g., `ConcurrentHashMap`) where needed.

The system follows the **Observer Pattern** and handles **multiple topics** to which subscribers can subscribe, unsubscribe, and get notified when there are events.

### Full Code

#### Enum: `EventType`
This enum represents different types of events (or topics) to which subscribers can subscribe.

```java
enum EventType {
    SPORTS, NEWS, TECHNOLOGY, ENTERTAINMENT;
}
```

#### Interface: `Subscriber`
Defines the basic interface for a subscriber.

```java
interface Subscriber {
    void update(EventType eventType, String message);
}
```

#### Class: `EventManager`
Handles subscription, unsubscription, and event publishing for various event types. It uses `ConcurrentHashMap` to manage lists of subscribers per event type, ensuring thread-safe operations.

```java
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CopyOnWriteArrayList;

class EventManager {
    private final Map<EventType, List<Subscriber>> subscribers = new ConcurrentHashMap<>();

    public EventManager() {
        // Initialize lists for each event type
        for (EventType type : EventType.values()) {
            subscribers.put(type, new CopyOnWriteArrayList<>());
        }
    }

    // Subscribe a subscriber to an event type
    public void subscribe(EventType eventType, Subscriber subscriber) {
        List<Subscriber> subscriberList = subscribers.get(eventType);
        if (!subscriberList.contains(subscriber)) {
            subscriberList.add(subscriber);
            System.out.println("Subscriber added to " + eventType + " events.");
        }
    }

    // Unsubscribe a subscriber from an event type
    public void unsubscribe(EventType eventType, Subscriber subscriber) throws Exception {
        List<Subscriber> subscriberList = subscribers.get(eventType);
        if (subscriberList.contains(subscriber)) {
            subscriberList.remove(subscriber);
            System.out.println("Subscriber removed from " + eventType + " events.");
        } else {
            throw new Exception("Subscriber not found for event type: " + eventType);
        }
    }

    // Publish an event to all subscribers of a specific type
    public void publish(EventType eventType, String message) {
        List<Subscriber> subscriberList = subscribers.get(eventType);
        for (Subscriber subscriber : subscriberList) {
            subscriber.update(eventType, message);
        }
        System.out.println("Message published to all subscribers of " + eventType + ": " + message);
    }
}
```

#### Class: `Publisher`
Represents a publisher that can publish messages to different event types via the `EventManager`.

```java
class Publisher {
    private final EventManager eventManager;

    public Publisher(EventManager eventManager) {
        this.eventManager = eventManager;
    }

    // Publish a message to a specific event type
    public void publishMessage(EventType eventType, String message) {
        eventManager.publish(eventType, message);
    }
}
```

#### Class: `ConcreteSubscriber`
An example subscriber that implements the `Subscriber` interface, allowing it to receive messages for subscribed event types.

```java
class ConcreteSubscriber implements Subscriber {
    private final String name;

    public ConcreteSubscriber(String name) {
        this.name = name;
    }

    @Override
    public void update(EventType eventType, String message) {
        System.out.println("Subscriber " + name + " received " + eventType + " update: " + message);
    }
}
```

#### Main Class: `PublishSubscribeSystem`
Demonstrates how the publish-subscribe system works by creating an `EventManager`, `Publisher`, and multiple subscribers. It subscribes them to topics, publishes messages, and showcases unsubscription.

```java
public class PublishSubscribeSystem {
    public static void main(String[] args) {
        // Create an event manager
        EventManager eventManager = new EventManager();

        // Create a publisher
        Publisher publisher = new Publisher(eventManager);

        // Create subscribers
        Subscriber sportsSubscriber = new ConcreteSubscriber("Alice");
        Subscriber newsSubscriber = new ConcreteSubscriber("Bob");
        Subscriber techSubscriber = new ConcreteSubscriber("Charlie");

        // Subscribe subscribers to various topics
        eventManager.subscribe(EventType.SPORTS, sportsSubscriber);
        eventManager.subscribe(EventType.NEWS, newsSubscriber);
        eventManager.subscribe(EventType.TECHNOLOGY, techSubscriber);
        eventManager.subscribe(EventType.SPORTS, techSubscriber); // Charlie also subscribes to SPORTS

        // Publish events
        publisher.publishMessage(EventType.SPORTS, "New football match scheduled!");
        publisher.publishMessage(EventType.NEWS, "Breaking news: Market hits a new high.");
        publisher.publishMessage(EventType.TECHNOLOGY, "New smartphone released!");

        // Unsubscribe a subscriber
        try {
            eventManager.unsubscribe(EventType.SPORTS, techSubscriber);
            publisher.publishMessage(EventType.SPORTS, "Sports event update after unsubscribe.");
        } catch (Exception e) {
            System.err.println(e.getMessage());
        }
    }
}
```

### Explanation of Code Components

1. **EventType**: Enum representing different topics (events).
2. **Subscriber Interface**: Provides an `update` method, which each subscriber implements to receive event updates.
3. **ConcreteSubscriber Class**: A sample subscriber that receives updates and processes them.
4. **EventManager Class**:
   - Manages subscriptions by keeping lists of subscribers for each event type.
   - Uses `ConcurrentHashMap` for thread-safe access.
   - Each subscriber list is a `CopyOnWriteArrayList` for thread-safe read and write operations.
5. **Publisher Class**: Allows the publishing of messages to a specific event type by interacting with `EventManager`.
6. **PublishSubscribeSystem**: A sample simulation to demonstrate adding and removing subscribers, publishing messages, and handling exceptions for unregistered subscribers.

### Key Features
- **Thread-Safety**: `ConcurrentHashMap` and `CopyOnWriteArrayList` ensure that adding, removing, and notifying subscribers are thread-safe operations.
- **Exception Handling**: Custom exceptions are thrown if a subscriber is not found when unsubscribing.
- **Flexibility**: New event types can be added to the `EventType` enum, and subscribers can subscribe to any combination of events.

This code structure provides a high-level, scalable, and thread-safe Publish-Subscribe system.




### consume in parallel thread subcriber example

To implement parallelism in the Publish-Subscribe system, we need to modify the code where subscriber updates are handled, specifically in the `EventManager` class during the publish operation. Instead of using a traditional loop to notify subscribers, we can use parallel streams or create new threads for each subscriber. Here, I'll show you the changes required to achieve this.

### Updated Code for Parallelism in the `EventManager` Class

#### Original Publish Method (without parallelism)

```java
// Publish a message to all subscribers of a specific topic
public void publish(Topic topic, String message) {
    List<Subscriber> subscriberList = subscribers.get(topic);
    for (Subscriber subscriber : subscriberList) {
        subscriber.update(topic, message);
    }
    System.out.println("Message published to " + topic + ": " + message);
}
```

#### Updated Publish Method (with Parallelism)

Here, we will use `CompletableFuture.runAsync` to run the update method of each subscriber asynchronously.

```java
import java.util.concurrent.CompletableFuture;

// Publish a message to all subscribers of a specific topic
public void publish(Topic topic, String message) {
    List<Subscriber> subscriberList = subscribers.get(topic);
    for (Subscriber subscriber : subscriberList) {
        CompletableFuture.runAsync(() -> subscriber.update(topic, message));
    }
    System.out.println("Message published to " + topic + ": " + message);
}
```

### Explanation of Changes
- **`CompletableFuture.runAsync(...)`**: This method allows each subscriber's `update` method to be executed asynchronously in a separate thread, enabling parallel processing of subscriber notifications.
- **No Blocking**: Using `CompletableFuture` ensures that the main thread can continue executing without waiting for all subscriber notifications to complete, enhancing responsiveness.


instead of future we use this also
  // Run subscriber updates in separate threads for parallelism

            new Thread(() -> subscriber.update(topic, message)).start();

### Conclusion
This change enables the Publish-Subscribe system to notify subscribers in parallel, improving the overall performance when multiple subscribers are listening to the same topic.



### advantage of future over thread
Using `CompletableFuture` instead of manually creating threads for handling parallelism in the Publish-Subscribe system has several advantages:

### Advantages of Using `CompletableFuture`

1. **Simplified Code**:
   - `CompletableFuture` abstracts away the complexities of thread management, making the code cleaner and more readable. You don’t have to worry about thread creation, lifecycle, or handling thread interruptions explicitly.

2. **Thread Pool Management**:
   - `CompletableFuture.runAsync` utilizes the common ForkJoinPool (a thread pool) to manage threads efficiently. This means that it can take advantage of available threads and resources without creating new threads for each subscriber, leading to better performance and resource utilization.

3. **Error Handling**:
   - `CompletableFuture` provides a fluent API that allows for easier error handling through methods like `exceptionally`, `handle`, and `whenComplete`. This makes it easier to manage exceptions that may arise during the execution of asynchronous tasks.

4. **Composability**:
   - `CompletableFuture` allows you to chain multiple asynchronous operations together, enabling complex workflows with minimal code. This is useful if you want to perform additional actions after subscriber updates or combine multiple asynchronous tasks.

5. **Improved Performance**:
   - By using a thread pool, `CompletableFuture` can effectively manage the number of threads, leading to better performance, especially when there are many subscribers. Creating a new thread for each subscriber can quickly exhaust system resources, leading to performance degradation.

6. **Non-blocking Operations**:
   - `CompletableFuture` allows for non-blocking programming, which can be beneficial in environments where you want to maintain responsiveness (e.g., UI applications or networked applications).

### Example Comparison

Here’s a brief comparison of both approaches for context:

#### Using Threads
```java
for (Subscriber subscriber : subscriberList) {
    new Thread(() -> subscriber.update(topic, message)).start();
}
```
- Each `new Thread(...)` creates a new thread, which incurs overhead in thread creation and management.
- If there are many subscribers, this can lead to resource exhaustion and inefficiencies.

#### Using CompletableFuture
```java
for (Subscriber subscriber : subscriberList) {
    CompletableFuture.runAsync(() -> subscriber.update(topic, message));
}
```
- `CompletableFuture.runAsync` uses a shared thread pool, optimizing resource usage.
- The code remains clean and concise, and handling errors or chaining tasks becomes easier.

### Conclusion
Using `CompletableFuture` enhances the overall design and performance of the Publish-Subscribe system. It leverages the power of Java’s concurrency utilities while keeping the implementation simple and efficient.