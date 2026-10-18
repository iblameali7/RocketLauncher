# Mission Control — Facade Pattern (Assignment #4)

**Course:** Software Design Patterns
**Pattern:** Facade (structural)
**Topic:** Rocket launch control. The client launches a mission with one call, while the facade coordinates four independent subsystems in the correct order.

## Problem

Launching a rocket requires fuel loading, course plotting, life-support activation and launch-pad preparation, in a strict order. Without a facade, every client must know all four classes and the order of calls. A wrong order (for example, ignition before arming) breaks the launch.

## Solution

`MissionControlFacade` exposes one simple method:

```java
missionControl.launchMission("Moon", 300, 3);
```

The client passes only plain data (destination, payload in kg, crew size) and knows nothing about the subsystem classes.

## Structure

```
Main (Client) ───► MissionControlFacade ───► FuelSystem
                                       ├───► NavigationSystem
                                       ├───► LifeSupport
                                       └───► LaunchPad
```

| Class | Role | State | Fails when |
|---|---|---|---|
| `FuelSystem` | Calculates and loads fuel | `loadedFuel` | Required fuel exceeds the tank limit |
| `NavigationSystem` | Plots the course | `courseReady` | Destination is unknown (known: Moon, Mars, ISS) |
| `LifeSupport` | Activates life support | `active` | Crew is larger than the maximum |
| `LaunchPad` | Arms and ignites | `armed` | `ignite()` is called before `arm()` |
| `MissionControlFacade` | Coordinates the order of calls | none | — |
| `Main` | Client, uses only the facade | — | — |

Order of calls inside the facade: fuel → navigation → life support → arm → ignite.

## Pattern rules

- **Client uses only the Facade.** `Main` never creates or calls subsystem classes.
- **No back-references.** No subsystem class holds a reference to the facade or to another subsystem class.
- **Subsystem is still usable directly.** All subsystem classes are public, so an advanced client can bypass the facade. This is a defining property of the pattern.

## How to run

1. Open the project in IntelliJ IDEA (JDK 17 or newer).
2. Run `Main.main()`.

Expected output:

```
Fuel: loaded 4050.0 kg
Navigation: course set to Moon
Life support: active for 3 crew
Launch pad: armed
Launch pad: ignition! Liftoff!
```

Invalid input stops the launch at the first failing step, for example a crew larger than the maximum or an unknown destination.

## Clean Code principles

### 1. Single Responsibility Principle
The facade only coordinates. The fuel formula and the "too heavy" check live in `FuelSystem`, not in `MissionControlFacade`. Each subsystem class has exactly one reason to change.

### 2. No magic numbers
Limits and coefficients are named constants: `MAX_CREW`, `MAX_FUEL_KG`, `FUEL_PER_KG_OF_PAYLOAD`, `BASE_FUEL_KG`. If a limit changes, it is changed in one place and the code explains what the number means.

### 3. Meaningful names
Names describe intent: `launchMission`, `loadFuelFor`, `plotCourse`, `activateFor`, `isActive`. The facade API can be understood without reading the subsystem.

### 4. Small, focused classes
Each subsystem class answers one question (fuel, course, life support, launch readiness) and owns its own state and rules. No class mixes responsibilities.

### 5. No leaking of subsystem details through the public interface
`launchMission` takes and returns only simple types. The facade has no methods like `getFuelSystem()`, so the client never gets a subsystem object and does not depend on the internal structure. The subsystem can change without touching the client.

### Additional: meaningful error handling
Subsystem classes throw exceptions with clear messages (`IllegalArgumentException` for bad input, `IllegalStateException` for a wrong object state, for example "Pad is not armed"), instead of silently returning a flag.

## Known limitation

If a step fails in the middle of a launch, earlier steps are not rolled back (for example, fuel stays loaded). A real system would add a rollback or a reset step to the facade.
