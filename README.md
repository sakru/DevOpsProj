# DevOpsProj

# Aegir — Submarine Control Simulator

## Projektidé

Aegir är en terminalbaserad simulator av ett distribuerat styrsystem för en ubåt. Ubåten består av flera oberoende systemmoduler, exempelvis sonar, navigation, ballast och framdrivning, som reagerar på kommandon och förändringar i omgivningen.

Användaren styr inte varje komponent direkt utan kan ge övergripande kommandon och skapa händelser i simuleringsmiljön, exempelvis förändrat djup, strömmar, sensorfel eller kommunikationsproblem. Systemet försöker därefter fortsätta navigera genom att låta de olika modulerna reagera utifrån sin egen information och sitt eget ansvar.

Projektets huvudfokus är objektorienterad programmering i Java, polymorfism, inkapsling, felhantering och separation of concerns.

## Superklass

Namn: `SystemModule`

Gemensamma fält:

- `String id`
- `String name`
- `ModuleStatus status`
- `double health`
- `boolean enabled`

Gemensamma metoder:

```java
public abstract void update(SimulationContext context);

public abstract String getStatusReport();

public void activate();

public void deactivate();

public boolean isOperational();
```

`SystemModule` representerar en generell teknisk modul i ubåtens styrsystem.

Metoderna `update()` och `getStatusReport()` implementeras olika beroende på modulens funktion och overridas därför i samtliga subklasser.

## Subklasser

### `SonarModule`

Ansvarar för information om omgivningen och avståndet till havsbotten.

Overridar:

```java
@Override
public void update(SimulationContext context)
```

för att läsa simulerad sensordata och skapa observationer.

```java
@Override
public String getStatusReport()
```

för att visa aktuell bottendistans, sensorkvalitet och modulstatus.

### `NavigationModule`

Ansvarar för önskat djup, kurs och rörelse utifrån tillgänglig information.

Overridar:

```java
@Override
public void update(SimulationContext context)
```

för att analysera aktuell position, djup, ström och sensordata.

```java
@Override
public String getStatusReport()
```

för att visa navigationsläge, måldjup och aktuell avvikelse.

### `BallastModule`

Ansvarar för ubåtens simulerade flytkraft.

Overridar:

```java
@Override
public void update(SimulationContext context)
```

för att justera ballastnivån mot önskat djup.

```java
@Override
public String getStatusReport()
```

för att visa ballastnivå och aktuell vertikal korrigering.

### `PropulsionModule`

Ansvarar för framdrivning och simulerad hastighet.

Overridar:

```java
@Override
public void update(SimulationContext context)
```

för att anpassa motoreffekt efter begärd hastighet och aktuell belastning.

```java
@Override
public String getStatusReport()
```

för att visa motoreffekt, önskad hastighet och faktisk hastighet.

### `ControlSurfaceModule`

Ansvarar för simulerade styr- och dykroder.

Overridar:

```java
@Override
public void update(SimulationContext context)
```

för att justera styrvinkel utifrån navigationssystemets begäran.

```java
@Override
public String getStatusReport()
```

för att visa aktuella styrvinklar och modulstatus.

## Polymorfism

Alla moduler lagras i samma collection:

```java
List<SystemModule> modules;
```

Programmet kan därför exempelvis köra:

```java
for (SystemModule module : modules) {
    module.update(context);
    System.out.println(module.getStatusReport());
}
```

Java väljer automatiskt korrekt implementation av `update()` och `getStatusReport()` beroende på objektets verkliga typ.

Detta används som en central del av simuleringsloopen och inte enbart som ett separat demonstrationsexempel.

## Interface

Namn: `Communicating`

Metoder:

```java
void receiveMessage(SystemMessage message);

void sendMessage(SystemMessage message);
```

Interfacet implementeras bland annat av:

`SonarModule`

`NavigationModule`

`PropulsionModule`

Olika moduler kan därför behandlas polymorft som kommunicerande komponenter utan att mottagaren behöver känna till deras konkreta klass.

Exempel:

```java
public void deliverMessage(
        Communicating receiver,
        SystemMessage message) {

    receiver.receiveMessage(message);
}
```

## Collections

`SubmarineControlSystem` innehåller:

```java
List<SystemModule> modules;
```

Listan används aktivt i systemet.

Programmet ska kunna söka efter en modul utifrån ett värde som användaren skriver in.

Exempel:

```java
findModuleByName(searchTerm);
```

Programmet ska även kunna aggregera verklig information från listan, exempelvis:

```text
Operational modules: 4 / 5
Failed modules: 1
Average health: 82 %
```

Collection används därför för sökning, filtrering och sammanställning och inte endast för lagring.

## Meny

`ConsoleMenu` ansvarar endast för input och output.

Affärslogiken ligger i `SubmarineControlSystem`, `SimulationEngine` och domänklasserna.

Exempel på meny:

```text
=====================================
       AEGIR CONTROL SIMULATOR
=====================================

1. Show system modules
2. Search or inspect module
3. Send submarine command
4. Inject system failure
5. Recover module
6. Advance simulation
7. Show system summary
8. Show simulation view
0. Exit

Select:
```

### Show system modules

Itererar den polymorfa listan och visar status för samtliga moduler.

### Search or inspect module

Användaren skriver exempelvis:

```text
navigation
```

och programmet söker dynamiskt i collection efter motsvarande modul.

### Send submarine command

Användaren kan ge övergripande simuleringskommandon, exempelvis:

```text
Set target depth
Set target speed
Maintain current depth
```

### Inject system failure

Användaren väljer en verklig modul från collection och kan simulera ett fel.

Exempel:

```text
Sonar -> OFFLINE
```

### Recover module

Försöker återställa en vald modul till fungerande tillstånd.

### Advance simulation

Kör ett eller flera simulation ticks.

Under varje tick uppdateras moduler polymorft via:

```java
for (SystemModule module : modules) {
    module.update(context);
}
```

### Show system summary

Aggregerar data från modulsamlingen och visar exempelvis antal fungerande system, felande system och genomsnittlig hälsa.

### Show simulation view

Visar ubåten, havsbotten och systemstatus i terminalen.

## Simulation

`SimulationEngine` ansvarar för världens tillstånd och simuleringsloopen.

Exempel på tillstånd:

```java
class SubmarineState {
    double depth;
    double speed;
    double pitch;
    double targetDepth;
    double targetSpeed;
}
```

och:

```java
class EnvironmentState {
    double seaFloorDepth;
    double currentStrength;
}
```

Fysikmodellen hålls medvetet enkel eftersom projektets huvudsyfte är Java och objektorienterad programmering, inte realistisk hydrodynamik.

## Terminalvisualisering

En extra klass, `TerminalRenderer`, kan presentera simuleringsläget grafiskt med ASCII/Unicode.

Exempel:

```text
SONAR ───► COMMAND ───► NAVIGATION
  ●                         ●
                            │
                   ┌────────┼─────────┐
                   ▼        ▼         ▼
                BALLAST    FINS     ENGINE
                  ●         ●         ●


                     __________
             _______/          \______
      ______/                         \____
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

                           ----->


___________________             ______________
                   \___________/
                     SEA FLOOR


Depth:        84 m
Target:       80 m
Speed:       7.2
Pitch:       -2.1°
Mode:        AUTOMATIC
```

Visualiseringen är separerad från affärslogiken och läser endast simulatorns aktuella state.

## Felscenarion

### Ogiltig användarinmatning

Användaren kan skriva:

```text
abc
```

istället för ett menyval.

Programmet använder exempelvis:

```java
try {
    int choice = Integer.parseInt(input);
} catch (NumberFormatException e) {
    System.out.println("Menu selection must be a number.");
}
```

Programmet fortsätter därefter att köra.

Tom input och menyval utanför tillåtet intervall valideras också.

### Ogiltigt simuleringsvärde

Försök att exempelvis sätta ett negativt måldjup eller en ogiltig hastighet ska inte accepteras.

Domänlogiken kan kasta:

```java
throw new IllegalArgumentException(
    "Target depth must be greater than zero."
);
```

`ConsoleMenu` fångar felet och visar ett informativt meddelande.

### Modul ej tillgänglig

Ett kommando kan skickas till en modul som befinner sig i status `OFFLINE`.

Detta hanteras genom en specifik validering och exempelvis ett eget exception:

```java
ModuleUnavailableException
```

istället för ett generellt:

```java
catch (Exception e)
```

## Ansvarsfördelning

`ConsoleMenu`

Ansvarar endast för användarinteraktion.

`SubmarineControlSystem`

Ansvarar för collection av moduler, sökning, systemkommandon och övergripande koordinering.

`SimulationEngine`

Ansvarar för simulation ticks och förändringar i världen.

`SystemModule`

Definierar gemensamt beteende för systemkomponenter.

Subklasserna

Ansvarar för respektive tekniskt delsystems regler.

`TerminalRenderer`

Ansvarar endast för presentation av systemets state.

Detta separerar user interface, business logic och domain model.

## Möjlig vidareutveckling

Grundversionen körs som ett komplett Java-program och uppfyller projektets kurskrav utan extern infrastruktur.

Arkitekturen kan senare utökas så att systemmodulerna körs som separata processer eller Docker-containers.

Exempel:

```text
sonar
navigation
ballast
propulsion
control-surface
command
simulation
```

Varje container kan då använda samma Java-domänmodell men kommunicera genom meddelanden istället för direkta metodanrop.

Detta är en vidareutveckling och inte ett krav för att grundversionen ska vara komplett.

## Motivering

Arv används eftersom systemmodulerna delar gemensamt state och beteende men reagerar olika under simulationen.

Polymorfism gör att `SimulationEngine` kan uppdatera alla komponenter genom typen `SystemModule` utan att känna till deras konkreta klasser.

Interfacet `Communicating` används separat från arv eftersom kommunikation är en förmåga som flera typer kan ha och inte beskriver vad objekten är.

User interface har separerats från domänlogiken så att simulationen senare kan användas med exempelvis ett annat terminalinterface, tester eller ett distribuerat Docker-baserat runtime utan att reglerna i domänklasserna behöver skrivas om.
