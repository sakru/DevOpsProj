# Aegir — Submarine Control Simulator

## Projektidé

Jag vill göra en enkel ubåtssimulator som körs i terminalen.

Tanken är att ubåten har flera olika system som sonar, navigation, ballast och motor. De olika systemen har sina egna uppgifter men behöver samtidigt samarbeta för att ubåten ska kunna hålla rätt djup, hastighet och riktning.

Användaren ska kunna ge ubåten kommandon och även skapa olika problem i simulationen, till exempel stark ström, förändrad havsbotten eller att ett system slutar fungera.

Målet är inte att göra en realistisk ubåtssimulator utan att använda projektet för att träna på Java, objektorientering och hur flera olika delar av ett program kan arbeta tillsammans.

## Superklass

**Namn:** `SystemModule`

Alla större system i ubåten ska ärva från samma basklass.

Gemensamma fält kan till exempel vara:

- `id`
- `name`
- `status`
- `health`
- `enabled`

Gemensamma metoder:

```java
public abstract void update(SimulationContext context);

public abstract String getStatusReport();

public void activate();

public void deactivate();

public boolean isOperational();
```

`SystemModule` ska innehålla sådant som är gemensamt för alla system, medan varje subklass själv bestämmer hur den reagerar under simulationen.

## Subklasser

Jag planerar just nu följande moduler.

### `SonarModule`

Sonaren läser information om omgivningen, framför allt avståndet till havsbotten.

Den kommer bland annat overrida:

```java
update()
getStatusReport()
```

`update()` används för att uppdatera sensordata och `getStatusReport()` visar aktuell sonarstatus.

### `NavigationModule`

Navigationen använder information om ubåtens position, djup och omgivning för att bestämma hur ubåten bör röra sig.

Den overridar också:

```java
update()
getStatusReport()
```

Navigationen ska till exempel kunna jämföra aktuellt djup med måldjupet.

### `BallastModule`

Ballastsystemet påverkar om ubåten ska stiga eller sjunka.

`update()` ska justera ballast beroende på vilket djup navigationen försöker nå.

`getStatusReport()` visar bland annat aktuell ballastnivå.

### `PropulsionModule`

Motorsystemet ansvarar för ubåtens hastighet.

Det ska kunna reagera på önskad hastighet och ändra motoreffekten.

### `ControlSurfaceModule`

Den här modulen representerar ubåtens styr- och dykroder.

Den används för att påverka bland annat pitch och riktning.

## Polymorfism

Alla systemmoduler ska sparas i samma lista:

```java
List<SystemModule> modules;
```

På så sätt kan simulationen uppdatera alla moduler på samma sätt:

```java
for (SystemModule module : modules) {
    module.update(context);
}
```

Även om alla objekt ligger i samma lista kommer Java att köra rätt version av `update()` beroende på vilken typ objektet egentligen är.

Samma sak ska användas för status:

```java
for (SystemModule module : modules) {
    System.out.println(module.getStatusReport());
}
```

## Interface

Jag vill också ha ett interface för de system som kan kommunicera med andra delar av ubåten.

Arbetsnamn:

`Communicating`

Till exempel:

```java
public interface Communicating {
    void receiveMessage(SystemMessage message);
    void sendMessage(SystemMessage message);
}
```

Det kan implementeras av till exempel:

- `SonarModule`
- `NavigationModule`
- `PropulsionModule`

Tanken är att olika typer av moduler ska kunna ta emot samma typ av meddelande utan att resten av programmet behöver känna till exakt vilken klass det är.

## Collections

Den viktigaste collectionen blir:

```java
List<SystemModule> modules;
```

Den ska inte bara användas för att lagra moduler.

Jag vill bland annat kunna:

- söka efter en modul efter namn
- visa alla fungerande moduler
- visa moduler som har problem
- räkna hur många moduler som fortfarande fungerar
- beräkna till exempel genomsnittlig health

Exempel på resultat:

```text
Operational modules: 4 / 5
Failed modules: 1
Average health: 82 %
```

## Meny

Programmet ska köras från terminalen.

Första versionen av menyn kan ungefär se ut så här:

```text
=============================
       AEGIR SIMULATOR
=============================

1. Show system modules
2. Inspect module
3. Send submarine command
4. Create system failure
5. Recover module
6. Advance simulation
7. Show system summary
8. Show simulation
0. Exit
```

### 1. Show system modules

Visar alla moduler och deras nuvarande status.

### 2. Inspect module

Användaren skriver namnet på en modul och programmet söker efter den i listan.

### 3. Send submarine command

Här ska användaren kunna ge enkla kommandon, exempelvis:

```text
Set target depth
Set target speed
Maintain depth
```

### 4. Create system failure

Användaren kan välja en modul och simulera att något går fel.

Exempel:

```text
Sonar -> OFFLINE
```

### 5. Recover module

Försöker starta eller återställa en modul som inte fungerar.

### 6. Advance simulation

Kör nästa steg i simulationen.

Varje steg uppdaterar alla systemmoduler.

### 7. Show system summary

Visar en sammanfattning av hur ubåten och systemen mår.

### 8. Show simulation

Visar ubåtens aktuella position och status i terminalen.

## Simulation

Jag tänker hålla själva fysiken ganska enkel eftersom huvudsyftet med projektet är Java och OOP.

Ubåten kan till exempel ha:

```java
class SubmarineState {
    double depth;
    double speed;
    double pitch;
    double targetDepth;
    double targetSpeed;
}
```

Omgivningen kan ha något liknande:

```java
class EnvironmentState {
    double seaFloorDepth;
    double currentStrength;
}
```

Simulationen behöver alltså inte följa verklig hydrodynamik. Det räcker att värdena påverkar varandra på ett logiskt sätt.

Till exempel kan högre motoreffekt öka hastigheten och förändrad ballast påverka djupet.

## Terminalgrafik

Om det fungerar bra vill jag också göra en enkel terminalvy med ASCII eller Unicode.

Ungefär:

```text
SONAR ----> COMMAND ----> NAVIGATION
                              |
                    +---------+---------+
                    |         |         |
                 BALLAST    FINS      ENGINE


                     __________
             _______/          \______
      ______/                         \____

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

                          ----->


__________________              ______________
                  \____________/
                    SEA FLOOR


Depth:   84 m
Target:  80 m
Speed:   7.2
Pitch:   -2.1
```

Tanken är att man senare även ska kunna se om kommunikationen mellan systemen fungerar eller om någon modul är offline.

## Felscenarion

### Felaktig input i menyn

Användaren kan till exempel skriva:

```text
abc
```

när programmet förväntar sig ett nummer.

Det ska hanteras med `try/catch`, till exempel genom att fånga `NumberFormatException`.

Programmet ska sedan fortsätta köra och be om ett nytt menyval istället för att krascha.

### Ogiltiga värden

Det ska inte gå att sätta helt orimliga värden.

Exempel:

```text
Target depth: -500
```

Domänklassen kan då kasta:

```java
IllegalArgumentException
```

och menyn visar ett begripligt felmeddelande.

### Modul som inte fungerar

Ett annat scenario är att programmet försöker använda en modul som är offline.

Det ska också hanteras på ett kontrollerat sätt istället för att programmet kraschar.

Jag funderar på att senare använda ett eget exception, till exempel:

```java
ModuleUnavailableException
```

## Struktur

Jag vill försöka hålla användargränssnitt och logik separerade.

`ConsoleMenu`

Tar hand om input och det som skrivs ut till användaren.

`SubmarineControlSystem`

Håller reda på modulerna och övergripande kommandon.

`SimulationEngine`

Uppdaterar själva simulationen.

`SystemModule`

Basklass för systemen.

`SonarModule`, `NavigationModule`, `BallastModule` osv.

Innehåller logiken för respektive system.

`TerminalRenderer`

Kan senare användas för att rita simulationen utan att själva simulationslogiken behöver känna till hur terminalen ser ut.

## Möjlig vidareutveckling

En idé jag vill testa senare är att köra de olika systemen som separata processer eller Docker-containers.

Till exempel:

```text
sonar
navigation
ballast
propulsion
control-surface
command
simulation
```

Då skulle systemen kunna kommunicera med meddelanden över nätverket istället för vanliga metodanrop.

Det är dock inte nödvändigt för den första versionen. Först vill jag få ett komplett Java-program att fungera och uppfylla kursens krav.

## Motivering

Jag valde arv för systemmodulerna eftersom de har flera gemensamma egenskaper, men samtidigt fungerar på olika sätt.

Polymorfism gör att simulationen kan behandla alla moduler som `SystemModule` och låta respektive klass bestämma vad som händer när `update()` körs.

Jag använder ett separat interface för kommunikation eftersom kommunikation är något en modul kan kunna göra och inte själva typen av modul.

Jag vill också hålla terminalmenyn separat från simulationslogiken. Det borde göra det enklare att ändra eller bygga vidare på programmet senare utan att behöva skriva om allt.
