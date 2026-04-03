==============================
FILE: Low-Level Design (LLD)
==============================

### HIGH PRIORITY

---

Q1. What are the SOLID principles? Explain each with a concrete example.

A1.
**S — Single Responsibility**: A class should have one reason to change. If your `UserService` handles user creation, email sending, AND logging, any change to email logic forces you to touch the user service. Split it: `UserService`, `EmailService`, `AuditLogger`.

**O — Open/Closed**: Open for extension, closed for modification. Instead of adding `if payment_type == "upi"` to a growing switch statement, define a `PaymentProcessor` interface and add new types as new classes. Existing code doesn't change.

**L — Liskov Substitution**: A subclass should be usable wherever its parent is expected without breaking behavior. Classic violation: `Square extends Rectangle`. Setting width on a square shouldn't independently change height — it breaks Rectangle's contract.

**I — Interface Segregation**: Don't force clients to depend on methods they don't use. A `Worker` interface with `work()` and `eat()` forces a `Robot` to implement `eat()`. Split into `Workable` and `Feedable`.

**D — Dependency Inversion**: High-level modules shouldn't depend on low-level modules. Both should depend on abstractions. `OrderService` shouldn't instantiate `MySQLDatabase` directly — it should depend on a `Database` interface, and the concrete implementation is injected.

---

Q2. What is the Strategy pattern, and when would you use it instead of a chain of if-else?

A2.
The Strategy pattern encapsulates a family of algorithms behind a common interface, letting you swap them at runtime. Instead of:

```
if pricing == "standard": calculate_standard()
elif pricing == "premium": calculate_premium()
elif pricing == "enterprise": calculate_enterprise()
```

You define a `PricingStrategy` interface with a `calculate()` method, create concrete strategies (`StandardPricing`, `PremiumPricing`, etc.), and inject the right one into `OrderCalculator`. Adding a new pricing tier is just a new class — zero modifications to existing code.

Use it when you have multiple variants of an algorithm that swap based on context, configuration, or user type. It's the behavioral twin of "if you have a growing switch statement, you need a pattern."

---

Q3. What is the Factory pattern, and how does it differ from the Abstract Factory pattern?

A3.
**Factory Method**: A single method that decides which concrete class to instantiate based on input. `NotificationFactory.create("email")` returns an `EmailNotification` object. The caller works with the `Notification` interface and doesn't know or care about the concrete type.

**Abstract Factory**: A factory of factories. It creates families of related objects. `UIFactory` could be `WindowsUIFactory` or `MacUIFactory`. Each creates a consistent set: `WindowsButton + WindowsScrollbar` or `MacButton + MacScrollbar`. You swap the entire family by changing the factory.

Use Factory Method for creating a single product type with multiple variants. Use Abstract Factory when you need to create a set of related products that must be consistent — like a theme/skin system or a cross-platform UI toolkit.

---

Q4. What is the Observer pattern? How is it used in event-driven systems?

A4.
The Observer pattern defines a one-to-many relationship: when the subject's state changes, all registered observers are notified automatically. The subject doesn't know the concrete types of its observers — it just calls `notify()` on its subscriber list.

Classic example: a stock price ticker. Multiple displays (chart, table, alert system) observe the ticker. When the price changes, all displays update. Adding a new display type doesn't require changing the ticker.

In event-driven backend systems, this is essentially the pub/sub pattern. An `OrderService` publishes an "OrderPlaced" event. `InventoryService`, `NotificationService`, and `AnalyticsService` all subscribe. The order service is decoupled from all downstream consumers.

The risk: too many observers can make debugging hard — it's not always obvious what happens when an event fires. And observer chains (observer A triggers event B, which triggers observer C) can create surprising cascades.

---

Q5. What is the Singleton pattern? What are its drawbacks, and why do some consider it an anti-pattern?

A5.
Singleton ensures only one instance of a class exists globally, and provides a global access point to it. Database connection pools, configuration managers, and logger instances are common examples.

Why it's dangerous:
- **Hidden dependencies**: Any code anywhere can call `Singleton.getInstance()`, making dependencies invisible and hard to trace.
- **Testing nightmare**: You can't replace the singleton with a mock easily. Tests become entangled with global state.
- **Concurrency issues**: In multi-threaded environments, the lazy initialization pattern can create multiple instances unless carefully synchronized.
- **Tight coupling**: Code that uses the singleton is tightly coupled to that specific class.

Better alternative: use dependency injection to manage the lifecycle. Create one instance and inject it where needed. You get the "single instance" behavior without the global access anti-pattern. The DI container manages the lifecycle; the classes remain testable and decoupled.

---

Q6. How would you design a parking lot system? What classes and relationships would you define?

A6.
**Core entities**:
- `ParkingLot`: Has multiple `Floor`s, each with multiple `ParkingSpot`s.
- `ParkingSpot`: Has a type (compact, regular, large), status (available/occupied), and a reference to the `Vehicle` parked there.
- `Vehicle`: Abstract class with subclasses `Car`, `Truck`, `Motorcycle`. Each has a size that maps to spot types.
- `Ticket`: Created on entry — stores vehicle info, spot assignment, entry time.
- `ParkingRate`: Strategy pattern — hourly rate, flat rate, weekend rate.

**Key behaviors**:
- `ParkingLot.findAvailableSpot(vehicleType)`: Finds the nearest appropriate spot.
- `ParkingLot.issueTicket(vehicle)`: Assigns spot, creates ticket.
- `ParkingLot.processExit(ticket)`: Calculates fee based on duration and rate strategy, frees the spot.

**Design decisions**: Use a priority queue or floor-level counters for efficient spot finding. Apply Strategy pattern for pricing. Use Observer to update display boards when spot availability changes.

---

Q7. How would you design an elevator system? What states and transitions does it need?

A7.
**States**: Idle, MovingUp, MovingDown, DoorOpen. Transitions are driven by requests.

**Core classes**:
- `Elevator`: Has current floor, direction, state, and a list of requests.
- `ElevatorController`: Manages multiple elevators, dispatches requests using a scheduling algorithm.
- `Request`: floor number + direction (up/down). External requests come from hallway buttons, internal from elevator buttons.

**Scheduling strategies** (Strategy pattern):
- **SCAN (elevator algorithm)**: Move in one direction, serve all requests in that direction, then reverse. Like a disk head.
- **Shortest Seek First**: Go to the nearest requested floor. Optimizes latency but can starve distant floors.
- **Zone-based**: Assign elevators to floor ranges in a tall building.

**State pattern** works well here — each state (Idle, MovingUp, etc.) encapsulates its behavior and valid transitions. When Idle receives a request below current floor, it transitions to MovingDown.

---

### MEDIUM PRIORITY

---

Q8. What is the Decorator pattern, and how does it differ from inheritance for extending behavior?

A8.
The Decorator wraps an object to add behavior dynamically, without modifying the original class. Both the wrapper and the original implement the same interface, so they're interchangeable.

Example: `InputStream → BufferedInputStream → GZipInputStream`. Each layer adds behavior (buffering, compression) by wrapping the previous one. The base stream doesn't change.

Inheritance adds behavior at compile time — it's static. Decorator adds behavior at runtime — it's dynamic. Inheritance creates a class explosion: `BufferedGZipEncryptedInputStream`. Decorators compose: `Encrypted(GZip(Buffered(stream)))`.

The decorator is the right choice when you want to mix and match behaviors independently, rather than creating subclasses for every combination.

---

Q9. What is the Builder pattern, and when is it preferable to a constructor with many parameters?

A9.
The Builder pattern constructs complex objects step by step. Instead of a constructor with 10 parameters (half of them optional), you chain builder calls:

```
user = User.builder()
    .name("Aman")
    .email("aman@email.com")
    .role("admin")
    .build()
```

Advantages: readable (you see what each parameter means), you can set only what you need (without passing null for optional params), and `build()` can validate the complete object before creation.

Use it when: a class has more than 3-4 constructor parameters, many are optional, or the construction process has multiple valid configurations. Lombok's `@Builder` and Python's `dataclass` with defaults reduce the boilerplate.

---

Q10. What is the Adapter pattern? Give a scenario where you would use it.

A10.
The Adapter makes incompatible interfaces work together. It wraps one interface and exposes another that the client expects. It's a translator.

Real scenario: your application uses a `PaymentGateway` interface with a `processPayment(amount, currency)` method. You're integrating a third-party SDK that has `ThirdPartyPayment.charge(cents, country_code)`. You write an adapter that implements `PaymentGateway`, internally calls `ThirdPartyPayment`, and handles the unit/parameter conversion.

Another common use: legacy system integration. The old system speaks XML; the new system speaks JSON. An adapter sits between them and translates. Your new code doesn't know or care about the legacy format.

---

Q11. What is the Command pattern, and how does it enable undo functionality?

A11.
The Command pattern encapsulates a request as an object — with all the information needed to execute it. Instead of calling a method directly, you create a command object and pass it to an invoker.

Each command implements `execute()` and `undo()`. The invoker maintains a history stack of executed commands. Undo pops the last command and calls `undo()`. Redo pushes it back and calls `execute()`.

Example: `MoveFileCommand` stores the file, source path, and destination path. `execute()` moves the file. `undo()` moves it back. The text editor's undo/redo, database transaction rollback, and macro recording all use this pattern.

It also decouples the object that invokes the operation from the object that knows how to perform it — useful for queuing operations, logging them, or executing them remotely.

---

Q12. How would you design a library management system? What entities and methods are needed?

A12.
**Entities**:
- `Book`: ISBN, title, author, copies. `BookItem` represents a physical copy with barcode, status (available/checked-out/reserved/lost).
- `Member`: ID, name, account status, currently borrowed items, fine balance.
- `Librarian`: Extends Member (or is a role), can add/remove books.
- `Loan`: BookItem, Member, issue date, due date, return date. Tracks an active checkout.
- `Fine`: Calculated from overdue days, associated with a Loan and Member.
- `Reservation`: Allows members to reserve a book when all copies are checked out.

**Key methods**:
- `checkout(member, bookItem)`: Validates member eligibility, marks item as checked-out, creates Loan.
- `return(bookItem)`: Marks item available, calculates fine if overdue, notifies first reservation.
- `search(title/author/ISBN)`: Uses Strategy pattern for different search criteria.
- `reserve(member, book)`: Queues a reservation. Notifies member when available (Observer).

---

Q13. How would you design a ride-sharing fare calculator that supports different pricing strategies?

A13.
This is a classic Strategy pattern use case.

**Interface**: `FareStrategy` with `calculateFare(distance, duration, demandMultiplier)`.

**Strategies**: `StandardFare`, `PremiumFare`, `SharedRideFare`, `SubscriptionFare`. Each implements the calculation differently — shared ride divides by estimated passengers, premium adds a comfort multiplier, subscription applies pre-paid credits.

**Surge pricing**: Implement as a Decorator or a separate multiplier injected into the strategy. `SurgePricingDecorator(StandardFare())` wraps the base calculation and applies a demand-based multiplier.

**Context**: `RideService` takes a `FareStrategy` (determined by ride type the user selects) and calls `calculateFare()` with ride data.

This way, adding a new ride type (e.g., luxury, eco) is just a new strategy class. No changes to `RideService`.

---

Q14. What is the Template Method pattern, and when does it enforce a skeleton algorithm with customizable steps?

A14.
The Template Method defines the skeleton of an algorithm in a base class, with hooks (abstract methods) that subclasses override to customize specific steps.

Example: `DataProcessor` has a fixed workflow: `readData() → validateData() → transformData() → writeData()`. The template method `process()` calls these in order. `readData()` and `writeData()` are abstract — `CSVProcessor` reads CSV, `JSONProcessor` reads JSON. But the overall flow is the same.

Use it when: multiple classes follow the same algorithm structure but differ in specific steps. It avoids code duplication of the shared structure while allowing variation in the details.

The difference from Strategy: Template Method uses inheritance (subclass overrides steps), Strategy uses composition (swap algorithm objects). Template Method controls the overall flow; Strategy replaces the entire algorithm.

---

### LOW PRIORITY

---

Q15. What is the Flyweight pattern, and where does it save memory?

A15.
The Flyweight pattern shares objects to reduce memory usage. Instead of creating a million identical objects, you create one and share it across all references.

Classic example: text rendering. Each character in a document doesn't store its own font, size, and style. Instead, a flyweight object stores the shared formatting, and only the character value and position are stored per character. For a million characters using 5 font styles, you have 5 flyweight objects instead of a million copies of font data.

Game development uses this heavily — thousands of trees in a game share the same mesh and texture (flyweight), differing only in position and rotation (extrinsic state).

---

Q16. What is the Chain of Responsibility pattern, and how is it used in middleware pipelines?

A16.
Chain of Responsibility passes a request through a chain of handlers until one handles it. Each handler decides: process it or pass it to the next handler.

Middleware pipelines are the textbook example. In Express.js or Django, each request passes through middleware: authentication → rate limiting → logging → request parsing → your handler. Each middleware can short-circuit (return 401 if not authenticated) or pass to the next.

It's also used in exception handling chains, event processing pipelines, and approval workflows (expense approval: manager → director → VP based on amount).

---

Q17. What is the Proxy pattern, and how does it differ from the Decorator pattern?

A17.
Both wrap an object and implement the same interface. The difference is intent:

**Proxy** controls access to the real object — lazy initialization (don't create the expensive object until needed), access control (check permissions before forwarding), caching proxy (return cached response without hitting the real service), remote proxy (represent a remote object locally).

**Decorator** adds new behavior — logging, encryption, buffering. It enhances functionality rather than controlling access.

In practice: a `DatabaseProxy` that checks if the user has permissions before forwarding queries is a proxy. A `LoggingDatabaseWrapper` that logs every query before forwarding is a decorator.

---

Q18. How would you design a vending machine using state-based transitions?

A18.
This is a State pattern application. The machine has distinct states, and behavior changes based on the current state.

**States**: `IdleState`, `HasMoneyState`, `DispenseState`, `OutOfStockState`.

**Transitions**:
- Idle → HasMoney: coin inserted.
- HasMoney → DispenseState: product selected and sufficient money.
- HasMoney → Idle: cancel pressed (return money).
- DispenseState → Idle: product dispensed, return change.
- Any → OutOfStock: last item dispensed.

Each state is a class implementing a `VendingMachineState` interface with methods: `insertCoin()`, `selectProduct()`, `dispense()`, `cancel()`. Invalid operations in a state (like `dispense()` in Idle) throw an appropriate error or do nothing.

The vending machine holds a reference to its current state and delegates all actions to it.

---

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q19. How would you design an online booking system (e.g., BookMyShow) with seat locking and concurrent booking handling?

A19.
**Core entities**: `Movie`, `Theater`, `Screen`, `Show` (movie + screen + time), `Seat`, `Booking`, `Payment`.

**The concurrency challenge**: 500 users try to book the same seat simultaneously. Only one should succeed.

**Seat locking approach**: When a user selects seats, temporarily lock them (5-10 minute TTL in Redis). `SET seat:show123:A5 user456 NX EX 300`. If the lock succeeds, the user has 5 minutes to complete payment. If they abandon, the lock expires and the seat is available again.

**Alternative — optimistic locking**: No explicit locking. On booking, attempt `UPDATE seats SET status = 'BOOKED', user_id = X WHERE seat_id = Y AND status = 'AVAILABLE'`. If affected rows = 0, someone else booked it — return "seat unavailable."

**Flow**: Browse shows → Select seats → Lock seats (temp hold) → Enter payment details → Process payment → Confirm booking → Unlock any extra held seats. If payment fails → release locks.

**Key design decisions**: Separate the seat inventory service from the payment service. Use a state machine for booking states: `INITIATED → SEATS_HELD → PAYMENT_PENDING → CONFIRMED / EXPIRED`. Run a background job to release expired holds.

---

Q20. How would you design a chess or tic-tac-toe game? What classes, interfaces, and game state management would you use?

A20.
**For Chess**:

**Core classes**:
- `Board`: 8×8 grid of cells. Manages piece positions.
- `Piece` (abstract): Subclasses `King`, `Queen`, `Rook`, `Bishop`, `Knight`, `Pawn`. Each implements `getValidMoves(board)` — returns legal positions.
- `Player`: Name, color (WHITE/BLACK), list of captured pieces.
- `Move`: Source position, destination, piece moved, piece captured (if any), special flags (castling, en passant, promotion).
- `Game`: Manages game state — current turn, move history, check/checkmate detection.

**State management**: `Game` holds current `GameStatus`: `ACTIVE`, `CHECK`, `CHECKMATE`, `STALEMATE`, `RESIGNED`, `DRAW`. After each move, validate the move is legal, update the board, check for check/checkmate, switch turns.

**Key design patterns**: Strategy pattern for different piece movement logic. Command pattern for moves (enables undo — pop from move history, reverse the move). Observer for notifying UI of board changes.

**Validation**: A move is valid if: the piece can reach that position, the path is unobstructed (except knights), the move doesn't leave own king in check.

---

Q21. How would you design a food delivery system (restaurants, menus, orders, delivery assignment)?

A21.
**Core entities**: `Restaurant` (name, location, menu, operating hours, status), `MenuItem` (name, price, category, availability), `Cart`, `Order` (items, restaurant, customer, delivery address, status, payment), `DeliveryAgent` (location, availability, current order), `Customer`.

**Key flows**:
- **Ordering**: Customer browses restaurants (filtered by location/cuisine) → adds items to cart → places order → payment processed → order sent to restaurant.
- **Restaurant side**: Order received → restaurant accepts/rejects → prepares food → marks "ready for pickup."
- **Delivery assignment**: When order is confirmed, find available delivery agents near the restaurant. Use a scoring function: distance to restaurant, current load, rating. Assign the best match. If rejected → re-assign.
- **Tracking**: Real-time location updates from the delivery agent (GPS every 5 seconds). Push to customer via WebSocket. ETA computation using routing APIs.

**Design patterns**: State pattern for order lifecycle (`PLACED → CONFIRMED → PREPARING → READY → PICKED_UP → DELIVERED → CANCELLED`). Observer for status change notifications. Strategy for delivery fee calculation (distance-based, surge, subscription discount).

---

### IMPORTANT

---

Q22. What is the State pattern? How does it differ from using if-else chains for state transitions?

A22.
The State pattern encapsulates state-specific behavior in separate classes. The context object delegates behavior to its current state object. When the state changes, the context swaps its state object.

**Without State pattern**: A `Document` with states (DRAFT, REVIEW, PUBLISHED) uses if-else everywhere: `if state == DRAFT: allow edit... elif state == REVIEW: allow approve... elif state == PUBLISHED: deny edit...`. Adding a new state means modifying every method with every if-else chain. Violates Open/Closed Principle.

**With State pattern**: Each state is a class — `DraftState`, `ReviewState`, `PublishedState`. Each implements `edit()`, `approve()`, `publish()`. The `Document` holds a reference to the current state and delegates: `currentState.edit()`. Adding a new state means adding one class — no existing code is modified.

**When to use**: Objects with clearly defined states where behavior depends on state. Traffic lights, document workflows, game character states (idle/running/jumping/attacking), order processing.

---

Q23. What is the Mediator pattern? How does it reduce coupling between components?

A23.
The Mediator centralizes communication between objects. Instead of objects referencing each other directly (N×N connections), they all communicate through a mediator (N connections to one mediator).

**Example**: An airport control tower is a mediator. Planes don't communicate with each other directly — they report to the tower, and the tower coordinates. Adding a new runway or plane doesn't require changing other planes.

**In software**: A chat room is a mediator — users send messages to the room, the room distributes them. A UI dialog where changing one dropdown affects several other fields — the dialog mediator coordinates updates.

**Benefits**: Decouples components (they only know about the mediator interface). Easier to add new components. Centralizes complex coordination logic. **Risk**: The mediator can become a god object if it takes on too much logic.

---

Q24. How would you design an ATM machine? What objects and state transitions are involved?

A24.
**Core classes**: `ATM` (card reader, cash dispenser, receipt printer, screen), `Card`, `Account`, `Transaction` (withdrawal, deposit, balance inquiry, transfer), `Bank` (validates cards and accounts via network).

**States (State pattern)**: `IdleState` → `CardInsertedState` → `PINEnteredState` → `TransactionSelectedState` → `ProcessingState` → `IdleState`.

**Flow**: Insert card → ATM reads card, contacts bank for validation → enter PIN → select transaction type → enter amount → ATM contacts bank for authorization → dispense cash (if withdrawal) → print receipt → eject card.

**Key considerations**: Network failure handling (what if the bank connection drops mid-transaction? → rollback, eject card). Cash management (track denomination inventory — can't dispense ₹500 note if only ₹100 notes remain). Daily withdrawal limits. Concurrent access (one user at a time per ATM, but the bank handles multiple ATMs concurrently — database-level locking for account balance updates).

---

Q25. What is the difference between composition, aggregation, and association in OOP?

A25.
**Association**: A general relationship — "uses" or "knows about." A `Teacher` has a reference to `Course`. Neither owns the other. Both can exist independently.

**Aggregation**: "Has-a" relationship with independent lifetimes. A `Department` has `Employees`. If the department is deleted, employees still exist — they can move to another department. The container doesn't own the contained objects.

**Composition**: Strong "has-a" — the contained objects can't exist without the container. A `House` has `Rooms`. If the house is destroyed, the rooms are destroyed. The container owns and manages the lifecycle of its parts.

**In code**: Aggregation — pass objects in via constructor or setter (they exist externally). Composition — create objects inside the constructor (they're created and destroyed with the parent).

**Interview relevance**: Shows you understand object relationships beyond basic inheritance. UML diagrams use different arrow styles: association (plain line), aggregation (open diamond), composition (filled diamond).

---

### GOOD-TO-HAVE

---

Q26. What is the Iterator pattern, and why is it useful?

A26.
The Iterator pattern provides a way to access elements of a collection sequentially without exposing the underlying representation. You don't need to know if it's an array, linked list, tree, or graph — you just call `next()` and `hasNext()`.

**Why it matters**: Decouples the traversal algorithm from the collection structure. You can have multiple iterators on the same collection simultaneously. You can define different traversal orders (in-order, pre-order, reverse) as different iterator implementations.

**Built into languages**: Python's `__iter__` and `__next__` — every `for` loop uses the iterator protocol. Java's `Iterator<T>` interface. JavaScript's Symbol.iterator. So natural that most developers don't realize they're using the pattern.

---

Q27. How would you design a logging framework with different log levels and output destinations?

A27.
**Core classes**: `Logger` (the main interface — `logger.info("msg")`, `logger.error("msg")`), `LogLevel` enum (DEBUG, INFO, WARN, ERROR, FATAL), `LogHandler`/`Appender` (abstract — where to send logs), `LogFormatter` (how to format the message).

**Handlers**: `ConsoleHandler` (print to stdout), `FileHandler` (write to file with rotation), `RemoteHandler` (send to Elasticsearch/Splunk). Multiple handlers can be attached to one logger.

**Design patterns used**: Chain of Responsibility (log message passes through handlers — each decides whether to handle it based on level). Strategy (different formatters — JSON, plain text, structured). Singleton (global logger instance, though this is debatable). Observer (handlers are notified of new log events).

**Log level filtering**: Each handler has a minimum level. FileHandler might log everything (DEBUG+), while ConsoleHandler only logs WARN+. The logger checks: for each handler, if `message.level >= handler.minLevel`, forward the message.

---

Q28. What is the Null Object pattern, and how does it reduce null checks?

A28.
Instead of returning `null` and forcing callers to check for it, return a "null object" — an object that implements the expected interface but does nothing.

**Example**: A logging interface with `ConsoleLogger` and `NullLogger`. `NullLogger.log()` does nothing. Code that optionally logs doesn't need `if logger != null: logger.log(...)`. It just calls `logger.log(...)` — if logging is disabled, the NullLogger silently absorbs the call.

**Benefits**: Eliminates null checks scattered throughout the codebase. Prevents NullPointerExceptions. Simplifies client code — no defensive programming needed.

**When NOT to use**: When `null` carries meaningful information — "this user has no address" is different from "this user has a do-nothing address." Use Null Object for optional behavior, not for representing missing data.