To design the low-level structure for this parking lot system, we'll break down the problem into key components and define the class structure, relationships, and flow of operations.

### Key Entities:
1. **ParkingLot**: Represents the overall parking lot system, containing multiple floors and managing entry and exit operations.
2. **ParkingFloor**: Represents each floor within the parking lot, containing different types of parking spots.
3. **ParkingSpot**: Represents individual parking spots, categorized by type (Compact, Large, Handicapped, Motorcycle, Electric, etc.).
4. **Vehicle**: Represents different types of vehicles that can park in the lot (Car, Truck, Motorcycle, Electric Car, etc.).
5. **Ticket**: Issued to customers when they enter the parking lot and contains entry time, payment details, and the vehicle's spot information.
6. **Payment**: Represents payment processing, supporting both cash and card transactions.
7. **EntrancePanel/ExitPanel**: Manages vehicle entry and exit, ticket issuance, and payment processing.
8. **DisplayBoard**: Shows the availability of parking spots on each floor and the overall parking lot status.
9. **CustomerInfoPortal**: Allows customers to pay the parking fee on each floor, negating the need for payment at the exit.
10. **ElectricPanel**: Supports electric vehicle charging and payment.

### Class Diagram (Overview)

1. **ParkingLot**
   - `List<ParkingFloor> floors`
   - `List<EntrancePanel> entrancePanels`
   - `List<ExitPanel> exitPanels`
   - `int totalCapacity`
   - `int availableSpots`
   - `void addFloor(ParkingFloor floor)`
   - `void showParkingAvailability()`

2. **ParkingFloor**
   - `String floorNumber`
   - `List<ParkingSpot> parkingSpots`
   - `DisplayBoard displayBoard`
   - `void updateAvailability()`

3. **ParkingSpot**
   - `String spotId`
   - `ParkingSpotType type`
   - `boolean isAvailable`
   - `Vehicle currentVehicle`

4. **Vehicle (Abstract Class)**
   - `String licensePlate`
   - `VehicleType type`

   **Subclasses**: `Car`, `Truck`, `Motorcycle`, `ElectricCar`

5. **Ticket**
   - `String ticketId`
   - `Date entryTime`
   - `ParkingSpot parkingSpot`
   - `double totalFee`
   - `boolean isPaid`
   - `Date paymentTime`
   - `Payment paymentDetails`

6. **Payment**
   - `PaymentType type`
   - `double amount`
   - `boolean processPayment(double amount)`
   
   **Subclasses**: `CashPayment`, `CardPayment`

7. **EntrancePanel**
   - `String panelId`
   - `Ticket issueTicket(Vehicle vehicle)`

8. **ExitPanel**
   - `String panelId`
   - `boolean processExit(Ticket ticket, Payment payment)`

9. **CustomerInfoPortal**
   - `String portalId`
   - `boolean processPayment(Ticket ticket)`

10. **DisplayBoard**
    - `Map<ParkingSpotType, Integer> availableSpotsByType`
    - `void updateBoard()`

11. **ElectricPanel**
    - `String panelId`
    - `boolean chargeAndPay(Ticket ticket, double hours)`

### Detailed Class Breakdown

#### 1. ParkingLot
- **Responsibilities**: Manages multiple floors, keeps track of the overall capacity, provides entry and exit for vehicles, and ensures that the lot is not over capacity.
```java
class ParkingLot {
    List<ParkingFloor> floors;
    List<EntrancePanel> entrancePanels;
    List<ExitPanel> exitPanels;
    int totalCapacity;
    int availableSpots;

    public ParkingLot(List<ParkingFloor> floors, int totalCapacity) {
        this.floors = floors;
        this.totalCapacity = totalCapacity;
        this.availableSpots = totalCapacity;
    }

    public void showParkingAvailability() {
        for (ParkingFloor floor : floors) {
            floor.updateAvailability();
        }
    }

    public Ticket issueTicket(Vehicle vehicle) {
        // Logic to assign spot and generate a ticket
    }

    public boolean processExit(Ticket ticket, Payment payment) {
        // Logic for exit and payment processing
    }
}
```

#### 2. ParkingFloor
- **Responsibilities**: Manages multiple parking spots, updates availability, and displays information.
```java
class ParkingFloor {
    String floorNumber;
    List<ParkingSpot> parkingSpots;
    DisplayBoard displayBoard;

    public void updateAvailability() {
        displayBoard.updateBoard();
    }
}
```

#### 3. ParkingSpot
- **Responsibilities**: Represents a specific parking spot with its type and availability.
```java
class ParkingSpot {
    String spotId;
    ParkingSpotType type;
    boolean isAvailable;
    Vehicle currentVehicle;

    public void parkVehicle(Vehicle vehicle) {
        this.currentVehicle = vehicle;
        this.isAvailable = false;
    }

    public void removeVehicle() {
        this.currentVehicle = null;
        this.isAvailable = true;
    }
}
```

#### 4. Ticket
- **Responsibilities**: Tracks parking time, fees, and payment status.
```java
class Ticket {
    String ticketId;
    Date entryTime;
    ParkingSpot parkingSpot;
    double totalFee;
    boolean isPaid;
    Date paymentTime;
    Payment paymentDetails;

    public double calculateParkingFee(Date exitTime) {
        // Fee calculation logic based on parking duration
    }
}
```

#### 5. Payment
- **Responsibilities**: Processes payments using different methods.
```java
abstract class Payment {
    PaymentType type;
    double amount;

    public abstract boolean processPayment(double amount);
}

class CashPayment extends Payment {
    public boolean processPayment(double amount) {
        // Cash payment logic
    }
}

class CardPayment extends Payment {
    public boolean processPayment(double amount) {
        // Card payment logic
    }
}
```

#### 6. DisplayBoard
- **Responsibilities**: Displays the available parking spots for each spot type.
```java
class DisplayBoard {
    Map<ParkingSpotType, Integer> availableSpotsByType;

    public void updateBoard() {
        // Update available spots
    }
}
```

### Sequence of Operations
1. **Vehicle Entry**:
   - The customer arrives at the entrance.
   - The `EntrancePanel` generates a `Ticket`, assigns a `ParkingSpot`, and updates the availability.

2. **Vehicle Parking**:
   - The customer parks in the assigned spot.
   - The `ParkingSpot` availability is updated, and the `DisplayBoard` reflects the changes.

3. **Payment**:
   - The customer can pay at the `CustomerInfoPortal` on the floor or the `ExitPanel`.
   - Payment can be made via cash or card, and electric vehicles can also pay through the `ElectricPanel`.

4. **Vehicle Exit**:
   - The customer leaves the parking spot.
   - The `ExitPanel` verifies the payment and allows the vehicle to exit.
   - Availability is updated, and the spot is freed.

This design accommodates various requirements such as multiple entry/exit points, different parking spot types, payment flexibility, and parking lot capacity management.









Sure! I'll provide a full implementation of the parking lot system, incorporating the details you requested. This will include classes for the parking lot, vehicles, spots, tickets, payments, panels, and the per-hour parking fee model.

### ParkingLot System - Full Code Implementation

#### Enums and Constants

```java
// Enum for Vehicle types
enum VehicleType {
    CAR, TRUCK, MOTORCYCLE, VAN, ELECTRIC_CAR;
}

// Enum for ParkingSpot types
enum ParkingSpotType {
    COMPACT, LARGE, HANDICAPPED, MOTORCYCLE, ELECTRIC;
}

// Enum for Payment types
enum PaymentType {
    CASH, CREDIT_CARD;
}
```

#### Class: `Vehicle`

```java
abstract class Vehicle {
    String licensePlate;
    VehicleType type;

    public Vehicle(String licensePlate, VehicleType type) {
        this.licensePlate = licensePlate;
        this.type = type;
    }

    public VehicleType getType() {
        return type;
    }
}

class Car extends Vehicle {
    public Car(String licensePlate) {
        super(licensePlate, VehicleType.CAR);
    }
}

class Truck extends Vehicle {
    public Truck(String licensePlate) {
        super(licensePlate, VehicleType.TRUCK);
    }
}

class Motorcycle extends Vehicle {
    public Motorcycle(String licensePlate) {
        super(licensePlate, VehicleType.MOTORCYCLE);
    }
}

class ElectricCar extends Vehicle {
    public ElectricCar(String licensePlate) {
        super(licensePlate, VehicleType.ELECTRIC_CAR);
    }
}
```

#### Class: `ParkingSpot`

```java
class ParkingSpot {
    private String spotId;
    private ParkingSpotType type;
    private boolean isAvailable;
    private Vehicle currentVehicle;

    public ParkingSpot(String spotId, ParkingSpotType type) {
        this.spotId = spotId;
        this.type = type;
        this.isAvailable = true;
    }

    public boolean isAvailable() {
        return isAvailable;
    }

    public ParkingSpotType getType() {
        return type;
    }

    public void parkVehicle(Vehicle vehicle) {
        if (!isAvailable) {
            throw new IllegalStateException("Spot is already occupied.");
        }
        this.currentVehicle = vehicle;
        this.isAvailable = false;
    }

    public void removeVehicle() {
        this.currentVehicle = null;
        this.isAvailable = true;
    }

    public Vehicle getCurrentVehicle() {
        return currentVehicle;
    }
}
```

#### Class: `Ticket`

```java
import java.util.Date;

class Ticket {
    private String ticketId;
    private Date entryTime;
    private ParkingSpot parkingSpot;
    private double totalFee;
    private boolean isPaid;
    private Date paymentTime;
    private Payment paymentDetails;

    public Ticket(String ticketId, ParkingSpot parkingSpot) {
        this.ticketId = ticketId;
        this.parkingSpot = parkingSpot;
        this.entryTime = new Date(); // Entry time is the current time
    }

    public double calculateParkingFee(Date exitTime) {
        long diffInMilliseconds = exitTime.getTime() - entryTime.getTime();
        long hours = diffInMilliseconds / (1000 * 60 * 60);
        return calculateFeeBasedOnHours(hours);
    }

    private double calculateFeeBasedOnHours(long hours) {
        if (hours <= 1) {
            return 4.0;
        } else if (hours <= 3) {
            return 4.0 + (hours - 1) * 3.5;
        } else {
            return 4.0 + (2 * 3.5) + (hours - 3) * 2.5;
        }
    }

    public void pay(Payment payment) {
        this.paymentDetails = payment;
        this.isPaid = true;
        this.paymentTime = new Date();
    }

    public boolean isPaid() {
        return isPaid;
    }
}
```

#### Abstract Class: `Payment` and Subclasses

```java
abstract class Payment {
    protected PaymentType type;
    protected double amount;

    public Payment(PaymentType type, double amount) {
        this.type = type;
        this.amount = amount;
    }

    public abstract boolean processPayment();

    public double getAmount() {
        return amount;
    }
}

class CashPayment extends Payment {
    public CashPayment(double amount) {
        super(PaymentType.CASH, amount);
    }

    @Override
    public boolean processPayment() {
        System.out.println("Processing cash payment of $" + amount);
        return true;
    }
}

class CreditCardPayment extends Payment {
    public CreditCardPayment(double amount) {
        super(PaymentType.CREDIT_CARD, amount);
    }

    @Override
    public boolean processPayment() {
        System.out.println("Processing credit card payment of $" + amount);
        return true;
    }
}
```

#### Class: `ParkingFloor`

```java
import java.util.ArrayList;
import java.util.List;

class ParkingFloor {
    private String floorId;
    private List<ParkingSpot> spots;
    private DisplayBoard displayBoard;

    public ParkingFloor(String floorId, int numCompact, int numLarge, int numHandicapped, int numMotorcycle, int numElectric) {
        this.floorId = floorId;
        this.spots = new ArrayList<>();
        // Create spots based on type
        for (int i = 0; i < numCompact; i++) {
            spots.add(new ParkingSpot(floorId + "_C" + i, ParkingSpotType.COMPACT));
        }
        for (int i = 0; i < numLarge; i++) {
            spots.add(new ParkingSpot(floorId + "_L" + i, ParkingSpotType.LARGE));
        }
        for (int i = 0; i < numHandicapped; i++) {
            spots.add(new ParkingSpot(floorId + "_H" + i, ParkingSpotType.HANDICAPPED));
        }
        for (int i = 0; i < numMotorcycle; i++) {
            spots.add(new ParkingSpot(floorId + "_M" + i, ParkingSpotType.MOTORCYCLE));
        }
        for (int i = 0; i < numElectric; i++) {
            spots.add(new ParkingSpot(floorId + "_E" + i, ParkingSpotType.ELECTRIC));
        }
        this.displayBoard = new DisplayBoard();
    }

    public List<ParkingSpot> getAvailableSpotsByType(ParkingSpotType type) {
        List<ParkingSpot> availableSpots = new ArrayList<>();
        for (ParkingSpot spot : spots) {
            if (spot.isAvailable() && spot.getType() == type) {
                availableSpots.add(spot);
            }
        }
        return availableSpots;
    }

    public DisplayBoard getDisplayBoard() {
        return displayBoard;
    }

    public void updateDisplayBoard() {
        // Update the display board with available spots
        displayBoard.update(spots);
    }
}
```

#### Class: `DisplayBoard`

```java
import java.util.HashMap;
import java.util.List;
import java.util.Map;

class DisplayBoard {
    private Map<ParkingSpotType, Integer> availableSpots;

    public DisplayBoard() {
        this.availableSpots = new HashMap<>();
    }

    public void update(List<ParkingSpot> spots) {
        for (ParkingSpotType type : ParkingSpotType.values()) {
            int count = 0;
            for (ParkingSpot spot : spots) {
                if (spot.getType() == type && spot.isAvailable()) {
                    count++;
                }
            }
            availableSpots.put(type, count);
        }
        showAvailability();
    }

    public void showAvailability() {
        System.out.println("Available spots by type:");
        for (Map.Entry<ParkingSpotType, Integer> entry : availableSpots.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
    }
}
```

#### Class: `EntrancePanel`

```java
class EntrancePanel {
    private String panelId;

    public EntrancePanel(String panelId) {
        this.panelId = panelId;
    }

    public Ticket issueTicket(Vehicle vehicle, ParkingFloor floor) {
        List<ParkingSpot> availableSpots = floor.getAvailableSpotsByType(vehicle.getType().equals(VehicleType.ELECTRIC_CAR) ? ParkingSpotType.ELECTRIC : ParkingSpotType.COMPACT);
        if (availableSpots.isEmpty()) {
            System.out.println("Parking full for vehicle type: " + vehicle.getType());
            return null;
        }
        ParkingSpot assignedSpot = availableSpots.get(0);
        assignedSpot.parkVehicle(vehicle);
        System.out.println("Issued ticket for vehicle: " + vehicle.licensePlate + " at spot: " + assignedSpot.getType());
        return new Ticket(panelId + "_TICKET_" + vehicle.licensePlate, assignedSpot);
    }
}
```

#### Class: `ExitPanel`

```java
class ExitPanel {
    private String panelId;

    public ExitPanel(String panelId) {
        this.panelId = panelId;
    }

    public boolean processExit(Ticket ticket, Payment payment) {
        if (!ticket.isPaid()) {
            double fee = ticket.calculateParkingFee(new Date());
            payment = payment instanceof CashPayment ? new CashPayment(fee) : new CreditCardPayment(fee);
            payment.processPayment();
            ticket.pay(payment);
        }
        ticket.getParkingSpot().removeVehicle();
        System.out.println("Vehicle exited. Spot is now available.");
        return true;
    }
}
```

#### Class: `ParkingLot`

```java
import java.util.List;

class ParkingLot {
    private List<ParkingFloor

> floors;
    private List<EntrancePanel> entrancePanels;
    private List<ExitPanel> exitPanels;
    private int totalCapacity;
    private int availableSpots;

    public ParkingLot(List<ParkingFloor> floors, int totalCapacity) {
        this.floors = floors;
        this.totalCapacity = totalCapacity;
        this.availableSpots = totalCapacity;
    }

    public void showParkingAvailability() {
        for (ParkingFloor floor : floors) {
            floor.updateDisplayBoard();
        }
    }

    public boolean isParkingFull() {
        return availableSpots == 0;
    }
}
```

### Explanation:

- **ParkingLot** manages all floors and the entry/exit panels.
- **ParkingFloor** handles individual parking spots, keeps track of availability, and updates the display boards.
- **Ticket** manages entry time, fee calculation, and payment status.
- **Payment** supports multiple payment types.
- **EntrancePanel** and **ExitPanel** issue tickets, handle vehicle entry/exit, and process payments.

This design should meet your requirements for a parking system with multiple floors, vehicle types, and a dynamic fee model. Let me know if you need further adjustments or explanations!




###concurrency and exception handling

To create a high-standard parking lot system, we'll address **concurrency**, **exception handling**, and add **design improvements** to meet the expectations of interviewers. We'll use **synchronized blocks** and **locks** for thread safety, ensure **custom exception handling**, and optimize the code structure for maintainability and scalability.

### Key Enhancements:
1. **Concurrency Handling**:
   - Use Java's `ReentrantLock` to ensure proper synchronization when accessing shared resources like parking spots and payment systems.
   - Ensure that parking spot allocation and payment processing are atomic to avoid race conditions.

2. **Exception Handling**:
   - Create custom exceptions like `ParkingFullException`, `PaymentFailedException`, and handle them gracefully.
   - Validate inputs and handle edge cases like invalid vehicle types, insufficient balance, etc.

3. **Thread Safety**:
   - Ensure thread safety for critical operations like parking spot assignment and fee calculation by using `synchronized` methods or `locks`.

4. **Scalability**:
   - Separate the logic into services (like `ParkingService`, `PaymentService`) to maintain modularity and scalability.
   - Implement factory patterns for payment methods to handle multiple types dynamically.

### Final Code

#### Custom Exceptions

```java
class ParkingFullException extends Exception {
    public ParkingFullException(String message) {
        super(message);
    }
}

class PaymentFailedException extends Exception {
    public PaymentFailedException(String message) {
        super(message);
    }
}

class SpotNotAvailableException extends Exception {
    public SpotNotAvailableException(String message) {
        super(message);
    }
}
```

#### Class: `Vehicle`

```java
abstract class Vehicle {
    String licensePlate;
    VehicleType type;

    public Vehicle(String licensePlate, VehicleType type) {
        if (licensePlate == null || licensePlate.isEmpty()) {
            throw new IllegalArgumentException("License plate cannot be empty");
        }
        this.licensePlate = licensePlate;
        this.type = type;
    }

    public VehicleType getType() {
        return type;
    }

    public String getLicensePlate() {
        return licensePlate;
    }
}
```

#### Class: `ParkingSpot` (Thread-Safe)

```java
import java.util.concurrent.locks.ReentrantLock;

class ParkingSpot {
    private String spotId;
    private ParkingSpotType type;
    private boolean isAvailable;
    private Vehicle currentVehicle;
    private final ReentrantLock lock;

    public ParkingSpot(String spotId, ParkingSpotType type) {
        this.spotId = spotId;
        this.type = type;
        this.isAvailable = true;
        this.lock = new ReentrantLock();
    }

    public boolean isAvailable() {
        return isAvailable;
    }

    public ParkingSpotType getType() {
        return type;
    }

    public Vehicle getCurrentVehicle() {
        return currentVehicle;
    }

    public boolean parkVehicle(Vehicle vehicle) throws SpotNotAvailableException {
        lock.lock();
        try {
            if (!isAvailable) {
                throw new SpotNotAvailableException("Spot is already occupied.");
            }
            this.currentVehicle = vehicle;
            this.isAvailable = false;
            return true;
        } finally {
            lock.unlock();
        }
    }

    public boolean removeVehicle() {
        lock.lock();
        try {
            this.currentVehicle = null;
            this.isAvailable = true;
            return true;
        } finally {
            lock.unlock();
        }
    }
}
```

#### Class: `Ticket` (Concurrency and Exception Safe)

```java
import java.util.Date;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Ticket {
    private String ticketId;
    private Date entryTime;
    private ParkingSpot parkingSpot;
    private double totalFee;
    private boolean isPaid;
    private Date paymentTime;
    private Payment paymentDetails;
    private final Lock lock = new ReentrantLock();

    public Ticket(String ticketId, ParkingSpot parkingSpot) {
        this.ticketId = ticketId;
        this.parkingSpot = parkingSpot;
        this.entryTime = new Date(); // Entry time is the current time
    }

    public double calculateParkingFee(Date exitTime) {
        long diffInMilliseconds = exitTime.getTime() - entryTime.getTime();
        long hours = diffInMilliseconds / (1000 * 60 * 60);
        return calculateFeeBasedOnHours(hours);
    }

    private double calculateFeeBasedOnHours(long hours) {
        if (hours <= 1) {
            return 4.0;
        } else if (hours <= 3) {
            return 4.0 + (hours - 1) * 3.5;
        } else {
            return 4.0 + (2 * 3.5) + (hours - 3) * 2.5;
        }
    }

    public void pay(Payment payment) throws PaymentFailedException {
        lock.lock();
        try {
            if (this.isPaid) {
                throw new PaymentFailedException("Ticket is already paid");
            }
            if (!payment.processPayment()) {
                throw new PaymentFailedException("Payment processing failed");
            }
            this.paymentDetails = payment;
            this.isPaid = true;
            this.paymentTime = new Date();
        } finally {
            lock.unlock();
        }
    }

    public boolean isPaid() {
        return isPaid;
    }
}
```

#### Abstract Class: `Payment` and Subclasses (Factory for Dynamic Payment Types)

```java
abstract class Payment {
    protected PaymentType type;
    protected double amount;

    public Payment(PaymentType type, double amount) {
        this.type = type;
        this.amount = amount;
    }

    public abstract boolean processPayment();

    public double getAmount() {
        return amount;
    }
}

class CashPayment extends Payment {
    public CashPayment(double amount) {
        super(PaymentType.CASH, amount);
    }

    @Override
    public boolean processPayment() {
        System.out.println("Processing cash payment of $" + amount);
        return true;
    }
}

class CreditCardPayment extends Payment {
    public CreditCardPayment(double amount) {
        super(PaymentType.CREDIT_CARD, amount);
    }

    @Override
    public boolean processPayment() {
        System.out.println("Processing credit card payment of $" + amount);
        return true;
    }
}

class PaymentFactory {
    public static Payment getPayment(PaymentType type, double amount) {
        switch (type) {
            case CASH:
                return new CashPayment(amount);
            case CREDIT_CARD:
                return new CreditCardPayment(amount);
            default:
                throw new IllegalArgumentException("Invalid payment type");
        }
    }
}
```

#### Class: `ParkingFloor` (Thread-Safe)

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.locks.ReentrantLock;

class ParkingFloor {
    private String floorId;
    private List<ParkingSpot> spots;
    private DisplayBoard displayBoard;
    private final ReentrantLock floorLock;

    public ParkingFloor(String floorId, int numCompact, int numLarge, int numHandicapped, int numMotorcycle, int numElectric) {
        this.floorId = floorId;
        this.spots = new ArrayList<>();
        this.floorLock = new ReentrantLock();
        // Create spots based on type
        createSpots(numCompact, ParkingSpotType.COMPACT);
        createSpots(numLarge, ParkingSpotType.LARGE);
        createSpots(numHandicapped, ParkingSpotType.HANDICAPPED);
        createSpots(numMotorcycle, ParkingSpotType.MOTORCYCLE);
        createSpots(numElectric, ParkingSpotType.ELECTRIC);
        this.displayBoard = new DisplayBoard();
    }

    private void createSpots(int count, ParkingSpotType type) {
        for (int i = 0; i < count; i++) {
            spots.add(new ParkingSpot(floorId + "_" + type.name().substring(0, 1) + i, type));
        }
    }

    public List<ParkingSpot> getAvailableSpotsByType(ParkingSpotType type) {
        List<ParkingSpot> availableSpots = new ArrayList<>();
        floorLock.lock();
        try {
            for (ParkingSpot spot : spots) {
                if (spot.isAvailable() && spot.getType() == type) {
                    availableSpots.add(spot);
                }
            }
        } finally {
            floorLock.unlock();
        }
        return availableSpots;
    }

    public void updateDisplayBoard() {
        displayBoard.update(spots);
    }
}
```

#### Class: `ParkingLot` (Concurrency Safe, Full Capacity Handling)

```java
import java.util.List;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class ParkingLot {
    private List<ParkingFloor> floors;
    private List<EntrancePanel> entrancePanels;
    private List<ExitPanel> exitPanels;
    private int totalCapacity;
    private int availableSpots;
    private final Lock lock = new ReentrantLock();

    public ParkingLot(List<ParkingFloor> floors, int totalCapacity) {
        this.floors = floors;
        this.totalCapacity = totalCapacity;
        this.availableSpots = totalCapacity;
    }

    public void showParkingAvailability() {
        for (ParkingFloor floor : floors) {
            floor.updateDisplayBoard();
        }
    }

    public Ticket issueTicket(Vehicle vehicle) throws ParkingFullException {
        lock.lock();
        try {
            if (availableSpots <= 0) {
                throw new ParkingFullException("Parking lot is full");
            }
            for (ParkingFloor floor : floors) {
                List<ParkingSpot> availableSpots = floor.getAvailableSpotsByType(ParkingSpotType.COMPACT); // Adjust based on vehicle type
                if (!availableSpots.isEmpty()) {
                    ParkingSpot spot = availableSpots.get(0);  // Select first available spot
                    spot.parkVehicle(vehicle);
                    this.availableSpots--;
                    return new Ticket("TICKET_" + vehicle.getLicensePlate(), spot);
                }
            }
        } finally {
            lock.unlock();
        }
        throw new ParkingFullException("No available spot for the vehicle.");
    }

    public boolean isParkingFull() {
        return availableSpots == 0;
    }
}
```

### Conclusion:

- **Concurrency**: Handled using `ReentrantLock` for thread-safe operations on shared resources.
- **Exception Handling**: Custom exceptions ensure graceful error handling during parking and payment processes.
- **Scalability**: The code is modular and flexible to handle additional features (e.g., more payment types, new vehicle types).
- **Maintainability**: Well-structured classes and services allow easy modifications and testing.

This ensures a complete, robust solution that addresses your interview requirements effectively.