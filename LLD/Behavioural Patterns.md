# Iterator
- separating the logic of how you move through a collection from the collection itself.
## Example - Iterating through a Playlist
```Java
class Playlist {
    private List<String> songs = new ArrayList<>();
    public void addSong(String song) {
        songs.add(song);
    }
    public List<String> getSongs() {
        return songs;
    }
}
class MusicPlayer {
    public void playAll(Playlist playlist) {
        for (String song : playlist.getSongs()) {
            System.out.println("Playing: " + song);
        }
    }
}
// This should not be possible, but it is
playlist.getSongs().add("Unauthorized Song");
playlist.getSongs().remove(0);
```
## Problems
- **Breaks Encapsulation**
- **Tightly couples client to implementation**
- **Limited Traversal Options**
- **Testing becomes difficult**
## Implementing Iterator
```Java
interface Iterator<T>{
	boolean hanNext();
	T next();
}
interface IterableCollection<T> {
    Iterator<T> createIterator();
}
class Playlist implements IterableCollection<String> {
    private final List<String> songs = new ArrayList<>();
    public void addSong(String song) {
        songs.add(song);
    }
    public String getSongAt(int index) {
        return songs.get(index);
    }
    public int getSize() {
        return songs.size();
    }
    @Override
    public Iterator<String> createIterator() {
        return new PlaylistIterator(this);
    }
}
class PlaylistIterator implements Iterator<String> {
    private final Playlist playlist;
    private int index = 0;
    public PlaylistIterator(Playlist playlist) {
        this.playlist = playlist;
    }
    @Override
    public boolean hasNext() {
        return index < playlist.getSize();
    }
    @Override
    public String next() {
        return playlist.getSongAt(index++);
    }
}
public class MusicPlayer {
    public static void main(String[] args) {
        Playlist playlist = new Playlist();
        playlist.addSong("Shape of You");
        playlist.addSong("Bohemian Rhapsody");
        playlist.addSong("Blinding Lights");
        Iterator<String> iterator = playlist.createIterator();
        System.out.println("Now Playing:");
        while (iterator.hasNext()) {
            System.out.println(" 🎵 " + iterator.next());
        }
    }
}
```
## Class Diagram
![[Screenshot 2026-01-05 at 11.29.44 PM.png|700]]
## Benefits
- **Encapsulation is preserved**
- **Implementation independence** - The client code works with the Iterator interface.
- **Single Responsibility Principle** - Playlist class focuses on managing songs. PlaylistIterator class focuses on traversal logic.
- **Multiple simultaneous traversals** - create iterator returns a new iterator everytime.
- **Foundation for extensions** - We can now easily add new types of iterators
# Observer
- If the state of one object changes, all the other objects are notified and updated.
- Defines a one to many relationship.
## Example - Broadcasting Fitness Data
- Naive approach would be to update each object individually. This would require each object to be initialized in the main object.
```Java
class FitnessDataNaive {
    private int steps;
    private int activeMinutes;
    private int calories;

    private LiveActivityDisplayNaive liveDisplay = new LiveActivityDisplayNaive();
    private ProgressLoggerNaive progressLogger = new ProgressLoggerNaive();
    private NotificationServiceNaive notificationService = new NotificationServiceNaive();
    
    public void newFitnessDataPushed(int newSteps, int newActiveMinutes, int newCalories) {
        this.steps = newSteps;
        this.activeMinutes = newActiveMinutes;
        this.calories = newCalories;
        System.out.println("\nFitnessDataNaive: New data received - Steps: " + steps +
            ", ActiveMins: " + activeMinutes + ", Calories: " + calories);
        liveDisplay.showStats(steps, activeMinutes, calories);
        progressLogger.logDataPoint(steps, activeMinutes, calories);
        notificationService.checkAndNotify(steps);
    }
    public void dailyReset() {
        if (notificationService != null) {
            notificationService.resetDailyNotifications();
        }
        System.out.println("FitnessDataNaive: Daily data reset.");
        newFitnessDataPushed(0, 0, 0); // Notify with reset state
    }
}

public class FitnessAppNaiveClient {
    public static void main(String[] args) {
        FitnessDataNaive fitnessData = new FitnessDataNaive(display, logger, notifier);
        fitnessData.newFitnessDataPushed(500, 5, 20);
        fitnessData.newFitnessDataPushed(9800, 85, 350);
        fitnessData.newFitnessDataPushed(10100, 90, 380); // Goal should be hit
        fitnessData.dailyReset();
    }
}
```
## Problems
- **Tight Coupling** - FitnessData object is tightly coupled with 3 observer classes. This means any change in observer will change this object as well.
- **Violates OCP** - You need to add any feature, you modify the FitnessData class.
- **Inflexible and Static Design**
- **Responsibility Bloat**
- **Scalability Bottlenecks**
## Implementing Observer Pattern
```Java
interface FitnessDataObserver{
	public void update(FitnessData data);
}
interface FitnessDataSubject {
	public void addObserver(FitnessDataObserver observer);
	public void removeObserver(FitnessDataObserver observer);
	public void notifyObservers();
}
class FitnessData implements FitnessDataSubject{
	private int steps;
	private int activeMinutes;
	private int calories;
	private final List<FitnessDataObserver> observers = new ArrayList<FitnessDataObserver>;
	@Override
	public void addObserver(FitnessDataObserver observer){
		observers.add(observer);
	}
	@Override
	public void removeObserver(FitnessDataObserver observer){
		observers.remove(observer);
	}
	@Override
	public void notifyObservers(){
		for (FitnessDataObserver observer: observers){
			observer.update(this);
		}
	}
    public void newFitnessDataPushed(int steps, int activeMinutes, int calories) {
        this.steps = steps;
        this.activeMinutes = activeMinutes;
        this.calories = calories;
        System.out.println("\nFitnessData: New data received – Steps: " + steps +
            ", Active Minutes: " + activeMinutes + ", Calories: " + calories);
        notifyObservers();
    }
    public void dailyReset() {
        this.steps = 0;
        this.activeMinutes = 0;
        this.calories = 0;

        System.out.println("\nFitnessData: Daily reset performed.");
        notifyObservers();
    }
    public int getSteps() { return steps; }
    public int getActiveMinutes() { return activeMinutes; }
    public int getCalories() { return calories; }
	
}
class LiveActivityDisplay implements FitnessDataObserver {
    @Override
    public void update(FitnessData data) {
        System.out.println("Live Display → Steps: " + data.getSteps() +
            " | Active Minutes: " + data.getActiveMinutes() +
            " | Calories: " + data.getCalories());
    }
}
class ProgressLogger implements FitnessDataObserver {
    @Override
    public void update(FitnessData data) {
        System.out.println("Logger → Saving to DB: Steps=" + data.getSteps() +
            ", ActiveMinutes=" + data.getActiveMinutes() +
            ", Calories=" + data.getCalories());
        // Simulated DB/file write...
    }
}
class GoalNotifier implements FitnessDataObserver {
    private final int stepGoal = 10000;
    private boolean goalReached = false;
    @Override
    public void update(FitnessData data) {
        if (data.getSteps() >= stepGoal && !goalReached) {
            System.out.println("Notifier → 🎉 Goal Reached! You've hit " + stepGoal + " steps!");
            goalReached = true;
        }
    }
    public void reset() {
        goalReached = false;
    }
}
public class FitnessAppObserverDemo {
    public static void main(String[] args) {
        FitnessData fitnessData = new FitnessData();
        LiveActivityDisplay display = new LiveActivityDisplay();
        ProgressLogger logger = new ProgressLogger();
        GoalNotifier notifier = new GoalNotifier();
        // Register observers
        fitnessData.registerObserver(display);
        fitnessData.registerObserver(logger);
        fitnessData.registerObserver(notifier);
        // Simulate updates
        fitnessData.newFitnessDataPushed(500, 5, 20);
        fitnessData.newFitnessDataPushed(9800, 85, 350);
        fitnessData.newFitnessDataPushed(10100, 90, 380); // Goal should trigger
        // Daily reset
        notifier.reset();
        fitnessData.dailyReset();
    }
}
```
## Push based vs Pull based Observers
#### Push based
- No casting, Observer is simple
- Subject must know **what data every observer needs**
- **Signature changes** if data grows -> update all observers.
```Java
public void update(int steps, int activeMinutes, int calories);
```
#### Pull based
- Subject is simpler, Easy to add new observer needs  
- Observer must cast (unless generics used)
- Slightly more coupling
```Java
public void update(Subject subject){
	FitnessData fd = (FitnessData) subject;
}
```
## Class Diagram
![[Screenshot 2026-01-06 at 5.16.51 PM.png|700]]
# Strategy
- lets you define classes with each encapsulated algorithms and make them interchangeable at runtime.
- Strategy may rely on inheritance if algos have most of the logic in common and only a part varies.
## Shipping Cost Calculation
```Java
class ShippingCostCalculatorNaive {
    public double calculateShippingCost(Order order, String strategyType) {
        double cost = 0.0;
        if ("FLAT_RATE".equalsIgnoreCase(strategyType)) {
            System.out.println("Calculating with Flat Rate strategy.");
            cost = 10.0;
        } else if ("WEIGHT_BASED".equalsIgnoreCase(strategyType)) {
            System.out.println("Calculating with Weight-Based strategy.");
            cost = order.getTotalWeight() * 2.5;

        } else if ("DISTANCE_BASED".equalsIgnoreCase(strategyType)) {
            System.out.println("Calculating with Distance-Based strategy.");
            if ("ZoneA".equals(order.getDestinationZone())) {
                cost = 5.0;
            } else if ("ZoneB".equals(order.getDestinationZone())) {
                cost = 12.0;
            } else {
                cost = 20.0; // fallback
            }
        } else if ("THIRD_PARTY_API".equalsIgnoreCase(strategyType)) {
            System.out.println("Calculating with Third-Party API strategy.");
            // Simulated external call
            cost = 7.5 + (order.getOrderValue() * 0.02);
        } else {
            throw new IllegalArgumentException("Unknown shipping strategy: " + strategyType);
        }
        System.out.println("Calculated Shipping Cost: $" + cost);
        return cost;
    }
}

public class ECommerceAppV1 {
    public static void main(String[] args) {
        ShippingCostCalculatorNaive calculator = new ShippingCostCalculatorNaive();
        Order order1 = new Order();
        System.out.println("--- Order 1 ---");
        calculator.calculateShippingCost(order1, "FLAT_RATE");
        calculator.calculateShippingCost(order1, "WEIGHT_BASED");
        calculator.calculateShippingCost(order1, "DISTANCE_BASED");
        calculator.calculateShippingCost(order1, "THIRD_PARTY_API");
    }
}
```
## Problems
- **Violates OCP** - Any new method would require change in Calculator class.
- **Bloated Logic** - Loads of if - else condition.
- **Low Cohesion** - It knows how to handle **every possible algorithm**, rather than focusing on **orchestrating the calculation**.
## Implementing Strategy Pattern
```Java
interface ShippingStrategy {
    double calculateCost(Order order);
}
class FlatRateShipping implements ShippingStrategy {
    private double rate;
    public FlatRateShipping(double rate) {
        this.rate = rate;
    }
    @Override
    public double calculateCost(Order order) {
        System.out.println("Calculating with Flat Rate strategy ($" + rate + ")");
        return rate;
    }
}
class WeightBasedShipping implements ShippingStrategy {
    private final double ratePerKg;
    public WeightBasedShipping(double ratePerKg) {
        this.ratePerKg = ratePerKg;
    }
    @Override
    public double calculateCost(Order order) {
        System.out.println("Calculating with Weight-Based strategy ($" + ratePerKg + "/kg)");
        return order.getTotalWeight() * ratePerKg;
    }
}
class DistanceBasedShipping implements ShippingStrategy {
    private double ratePerKm;
    public DistanceBasedShipping(double ratePerKm) {
        this.ratePerKm = ratePerKm;
    }
    @Override
    public double calculateCost(Order order) {
        System.out.println("Calculating with Distance-Based strategy for zone: " + order.getDestinationZone());
        return switch (order.getDestinationZone()) {
            case "ZoneA" -> ratePerKm * 5.0;
            case "ZoneB" -> ratePerKm * 7.0;
            default -> ratePerKm * 10.0;
        };
    }
}
class ThirdPartyApiShipping implements ShippingStrategy {
    private final double baseFee;
    private final double percentageFee;
    public ThirdPartyApiShipping(double baseFee, double percentageFee) {
        this.baseFee = baseFee;
        this.percentageFee = percentageFee;
    }
    @Override
    public double calculateCost(Order order) {
        System.out.println("Calculating with Third-Party API strategy.");
        // Simulate API call
        return baseFee + (order.getOrderValue() * percentageFee);
    }
}
class ShippingCostService {
    private ShippingStrategy strategy;
    public ShippingCostService(ShippingStrategy strategy) {
        this.strategy = strategy;
    }
    public void setStrategy(ShippingStrategy strategy) {
        System.out.println("ShippingCostService: Strategy changed to " + strategy.getClass().getSimpleName());
        this.strategy = strategy;
    }
    public double calculateShippingCost(Order order) {
        if (strategy == null) {
            throw new IllegalStateException("Shipping strategy not set.");
        }
        double cost = strategy.calculateCost(order);
        System.out.println("ShippingCostService: Final Calculated Shipping Cost: $" + cost +
                           " (using " + strategy.getClass().getSimpleName() + ")");
        return cost;
    }
}
public class ECommerceAppV2 {
    public static void main(String[] args) {
        Order order1 = new Order();

        ShippingStrategy flatRate = new FlatRateShipping(10.0);
        ShippingStrategy weightBased = new WeightBasedShipping(2.5);
        ShippingStrategy distanceBased = new DistanceBasedShipping(5.0);
        ShippingStrategy thirdParty = new ThirdPartyApiShipping(7.5, 0.02);

        ShippingCostService shippingService = new ShippingCostService(flatRate);

        System.out.println("--- Order 1: Using Flat Rate (initial) ---");
        shippingService.calculateShippingCost(order1);

        System.out.println("\n--- Order 1: Changing to Weight-Based ---");
        shippingService.setStrategy(weightBased);
        shippingService.calculateShippingCost(order1);

        System.out.println("\n--- Order 1: Changing to Distance-Based ---");
        shippingService.setStrategy(distanceBased);
        shippingService.calculateShippingCost(order1);

        System.out.println("\n--- Order 1: Changing to Third-Party API ---");
        shippingService.setStrategy(thirdParty);
        shippingService.calculateShippingCost(order1);
    }
}
```
## Class Diagram
![[Screenshot 2026-01-06 at 5.37.17 PM.png|700]]
## Benefits
- **Open/Closed Principle**
- **Single Responsibility** - Each new class is independent and has a single responsibility,
- **Testability**
- **Runtime flexibility**
- **Composition over inheritance**
# Command
- Used to decouple actions from their objects. The actions themselves become objects.
- Helps in queuing, delaying or logging method calls. 
- Helps in implementing Undo/Redo functionality.
## Example - Tightly Coupled Smart Controller
```Java
Class Light{
    public void on() {
        System.out.println("Light turned ON");
    }
    public void off() {
        System.out.println("Light turned OFF");
    }
}
class Thermostat {
    public void setTemperature(int temp) {
        System.out.println("Thermostat set to " + temp + "°C");
    }
}
class SmartHomeControllerV1 {
    private final Light light;
    private final Thermostat thermostat;

    public SmartHomeControllerV1(Light light, Thermostat thermostat) {
        this.light = light;
        this.thermostat = thermostat;
    }

    public void turnOnLight() {
        light.on();
    }

    public void turnOffLight() {
        light.off();
    }

    public void setThermostatTemperature(int temperature) {
        thermostat.setTemperature(temperature);
    }
}
```
## Problems
- **Tight Coupling** - any changes in the Light code will change the remote code.
- **Poor Scalability** - New device addition will cause a lot of changes.
- **No Undo/Redo Support**
- **No Scheduling/Queuing**
- **No Generic Logging**
## Implementing Command
```Java
interface Command{
	public void execute();
	public void undo();
}
Class Light{
    public void on() {
        System.out.println("Light turned ON");
    }
    public void off() {
        System.out.println("Light turned OFF");
    }
}
class Thermostat {
	private int currentTemperature = 20;
    public void setTemperature(int temp) {
        System.out.println("Thermostat set to " + temp + "°C");
    }
    public int getCurrentTemperature(){
	    return currentTemperature;
    }
}
class LightOnCommand implements Command{
	private Light light;
	public LightOnCommand(Light light){
		this.light = light;
	}
	@Override
	public void execute(){
		light.on();
	}
	@Override
	public void undo(){
		light.off();
	}
}
class LightOffCommand implements Command{
	private Light light;
	public LightOnCommand(Light light){
		this.light = light;
	}
	@Override
	public void execute(){
		light.off();
	}
	@Override
	public void undo(){
		light.on();
	}
}
class SetTemperatureCommand implements Command{
	private Thermostat thermostat;
	private final int newTemperature;
	private int previousTemperature;
	public SetTemperatureCommand(Thermostat thermostat, int temperature){
		this.thermostat = thermostat;
		previousTemperature = temperature;
	}
    @Override
    public void execute() {
        previousTemperature = thermostat.getCurrentTemperature();
        thermostat.setTemperature(newTemperature);
    }
    @Override
    public void undo() {
        thermostat.setTemperature(previousTemperature);
    }
}
class SmartButton{
	private Command command;
	private final Stack<Command> histroy = new Stack<>();
	public setCommand(Command command){
		this.command = command;
	}
	public void press(){
		if(command != null){
			command.execute();
			history.push(command);
		}
		else{
			System.out.println("No command found");
		}
	}
	public void undoLastCommand(){
		if(!history.isEmpty()){
			Command lastCommand = Stack.pop();
			lastCommand.undo();
		}
		else{
			System.out.println("No command to undo");
		}
	}
}
public class SmartHomeApp {
    public static void main(String[] args) {
        // Receivers
        Light light = new Light();
        Thermostat thermostat = new Thermostat();
        // Commands
        Command lightOn  = new LightOnCommand(light);
        Command lightOff = new LightOffCommand(light);
        Command setTemp22 = new SetTemperatureCommand(thermostat, 22);
        // Invoker
        SmartButton button = new SmartButton();
        // Simulate usage
        System.out.println("→ Pressing Light ON");
        button.setCommand(lightOn);
        button.press();
        System.out.println("→ Pressing Set Temp to 22°C");
        button.setCommand(setTemp22);
        button.press();
        System.out.println("→ Pressing Light OFF");
        button.setCommand(lightOff);
        button.press();
        // Undo sequence
        System.out.println("\n↶ Undo Last Action");
        button.undoLast();  // undo Light OFF
        System.out.println("↶ Undo Previous Action");
        button.undoLast();  // undo Set Temp
        System.out.println("↶ Undo Again");
        button.undoLast();  // undo Light ON
    }
}
```
## Class Diagram
![[Screenshot 2026-01-06 at 6.11.34 PM.png|700]]
## Benefits
- **Encapsulated Commands:** Each action is a reusable, undoable object
- **Decoupled UI/Logic:** Invoker doesn’t know how a command works
- **Undo Support:** Each command tracks and reverts its own effect
- **Extensibility:** Easily add `PlayMusicCommand`, `OpenGarageCommand`, etc.
- **History Tracking:** Command history enables undo/redo or logging
# State
- Changes behaviour as internal state changes as if object is switching classes at runtime.
- Object might want to behave differently based on what state it is in.
- State pattern decouples the state from the object and treats **each state as a separate class.**
- Each state class will implement all the actions the user can perform.
## Example - Vending Machine
- The vending machine can only be in **one state**, such as:
	- **IdleState** – Waiting for user input (nothing selected, no money inserted).
	- **ItemSelectedState** – An item has been selected, waiting for payment.
	- **HasMoneyState** – Money has been inserted, waiting to dispense the selected item.
	- **DispensingState** – The machine is actively dispensing the item.
```Java
class VendingMachine {
    private enum State {
        IDLE, ITEM_SELECTED, HAS_MONEY, DISPENSING
    }
    private State currentState = State.IDLE;
    private String selectedItem = "";
    private double insertedAmount = 0.0;
    public void selectItem(String itemCode) {
        switch (currentState) {
            case IDLE:
                selectedItem = itemCode;
                currentState = State.ITEM_SELECTED;
                break;
            case ITEM_SELECTED:
                System.out.println("Item already selected");
                break;
            case HAS_MONEY:
                System.out.println("Payment already received for item");
                break;
            case DISPENSING:
                System.out.println("Currently dispensing");
                break;
        }
    }
    public void insertCoin(double amount) {
        switch (currentState) {
            case IDLE:
                System.out.println("No item selected");
                break;
            case ITEM_SELECTED:
                insertedAmount = amount;
                System.out.println("Inserted $" + amount + " for item");
                currentState = State.HAS_MONEY;
                break;
            case HAS_MONEY:
                System.out.println("Money already inserted");
                break;
            case DISPENSING:
                System.out.println("Currently dispensing");
                break;
        }
    }
    public void dispenseItem() {
        switch (currentState) {
            case IDLE:
                System.out.println("No item selected");
                break;
            case ITEM_SELECTED:
                System.out.println("Please insert coin first");
                break;
            case HAS_MONEY:
                System.out.println("Dispensing item '" + selectedItem + "'");
                currentState = State.DISPENSING;
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Item dispensed successfully.");
                resetMachine();
                break;
            case DISPENSING:
                System.out.println("Already dispensing. Please wait.");
                break;
        }
    }
    private void resetMachine() {
        selectedItem = "";
        insertedAmount = 0.0;
        currentState = State.IDLE;
    }
}
```
## Problems
- **Cluttered Code** - all state related code is in the same class.
- **Hard to extend** - Adding another state would lead to changes in all the methods.
- **Violate SRP** - The class is responsible for the state transitions, business rules and state specific logic.
## Implementing State Pattern
```Java
interface MachineState{
	void seelectItem(VendingMachine context, String itemCode);
	void insertCoin(VendingMachine context, double amount);
	void dispenseItem(VendingMachine context);
}
class IdleState imlements MachineState{
    @Override
    public void selectItem(VendingMachine context, String itemCode) {
        System.out.println("Item selected: " + itemCode);
        context.setSelectedItem(itemCode);
        context.setState(new ItemSelectedState());
    }
    @Override
    public void insertCoin(VendingMachine context, double amount) {
        System.out.println("Please select an item before inserting coins.");
    }
    @Override
    public void dispenseItem(VendingMachine context) {
        System.out.println("No item selected. Nothing to dispense.");
    }
}
class ItemSelectedState implements MachineState {
    @Override
    public void selectItem(VendingMachine context, String itemCode) {
        System.out.println("Item already selected: " + context.getSelectedItem());
    }
    @Override
    public void insertCoin(VendingMachine context, double amount) {
        System.out.println("Inserted $" + amount + " for item: " + context.getSelectedItem());
        context.setInsertedAmount(amount);
        context.setState(new HasMoneyState());
    }
    @Override
    public void dispenseItem(VendingMachine context) {
        System.out.println("Insert coin before dispensing.");
    }
}
class HasMoneyState implements MachineState {
    @Override
    public void selectItem(VendingMachine context, String itemCode) {
        System.out.println("Cannot change item after inserting money.");
    }
    @Override
    public void insertCoin(VendingMachine context, double amount) {
        System.out.println("Money already inserted.");
    }
    @Override
    public void dispenseItem(VendingMachine context) {
        System.out.println("Dispensing item: " + context.getSelectedItem());
        context.setState(new DispensingState());
        try { Thread.sleep(1000); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println("Item dispensed successfully.");
        context.reset();
    }
}
class DispensingState implements MachineState {
    @Override
    public void selectItem(VendingMachine context, String itemCode) {
        System.out.println("Please wait, dispensing in progress.");
    }
    @Override
    public void insertCoin(VendingMachine context, double amount) {
        System.out.println("Please wait, dispensing in progress.");
    }
    @Override
    public void dispenseItem(VendingMachine context) {
        System.out.println("Already dispensing. Please wait.");
    }
}
class VendingMachine {
	private MachineState state;
	private int insertedAmount;
	private String selectedItem;
	
	public VendingMachine(){
		state = new IdleState();
	}
	public void setState(MachineState newState){
		this.state = newState;
	}
	public void setSelectedItem(String itemCode) {
        this.selectedItem = itemCode;
    }
    public void setInsertedAmount(double amount) {
        this.insertedAmount = amount;
    }
    public String getSelectedItem() {
        return selectedItem;
    }
    public void selectItem(String itemCode) {
        currentState.selectItem(this, itemCode);
    }
    public void insertCoin(double amount) {
        currentState.insertCoin(this, amount);
    }
    public void dispenseItem() {
        currentState.dispenseItem(this);
    }
    public void reset() {
        this.selectedItem = "";
        this.insertedAmount = 0.0;
        this.currentState = new IdleState();
    }
}
public class VendingMachineApp {
    public static void main(String[] args) {
        VendingMachine vm = new VendingMachine();

        vm.insertCoin(1.0); // Invalid in IdleState
        vm.selectItem("A1");
        vm.insertCoin(1.5);
        vm.dispenseItem();

        System.out.println("\n--- Second Transaction ---");
        vm.selectItem("B2");
        vm.insertCoin(2.0);
        vm.dispenseItem();
    }
}
```

## Benefits
- Avoid switch-case madness
- Add or remove states **without modifying the core class**
- Keep each state’s logic **isolated and reusable**
## Class Diagram
![[Screenshot 2026-01-06 at 11.25.49 PM.png|700]]
# Template Method
- When you have steps to follow in a flow but some parts of it vary across classes.
- Create abstract classes with methods for parts that vary and implement them in subclasses.
## Example - Exporting Reports
![[Screenshot 2026-01-06 at 11.33.33 PM.png|700]]
- writeHeader() and writeDataRows() vary.
```Java
class ReportData {
    public List<String> getHeaders() {
        return Arrays.asList("ID", "Name", "Value");
    }
    public List<Map<String, Object>> getRows() {
        return Arrays.asList(
            Map.of("ID", 1, "Name", "Item A", "Value", 100.0),
            Map.of("ID", 2, "Name", "Item B", "Value", 150.5),
            Map.of("ID", 3, "Name", "Item C", "Value", 75.25)
        );
    }
}
class CsvReportExporterNaive {
    public void export(ReportData data, String filePath) {
        System.out.println("CSV Exporter: Preparing data (common)...");
        // ... data preparation logic ...
        System.out.println("CSV Exporter: Opening file '" + filePath + ".csv' (common)...");
        // ... file opening logic ...
        System.out.println("CSV Exporter: Writing CSV header (specific)...");
        // String.join(",", data.getHeaders());
        // ... write header to file ...
        System.out.println("CSV Exporter: Writing CSV data rows (specific)...");
        // for (Map<String, Object> row : data.getRows()) { ... format and write row ... }
        System.out.println("CSV Exporter: Writing CSV footer (if any) (common)...");
        System.out.println("CSV Exporter: Closing file '" + filePath + ".csv' (common)...");
        // ... file closing logic ...
        System.out.println("CSV Report exported to " + filePath + ".csv");
    }
}
class PdfReportExporterNaive {
    public void export(ReportData data, String filePath) {
        System.out.println("PDF Exporter: Preparing data (common)...");
        // ... data preparation logic ...
        System.out.println("PDF Exporter: Opening file '" + filePath + ".pdf' (common)...");
        // ... PDF library specific file opening ...
        System.out.println("PDF Exporter: Writing PDF header (specific)...");
        // ... PDF library specific header writing ...
        System.out.println("PDF Exporter: Writing PDF data rows (specific)...");
        // ... PDF library specific data row writing ...
        System.out.println("PDF Exporter: Writing PDF footer (if any) (common)...");
        System.out.println("PDF Exporter: Closing file '" + filePath + ".pdf' (common)...");
        // ... PDF library specific file closing ...
        System.out.println("PDF Report exported to " + filePath + ".pdf");
    }
}
public class ReportAppNaive {
    public static void main(String[] args) {
        ReportData reportData = new ReportData();
        CsvReportExporterNaive csvExporter = new CsvReportExporterNaive();
        csvExporter.export(reportData, "sales_report");
        System.out.println();
        PdfReportExporterNaive pdfExporter = new PdfReportExporterNaive();
        pdfExporter.export(reportData, "financial_summary");
    }
}
```
## Problems
- **Code Duplication**
- **Maintenance Overhead** - If a common step changes it needs to change in multiple classes.
- **Inconsistent Behavior** - Since each exporter implements the entire workflow new class might: omit a step, change the order etc
- **Poor Extensibility**
## Implementing Template Method
```Java
abstract class AbstractReportExporter {
    public final void exportReport(ReportData data, String filePath) {
        prepareData(data);
        openFile(filePath);
        writeHeader(data);
        writeDataRows(data);
        writeFooter(data);
        closeFile(filePath);
        System.out.println("Export complete: " + filePath);
    }
    protected void prepareData(ReportData data) { // Hook method
        System.out.println("Preparing report data (common step)...");
    }
    protected void openFile(String filePath) { // Hook method
        System.out.println("Opening file '" + filePath);
    };
    protected abstract void writeHeader(ReportData data);
    protected abstract void writeDataRows(ReportData data);
    protected void writeFooter(ReportData data) { // Hook method
        System.out.println("Writing footer (default: no footer).");
    }
    protected void closeFile(String filePath) { // Hook method
        System.out.println("Closing file '" + filePath);
    };
}
class CsvReportExporter extends AbstractReportExporter {
    //prepareData() not overridden - default will be used
    //openFile() not overridden - default will be used
    @Override
    protected void writeHeader(ReportData data) {
        System.out.println("CSV: Writing header: " + String.join(",", data.getHeaders()));
    }
    @Override
    protected void writeDataRows(ReportData data) {
        System.out.println("CSV: Writing data rows...");
        for (Map<String, Object> row : data.getRows()) {
            System.out.println("CSV: " + row.values());
        }
    }
    // writeFooter() not overridden - default will be used
    // closeFile() not overridden - default will be used
}
class PdfReportExporter extends AbstractReportExporter {
    //prepareData() not overridden - default will be used
    //openFile() not overridden - default will be used
    @Override
    protected void writeHeader(ReportData data) {
        System.out.println("PDF: Writing header: " + String.join(",", data.getHeaders()));
    }
    @Override
    protected void writeDataRows(ReportData data) {
        System.out.println("PDF: Writing data rows...");
        for (Map<String, Object> row : data.getRows()) {
            System.out.println("PDF: " + row.values());
        }
    }
    // writeFooter() not overridden - default will be used
    // closeFile() not overridden - default will be used
}
public class ReportAppTemplateMethod {
    public static void main(String[] args) {
        ReportData data = new ReportData();
        AbstractReportExporter csvExporter = new CsvReportExporter();
        csvExporter.exportReport(data, "sales_report");
        System.out.println();
        AbstractReportExporter pdfExporter = new PdfReportExporter();
        pdfExporter.exportReport(data, "financial_summary");
    }
}
```
## Class Diagram
![[Screenshot 2026-01-06 at 11.37.39 PM.png|700]]
## Benefits
- **Eliminated code duplication** by extracting the shared export process into a base class.
- **Ensured consistency** across all exporters by enforcing the same algorithm structure.
- **Made the system extensible** — adding a new exporter (e.g., Excel) only requires creating a new subclass.
- **Improved maintainability** — common logic changes only in one place.
- **Reduced risk of errors** — the order of steps is controlled and protected by the abstract base class.
# Visitor<-*
- lets you **add new operations to existing object structures** without modifying their classes.
- The **Visitor Pattern** lets you **externalize operations** into separate visitor classes. 
- Each visitor implements behavior for every element type, while the elements simply accept the visitor.
## Example - Shapes
```Java
interface Shape {
    void draw();
    double calculateArea();
    String exportAsSvg();
    String toJson();
}
class Circle implements Shape {
    private double radius;
    public Circle(double radius) {
        this.radius = radius;
    }
    public void draw() {
        System.out.println("Drawing a circle");
    }
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
    public String exportAsSvg() {
        return "<circle r=\"" + radius + "\" />";
    }
    public String toJson() {
        return "{ \"type\": \"circle\", \"radius\": " + radius + " }";
    }
}
```
## Problems
- **Violates SRP**
- **Hard to Extend** - Adding any new method will modify each class.
- **You Don’t Always Control the Classes** - If you dont own the classes you cant add new behaviour to it.
## Implementing Visitor Pattern
```Java
interface Shape {
    void accept(ShapeVisitor visitor);
}
class Circle implements Shape {
    private final double radius;
    public Circle(double radius) {
        this.radius = radius;
    }
    public double getRadius() {
        return radius;
    }
    @Override
    public void accept(ShapeVisitor visitor) {
        visitor.visitCircle(this);
    }
}
class Rectangle implements Shape {
    private final double width;
    private final double height;
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    public double getWidth() {
        return width;
    }
    public double getHeight() {
        return height;
    }
    @Override
    public void accept(ShapeVisitor visitor) {
        visitor.visitRectangle(this);
    }
}
interface ShapeVisitor {
    void visitCircle(Circle circle);
    void visitRectangle(Rectangle rectangle);
}
class AreaCalculatorVisitor implements ShapeVisitor {
    @Override
    public void visitCircle(Circle circle) {
        double area = Math.PI * circle.getRadius() * circle.getRadius();
        System.out.println("Area of Circle: " + area);
    }
    @Override
    public void visitRectangle(Rectangle rectangle) {
        double area = rectangle.getWidth() * rectangle.getHeight();
        System.out.println("Area of Rectangle: " + area);
    }
}
class SvgExporterVisitor implements ShapeVisitor {
    @Override
    public void visitCircle(Circle circle) {
        System.out.println("<circle r=\"" + circle.getRadius() + "\" />");
    }
    @Override
    public void visitRectangle(Rectangle rectangle) {
        System.out.println("<rect width=\"" + rectangle.getWidth() + 
            "\" height=\"" + rectangle.getHeight() + "\" />");
    }
}
public class VisitorPatternDemo {
    public static void main(String[] args) {
        List<Shape> shapes = List.of(
            new Circle(5),
            new Rectangle(10, 4),
            new Circle(2.5)
        );
        System.out.println("=== Calculating Areas ===");
        ShapeVisitor areaCalculator = new AreaCalculatorVisitor();
        for (Shape shape : shapes) {
            shape.accept(areaCalculator);
        }
        System.out.println("\n=== Exporting to SVG ===");
        ShapeVisitor svgExporter = new SvgExporterVisitor();
        for (Shape shape : shapes) {
            shape.accept(svgExporter);
        }
    }
}
```
## Double Dispatch
- The following is double dispatch - 
```Java
Circle{
	public void accept(Visitor visitor){
		visitor.visitCircle(this);
	}
}
```
- Why not implement it in the following way?
```Java
class AreaVisitor implements ShapeVisitor{
	public void visit(Circle c){
	}
	public void visit(Rectangle c){
	}
}  

Shape s = new Circle();
AreaVisitor av = new AreaVisitor();
av.visit(s);
```
	- Because this is Method Overloading -> Resolved at compile time. So the complier will map the visit() to accept a parameter of type Shape. Hence, this will lead to errors.
## Benefits
- **Decoupled logic:** Shape classes are clean; logic lives in visitors
- **Open/Closed Principle:** Easily add new visitors (e.g., `JsonExporterVisitor`) without touching shapes
- **Double dispatch:** Eliminated need for `instanceof` or type-checking
- **Reusability & maintainability:** Each visitor focuses on one operation and is testable in isolation
## Class Diagram
![[Screenshot 2026-01-07 at 10.39.21 PM.png|700]]
# Interpretor

# Mediator <-*
- promotes **loose coupling** by preventing objects from referring to each other directly
- As more components are added each component ends up handling its own logic plus knowledge of how to coordinate with others.
- This leads to messed up code which is hard to understand.
- introducing a **central object that handles communication between components**.
## Example - UI components
```Java
class TextField {
    private String text = "";
    private Button loginButton;
    public void setLoginButton(Button button) {
        this.loginButton = button;
    }
    public void setText(String newText) {
        this.text = newText;
        System.out.println("TextField updated: " + text);
        if (loginButton != null) {
            loginButton.checkEnabled();
        }
    }
    public String getText() {
        return text;
    }
}
class Button {
    private TextField usernameField;
    private TextField passwordField;
    private Label statusLabel;
    public void setDependencies(TextField username, TextField password, Label status) {
        this.usernameField = username;
        this.passwordField = password;
        this.statusLabel = status;
    }
    public void checkEnabled() {
        boolean enable = !usernameField.getText().isEmpty() &&
            !passwordField.getText().isEmpty();
        System.out.println("Login Button is now " + (enable ? "ENABLED" : "DISABLED"));
    }
    public void click() {
        if (!usernameField.getText().isEmpty() && !passwordField.getText().isEmpty()) {
            System.out.println("Login successful!");
            statusLabel.setText("✅ Logged in!");
        } else {
            System.out.println("Login failed.");
            statusLabel.setText("❌ Please enter username and password.");
        }
    }
}
class Label {
    public void setText(String message) {
        System.out.println("Status: " + message);
    }
}
```
## Problems
- **Tight Coupling**
- **Lack of Reusability** - You can’t easily reuse these components elsewhere. They’re hard-wired to interact with specific peers, making them **context-dependent**.
- **Poor Maintainability** - violates the **Open/Closed Principle**
- **Hidden Logic Sprawled Across Components**
## Implementing Mediator
```Java
interface UIMediator{
	void componentChanged(UIComponent component);
}
abstract class UIComponent{
	private UIMediator mediator;
	void UIComponent(UIMediator mediator){
		this.mediator = mediator;
	}
	public void notifyMediator(){
		mediator.componentChanged(this);
	}
}
class TextField implements UIComponent{
	private String text="";
	public void TextField(Mediator mediator){
		super(mediator);
	}
	public void setText(String text){
		this.text = text;
		notifyMediator();
	}
	public void getText(){
		return this.text;
	}
}
class Button implements UIComponent{
	private boolean enabled = false;
	public Button(Mediator mediator){
		super(mediator);
	}
	public void click(){
		if (enabled) {
            System.out.println("Login Button clicked!");
            notifyMediator(); // Will trigger login attempt
        } else {
            System.out.println("Login Button is disabled.");
        }
	}
	public void setEnabled(boolean value){
		this.enabled = value;
	}
}
class Label extends UIComponent {
    private String text;
    public Label(UIMediator mediator) {
        super(mediator);
    }
    public void setText(String message) {
        this.text = message;
        System.out.println("Status: " + text);
    }
}
class FormMediator implements UIMediator{
	private TextField usernameField;
	private TextField passwordField;
	private Button submitButton;
	private Label statusLabel;
	public void setUsernameFiels(TextField usernameField){
		this.usernameField = usernameField;
	}
	public void setPasswordField(TextField passwordField) {
        this.passwordField = passwordField;
    }
    public void setLoginButton(Button loginButton) {
        this.loginButton = loginButton;
    }
    public void setStatusLabel(Label statusLabel) {
        this.statusLabel = statusLabel;
    }
	@Override
    public void componentChanged(UIComponent component) {
        if (component == usernameField || component == passwordField) {
            boolean enableButton = !usernameField.getText().isEmpty() && !passwordField.getText().isEmpty();
            loginButton.setEnabled(enableButton);
        } else if (component == loginButton) {
            String username = usernameField.getText();
            String password = passwordField.getText();

            if ("admin".equals(username) && "1234".equals(password)) {
                statusLabel.setText("✅ Login successful!");
            } else {
                statusLabel.setText("❌ Invalid credentials.");
            }
        }
    }
}
public class MediatorApp {
    public static void main(String[] args) {
        FormMediator mediator = new FormMediator();

        TextField usernameField = new TextField(mediator);
        TextField passwordField = new TextField(mediator);
        Button loginButton = new Button(mediator);
        Label statusLabel = new Label(mediator);

        mediator.setUsernameField(usernameField);
        mediator.setPasswordField(passwordField);
        mediator.setLoginButton(loginButton);
        mediator.setStatusLabel(statusLabel);

        // Simulate user interaction
        usernameField.setText("admin");
        passwordField.setText("1234");
        loginButton.click(); // Should succeed

        System.out.println("\n--- New Attempt with Wrong Password ---");
        passwordField.setText("wrong");
        loginButton.click(); // Should fail
    }
}
```
## Benefits
- **Loose coupling:** Components no longer know about each other
- **Separation of concerns:** Coordination logic lives in the mediator, not in the components
- **Ease of extension:** Add new components or behaviors without modifying existing ones
- **Reusability:** Components like `TextField`, `Button`, and `Label` can be reused in other contexts
## Class Diagram
![[Screenshot 2026-01-07 at 10.54.19 PM.png|700]]
# Memento <-*
- lets you **capture and store an object’s internal state** so it can be **restored later**, without violating encapsulation.
- implement **undo/redo** functionality.
- support **checkpointing or versioning** of an object’s state.
- separate the concerns of **state storage** from **state management logic**.
## Example - Implement Undo in Text Editor
```Java
class TextEditorNaive {
    private String content = "";
    public void type(String newText) {
        content += newText;
    }
    public void undo(String previousContent) {
        content = previousContent;
    }
    public String getContent() {
        return content;
    }
}
public class TextEditorUndoV1 {
    public static void main(String[] args) {
        TextEditorNaive editor = new TextEditorNaive();
        editor.type("Hello");
        String snapshot1 = editor.getContent(); // manual snapshot
        editor.type(" World");
        String snapshot2 = editor.getContent();
        System.out.println("Current Content: " + editor.getContent()); // Hello World
        // Undo 1 step
        editor.undo(snapshot1);
        System.out.println("After Undo: " + editor.getContent()); // Hello
    }
}
```
## Problems
- **Encapsulation is Broken**
- **Manual Snapshot Management**
- **Not Scalable**
## Implementing Memento
```Java
class TextEditorMemento {
    private final String state;
    public TextEditorMemento(String state) {
        this.state = state;
    }
    public String getState() {
        return state;
    }
}
class TextEditor {
    private String content = "";
    public void type(String newText) {
        content += newText;
        System.out.println("Typed: " + newText);
    }
    public String getContent() {
        return content;
    }
    public TextEditorMemento save() {
        System.out.println("Saving state: \"" + content + "\"");
        return new TextEditorMemento(content);
    }
    public void restore(TextEditorMemento memento) {
        content = memento.getState();
        System.out.println("Restored state to: \"" + content + "\"");
    }
}
class TextEditorUndoManager {
    private final Stack<TextEditorMemento> history = new Stack<>();
    public void save(TextEditor editor) {
        history.push(editor.save());
    }
    public void undo(TextEditor editor) {
        if (!history.isEmpty()) {
            editor.restore(history.pop());
        } else {
            System.out.println("Nothing to undo.");
        }
    }
}
public class TextEditorUndoV2 {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        TextEditorUndoManager undoManager = new TextEditorUndoManager();

        editor.type("Hello");
        undoManager.save(editor); // save state: Hello

        editor.type(" World");
        undoManager.save(editor); // save state: Hello World

        editor.type("!");
        System.out.println("Current Content: " + editor.getContent()); // Hello World!

        System.out.println("\n--- Undo 1 ---");
        undoManager.undo(editor); // Back to: Hello World

        System.out.println("\n--- Undo 2 ---");
        undoManager.undo(editor); // Back to: Hello

        System.out.println("\n--- Undo 3 ---");
        undoManager.undo(editor); // Nothing left to undo
    }
}
```
## Benefits
- **Encapsulation:** Editor’s internal state is never exposed directly to the client
- **Clean undo logic:** The client doesn’t need to manage or interpret state — it just saves and restores
- **Separation of concerns:** The `TextEditor` handles state, and the `TextEditorUndoManager` handles history
- **Scalability:** Easy to extend with redo support, multi-level undo, or persistent versioning
## Optimizing Memento for Large State (Interview-Grade Answer)

### Approach 1: Store Deltas (Incremental Mementos)
- Instead of storing the **entire state**, store **only what changed**.
* Track modified fields, Memento stores `(field → oldValue)` pairs, Undo = apply reverse patch(Store mementos in a stack)
**Pros**
* Huge memory savings
* Faster snapshot creation
**Cons**
* Undo logic becomes more complex
* Redo needs careful handling
* Order of operations matters
### Approach 2: Command + Memento Hybrid
Each action is a **Command** that:
* Knows how to `execute()`
* Knows how to `undo()`
The Memento stores **only what the command needs** to undo.
**Pros**
* Clean separation of responsibilities
* Scales well with complex workflows
* Natural fit for undo/redo stacks
**Cons**
* More classes
* Higher conceptual overhead
### Approach 3: Limit History / Checkpointing
* Keep only last **N** mementos
* Or keep **periodic full snapshots** + deltas in between
**Pros**
* Predictable memory usage
**Cons**
* Can’t undo arbitrarily far back
## Class Diagram
![[Screenshot 2026-01-07 at 10.59.21 PM.png|700]]
# Chain of Responsibility<-*
- lets you **pass requests along a chain of handlers**, allowing each handler to decide whether to process the request or pass it to the next handler in the chain.
- A request must be handled by **one of many possible handlers**, and the sender should not be tightly coupled to any specific one.
- **decouple request logic** from the code that processes it.
- **flexibly add, remove, or reorder handlers** without changing the client code.
## Example - Handling HTTP Requests
```Java
class Request {
    public String user;
    public String userRole;
    public int requestCount;
    public String payload;
    public Request(String user, String role, int requestCount, String payload) {
        this.user = user;
        this.userRole = role;
        this.requestCount = requestCount;
        this.payload = payload;
    }
}

```
![[Screenshot 2026-01-07 at 11.04.03 PM.png|700]]
```Java
class RequestHandler {
    public void handle(Request request) {
        if (!authenticate(request)) {
            System.out.println("Request Rejected: Authentication failed.");
            return;
        }
        if (!authorize(request)) {
            System.out.println("Request Rejected: Authorization failed.");
            return;
        }
        if (!rateLimit(request)) {
            System.out.println("Request Rejected: Rate limit exceeded.");
            return;
        }
        if (!validate(request)) {
            System.out.println("Request Rejected: Invalid payload.");
            return;
        }
        System.out.println("Request passed all checks. Executing business logic...");
        // Proceed to business logic
    }
    private boolean authenticate(Request req) {
        return req.user != null;
    }
    private boolean authorize(Request req) {
        return "ADMIN".equals(req.userRole);
    }
    private boolean rateLimit(Request req) {
        return req.requestCount < 100;
    }
    private boolean validate(Request req) {
        return req.payload != null && !req.payload.isEmpty();
    }
}
public class App {
    public static void main(String[] args) {
        Request req = new Request("john_doe", "ADMIN", 42, "{ 'data': 123 }");
        RequestHandler handler = new RequestHandler();
        handler.handle(req);
    }
}
```
## Problems
- **Violates OCP** - If you want to add a new step the entire class has to change.
- **Poor Separation of concern**
- **No Reusability**
- **Inflexible Configuration**
## Implementing Chain of Responsibility
```Java
interface RequestHandler {
    void setNext(RequestHandler next);
    void handle(Request request);
}
abstract class BaseHandler implements RequestHandler {
    protected RequestHandler next;
    @Override
    public void setNext(RequestHandler next) {
        this.next = next;
    }
    protected void forward(Request request) {
        if (next != null) {
            next.handle(request);
        }
    }
}
class AuthHandler extends BaseHandler {
    @Override
    public void handle(Request request) {
        if (request.user == null) {
            System.out.println("AuthHandler: ❌ User not authenticated.");
            return; // Stop the chain
        }
        System.out.println("AuthHandler: ✅ Authenticated.");
        forward(request);
    }
}
class AuthorizationHandler extends BaseHandler {
    @Override
    public void handle(Request request) {
        if (!"ADMIN".equals(request.userRole)) {
            System.out.println("AuthorizationHandler: ❌ Access denied.");
            return;
        }
        System.out.println("AuthorizationHandler: ✅ Authorized.");
        forward(request);
    }
}
class RateLimitHandler extends BaseHandler {
    @Override
    public void handle(Request request) {
        if (request.requestCount >= 100) {
            System.out.println("RateLimitHandler: ❌ Rate limit exceeded.");
            return;
        }
        System.out.println("RateLimitHandler: ✅ Within rate limit.");
        forward(request);
    }
}
class ValidationHandler extends BaseHandler {
    @Override
    public void handle(Request request) {
        if (request.payload == null || request.payload.trim().isEmpty()) {
            System.out.println("ValidationHandler: ❌ Invalid payload.");
            return;
        }
        System.out.println("ValidationHandler: ✅ Payload valid.");
        forward(request);
    }
}
class BusinessLogicHandler extends BaseHandler {
    @Override
    public void handle(Request request) {
        System.out.println("BusinessLogicHandler: 🚀 Processing request...");
        // Core application logic goes here
    }
}
public class RequestHandler {
    public static void main(String[] args) {
        // Create handlers
        RequestHandler auth = new AuthHandler();
        RequestHandler authorization = new AuthorizationHandler();
        RequestHandler rateLimit = new RateLimitHandler();
        RequestHandler validation = new ValidationHandler();
        RequestHandler businessLogic = new BusinessLogicHandler();

        // Build the chain
        auth.setNext(authorization);
        authorization.setNext(rateLimit);
        rateLimit.setNext(validation);
        validation.setNext(businessLogic);

        // Send a request through the chain
        Request request = new Request("john", "ADMIN", 10, "{ \"data\": \"valid\" }");
        auth.handle(request);

        System.out.println("\n--- Trying an invalid request ---");
        Request badRequest = new Request(null, "USER", 150, "");
        auth.handle(badRequest);
    }
}
```
## Class Diagram
![[Screenshot 2026-01-07 at 11.06.33 PM.png|700]]
## Benefits
- **Modularity:** Each handler is isolated and easy to test
- **Loose Coupling:** Handlers don’t need to know who comes next
- **Extensibility:** Easily insert, remove, or reorder handlers
- **Clean Client Code:** Only responsible for building the chain and sending the request
	- **Open/Closed Compliant:** You can add new functionality (e.g., LoggingHandler) without touching existing code