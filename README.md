# Subscriber-Controlled Pattern

Allow **subscribers** (listeners or observers) to **manage their own lifecycle**, including when and how to unsubscribe from a publisher, rather than relying on the publisher or external controllers to handle cleanup logic.

## Visual Representation 
```bash
+--------------------+        notify()         +---------------------+
|     Publisher      | ----------------------> |     Subscriber      |
| (Notifier/Subject) |                         | (Listener/Observer) |
+--------------------+                         +---------------------+
         ^                                                |
         |                                                |
         +------------------- release() <-----------------+

```
## Participants

- **Publisher (Notifier/Subject)**  
  - Manages a list of subscribers.  
  - Provides each subscriber with a `release()` function during registration.

- **Subscriber (Listener/Observer)**  
  - Listens to notifications as usual.  
  - Stores and invokes `release()` based on internal logic, conditions, or side-effects.

## Collaborations

1. Publisher calls `addListener(listener)`
2. Publisher generates a `release()` function and passes it to `listener.onCleanup(release)`
3. Listener stores the `release()` function and invokes it based on its own lifecycle
4. When `release()` is called, the publisher removes that listener from its internal list
## Problems with Traditional Cleanup

### Manual Unregistration
- Easy to forget  
- Memory leaks  
- Scattered, repetitive code

### Framework Auto-Cleanup
- Hard to understand  
- Doesn’t play well with other event systems

## Works In
- JavaScript  
- Python  
- Ruby  
- Lua  
(Any language with first-class functions)

## JavaScript Example
### Basic Implementation
This example demonstrates the core idea of the Subscriber Self Control Pattern (Subscriber-Controlled), where the subscriber controls cleanup via `onCleanup`:
```js
class Notifier {
  constructor() {
    this.listeners = new Set();
  }

  addListener(listener) {
    const release = () => this.removeListener(listener);
    if (!this.listeners.has(listener)) {
      this.listeners.add(listener);
      if (typeof listener.onCleanup === 'function') {
        listener.onCleanup(release); // Subscriber Self Control Pattern
      }
    }
    return release;
  }

  removeListener(listener) {
    this.listeners.delete(listener);
  }

  notify(...args) {
    this.listeners.forEach(listener => listener(...args));
  }
}

```
```js
// Example listener
function listener(data) {
  console.log('Received:', data);
}
// Optional auto-cleanup: unregister after 5 seconds
listener.onCleanup = (release) => {
  setTimeout(release, 5000);
};
let notifier = new Notifier()
notifier.addListener(listener);
// Later usage
notifier.notify('event fired!');
```
### Production-Ready Implementation
For production environments, the Notifier should support multiple event types, include error handling, and provide additional cleanup options. Below is a robust implementation:
- Javascript Official [event-notifier](https://github.com/huukhanhnguyen/event-notifier)
## Benefits
- **Programmable Unsubscription**: The `listener.onCleanup` function allows subscribers to define custom unsubscription logic, triggered by registered events (e.g., timeouts, user actions, or specific conditions), ensuring reliable cleanup without manual intervention.
- **Clear Intent:** `onCleanup` makes cleanup explicit
- **Flexible:** Listeners choose *when* to unregister (e.g., timeout, condition)
- **Minimal Boilerplate:** Keeps logic local to listener
  ## Extending Subscriber-Controlled for State Management and Reactivity
The Subscriber Self Control Pattern (Subscriber-Controlled) is well-suited for creating state-driven objects, such as those used in reactive programming or UI frameworks. By extending the `Notifier` class, you can build objects that manage state changes and notify listeners efficiently, while allowing subscribers to control their own cleanup. This reduces memory leaks and simplifies state management in dynamic applications.

- **State-Driven Objects**: Subscriber-Controlled enables listeners to react to state changes (e.g., via a "change" event) and unsubscribe when no longer needed.
- **Reactive Applications**: Perfect for scenarios where state updates trigger UI updates or other side effects.
- **Clean Resource Management**: Subscribers define their own cleanup logic, ensuring resources are released appropriately.

### Example: State Management
Below is an example of a `State` class that extends `Notifier` to manage state changes:
```javascript
class State extends Notifier {
    constructor(value) {
        super(); // Initialize Notifier's listeners
        this.value = value;
    }

    setValue(newValue) {
        if (this.value !== newValue) { // Only notify on actual changes
            this.value = newValue;
            this.notify('change', newValue); // Notify listeners of state change
        }
    }

    getValue() {
        return this.value; // Getter for accessing current state
    }
}
```
```
const state = new State(0); // Initial state: 0

function listener(newValue) {
    console.log('State changed to:', newValue);
}
listener.onCleanup = (release) => {
    setTimeout(release, 5000); // Unsubscribe after 5 seconds
    window.addEventListener('click', release, { once: true }); // Or on user click
};

state.addListener('change', listener);
state.setValue(1); // Logs: State changed to: 1
state.setValue(2); // Logs: State changed to: 2
```
## Useful Use Cases 
- Complex reactive applications (e.g., UI frameworks)
- Dynamic node graphs (e.g., visual scripting, geometry modeling)
- Event-driven systems with long-lived objects
- Temporary observers (e.g., animation triggers, game logic)
- Resource managers (e.g., network, cache, file watching)
- Logging or debugging hooks with limited lifespan
- Modular plugin systems where listeners self-register
License: MIT Copyright: 2025 Author: [Khánh Nguyễn](https://github.com/huukhanhnguyen)
