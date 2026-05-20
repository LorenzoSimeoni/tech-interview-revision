# 📘 Cours Java & Écosystème

## 1. 🧱 Programmation Orientée Objet (OOP)

## Définition

La programmation orientée objet est un paradigme où le programme est structuré autour d’objets, qui contiennent :

- des attributs (état)
- des méthodes (comportement)

Une classe est un modèle, un objet est une instance.

---

## 🔹 Encapsulation

On protège les données via des accès contrôlés.

```java
private String name;

public String getName() { return name; }
```

✔ Objectifs :

- contrôle des accès
- validation possible dans les setters
- éviter les modifications directes

👉 Exemple réel :

```java
public void setAge(int age) {
    if (age < 0) throw new IllegalArgumentException();
    this.age = age;
}
```

---

## 🔹 Héritage

Une classe peut hériter d’une autre classe.

- Java : 1 seule classe mère
- mais plusieurs interfaces

✔ permet :

- réutilisation de code
- spécialisation

⚠ risque :

- couplage fort
- hiérarchies rigides

---

## 🔹 Polymorphisme (corrigé + enrichi)

Le polymorphisme permet d’utiliser une référence générique (classe mère ou interface) pour manipuler des objets de types différents, tout en exécutant leur comportement spécifique au runtime.

---

### ✔ Runtime polymorphism (overriding)

```java
Animal a = new Dog();
a.speak();
```

👉 c’est le type réel (Dog) qui décide

---

### ✔ Compile-time polymorphism (overloading)

```java
print(int a)
print(String a)
```

👉 choix à la compilation

---

✔ IMPORTANT :

> Overloading FAIT bien partie du polymorphisme (statique)

---

## 🔹 Abstraction

On cache les détails d’implémentation.

```java
interface Payment {
    void pay();
}
```

✔ Objectif :

- définir un contrat
- masquer la logique interne

---

# 2. 📐 Cohesion & Coupling

## Cohesion

Une classe doit avoir une responsabilité claire et unique.

✔ Bonne cohésion :

- méthodes liées entre elles
- manipulation de la même donnée métier

---

## Coupling

Niveau de dépendance entre classes.

✔ Bon design :

- faible coupling
- dépendance à des interfaces

👉 Exemple :

```java
List<String> list = new ArrayList<>();
```

✔ (DIP) on dépend de `List`, pas de `ArrayList`

---

# 3. 🧠 SOLID

## S — Single Responsibility

Une classe = une responsabilité.

👉 Exemple mauvais :

- classe qui gère DB + logique métier + logs

---

## O — Open/Closed

Ouvert à extension, fermé à modification.

👉 Exemple :
Strategy pattern pour éviter les if/else

---

## L — Liskov Substitution

Une classe fille doit pouvoir remplacer la mère sans casser le comportement.

👉 Exemple classique :

```text
Bird → fly()
Penguin → ❌ ne peut pas fly()
```

✔ solution :

- revoir l’héritage
- introduire interfaces spécialisées

---

## I — Interface Segregation

Mieux vaut plusieurs petites interfaces que une grosse.

👉 mauvais :

```java
interface Worker {
    void work();
    void eat();
    void sleep();
}
```

✔ meilleur :

- Workable
- Eatable

---

## D — Dependency Inversion

Dépendre d’abstractions, pas d’implémentations.

👉 Exemple :

```java
List<String> list = new ArrayList<>();
```

✔ permet :

- swap facile d’implémentation
- testabilité

---

# 4. 🧩 Interfaces / Classes

## Classe abstraite

- méthodes concrètes + abstraites
- état possible

👉 utilisée quand :

- partage de code
- base commune forte

---

## Interface

✔ peut contenir :

- méthodes abstraites
- default methods
- static methods

👉 default methods :
✔ utiles pour évolution sans casser API
⚠ à éviter si logique métier complexe

---

## Sealed classes

Contrôle strict de l’héritage.

```java
sealed class Animal permits Dog, Cat {}
```

✔ utile pour :

- domain modeling
- switch exhaustif

---

## Inner classes (enrichi)

### Types :

- inner class (liée à instance externe)
- static nested class (indépendante)
- anonymous class

👉 inner class :

- accès à `this` externe
- crée référence implicite → risque memory leak

---

# 5. ☕ JVM / JDK / JRE

## JVM

Exécute bytecode.

## JRE

JVM + libs runtime.

## JDK

JRE + outils dev.

---

## Bytecode

Code intermédiaire portable.

---

# 6. ⚙️ JVM interne

## ClassLoaders

- Bootstrap
- Platform
- Application

👉 chargement dynamique des classes

---

## ⚠ ClassLoader leak (important)

Si un ClassLoader est référencé :
→ impossible à garbage collect
→ fuite mémoire en prod (très fréquent en webapps)

---

## Garbage Collector (corrigé)

✔ GC ne “réorganise pas la heap dynamiquement”

Il :

- libère objets non référencés
- gère générations

---

### Heap :

- Young Generation
- Old Generation (On va dans la old quand l'objet a survécu a plusieurs cycle. Une fois dans la old seul un full GC peut le clean, ça arrive moins souvent)

---

## JIT Compiler

✔ compile code chaud (le plus utilisé) en machine native

👉 amélioration progressive runtime

---

# 7. 🧵 Concurrence

## Problèmes :

- race conditions
- deadlocks
- visibility issues

---

## Solutions :

- synchronized
- volatile (visibility only)
- Atomic classes (CAS)
- concurrent collections
- locks
- ThreadLocal (très important)
- CompletableFuture
- Executors
- Virtual Threads

---

# 8. 🧠 Memory Java

## String pool

✔ String literals sont internés

```java
String a = "test";
String b = "test";
```

👉 même objet

---

## new String

👉 force création objet

---

## StringBuilder

✔ utilisé pour concat répétée

---

## try-with-resources

✔ fermeture automatique des ressources

---

# 9. 🌐 API REST

## REST

✔ stateless ✔ client-server ✔ cacheable ✔ uniform interface

---

## Idempotence

✔ même résultat après plusieurs appels

---

## HTTP methods

- GET
- POST
- PUT
- DELETE
- PATCH

---

## SOAP

- XML
- protocole strict
- souvent HTTP

---

# 10. 🗄 Collections

## List / Set / Map

✔ bon résumé

---

## HashSet (enrichi)

```text
hashCode() → bucket → equals()
```

✔ collision possible donc equals obligatoire

---

# 11. 🔄 Streams

✔ pipeline fonctionnel

- map
- filter
- reduce

⚠ lazy evaluation

---

# 12. 🧩 Annotations

✔ metadata + runtime processing

---

## Spring usage :

- DI
- AOP
- configuration

---

# 13. 🧾 Transactions SQL

✔ ACID toujours correct ou rollback de tout

---

# 14. 🧱 Design Patterns

## Création

- Singleton
- Factory
- Builder

---

## Comportement

- Strategy
- Observer ✔ (notification system)

👉 Observer exemple :

- event system
- Spring events

---

## Structure

- Adapter (legacy integration) convertir une interface en une autre attendue par le système.
- Decorator (IO streams) On empile des comportements de classes.
- Facade (simplification API)
- Proxy (Spring AOP ⚠ très important) On ajoute du code autour de ma classe sans toucher le code métier

---

# 15. 🚀 Spring Ecosystem

## Spring Framework

✔ IoC container + DI

---

## Spring Boot

✔ auto-configuration + embedded server + starters

---

## Internals (important)

- reflection
- proxies
- class scanning
- bean lifecycle

---

# 16. 🗄 Hibernate / JPA

## ORM

mapping objet ↔ table

---

## Entity states

- transient
- managed
- detached
- removed

---

## Concepts importants

- dirty checking
- lazy loading
- N+1 problem

---

# 17. 📦 Maven

✔ build tool + dependency management

---

## Lifecycle

validate → compile → test → package → install → deploy

---

## Scopes

- compile
- test
- runtime
- provided

---

# 18. ⚡ Performance Java

## Tools

- JMC
- VisualVM
- heap dumps

---

## JIT

✔ optimise runtime

---

## Règle senior

👉 profiler avant d’optimiser

# 📘 Cours complet : JMX (Java Management Extensions)

## 🧠 1. Introduction

### 🔹 Définition

**JMX (Java Management Extensions)** est une technologie Java permettant de :

- surveiller une application Java en cours d’exécution
- la contrôler à distance ou localement
- exposer des informations internes (métriques, état, etc.)
- exécuter des actions sur une application sans redémarrage

👉 En résumé :  
**JMX = système de monitoring et d’administration pour applications Java**

---

## 🏗️ 2. Architecture de JMX

JMX repose sur 3 éléments principaux :

### 1. MBean (Managed Bean)

Un objet Java exposé à JMX.

Il peut contenir :

- des attributs (données)
- des opérations (méthodes appelables)

---

### 2. MBean Server

C’est le registre central où sont enregistrés les MBeans.

👉 Il permet de :

- enregistrer des MBeans
- les retrouver
- les exposer à des outils externes

---

### 3. Connecteurs / outils

Ils permettent d’interagir avec JMX :

- JConsole
- JDK Mission Control
- applications distantes

---

## 🧩 3. Les MBeans

### 🔹 Définition

Un **MBean** est un objet Java “instrumenté” pour être contrôlable via JMX.

---

### 🔹 Types de MBeans (important)

| Type           | Description                                                   |
| -------------- | ------------------------------------------------------------- |
| Standard MBean | Le plus courant (interface + classe)                          |
| Dynamic MBean  | Décrit dynamiquement ses attributs                            |
| Open MBean     | Type générique (structures simples)                           |
| MXBean         | Version simplifiée pour JVM (utilisée par défaut pour la JVM) |

---

## 🧪 4. Exemple simple de MBean

### 4.1 Interface

```java
public interface AppStatsMBean {
    int getCounter();
    void reset();
}
```

---

### 4.2 Implémentation

```java
public class AppStats implements AppStatsMBean {

    private int counter = 0;

    public void increment() {
        counter++;
    }

    @Override
    public int getCounter() {
        return counter;
    }

    @Override
    public void reset() {
        counter = 0;
    }
}
```

---

### 4.3 Enregistrement dans JMX

```java
import java.lang.management.ManagementFactory;
import javax.management.*;

public class Main {
    public static void main(String[] args) throws Exception {

        MBeanServer server = ManagementFactory.getPlatformMBeanServer();

        AppStats stats = new AppStats();

        ObjectName name = new ObjectName("app:type=AppStats");

        server.registerMBean(stats, name);

        while (true) {
            stats.increment();
            Thread.sleep(1000);
        }
    }
}
```

---

## 🔍 5. Accès aux MBeans

Une fois enregistrés, les MBeans sont accessibles via :

- JConsole
- JDK Mission Control
- outils externes (remote JMX)

---

### 📊 Exemple d’utilisation

Avec JConsole ou Mission Control tu peux :

- voir `counter` évoluer en temps réel
- appeler `reset()` à distance
- surveiller l’application live

---

## ⚙️ 6. MBeans fournis par la JVM

La JVM expose déjà des MBeans automatiquement :

### Exemples :

- mémoire (heap / non-heap)
- garbage collector
- threads
- runtime JVM
- classes chargées

👉 C’est ce que tu vois dans JDK Mission Control

---

## 🔥 7. Cas d’usage réels

### 📊 Monitoring

- nombre de requêtes
- erreurs applicatives
- cache size

---

### 🧹 Administration live

- reset d’un cache
- purge de queues
- recalcul de données

---

### 🐞 Debug production

- inspection d’état interne
- activation de logs dynamiques

---

## 🧵 8. JMX et logs (cas important)

Grâce à JMX, tu peux changer le niveau de logs à chaud :

- INFO → DEBUG
- DEBUG → ERROR

👉 Sans redémarrage

Car :

- le logger est un objet en mémoire
- JMX modifie sa configuration runtime

---

## 🌐 9. Accès distant (Remote JMX)

JMX peut être exposé à distance via :

- RMI (Remote Method Invocation)
- ports dédiés

⚠️ Attention :

- nécessite sécurisation (auth + SSL)
- souvent désactivé en production pour raisons de sécurité

---

## ⚔️ 10. JMX vs technologies modernes

| Technologie          | Rôle                       |
| -------------------- | -------------------------- |
| JMX                  | monitoring bas niveau JVM  |
| Spring Actuator      | monitoring applicatif REST |
| Prometheus + Grafana | métriques + dashboards     |
| JDK Mission Control  | analyse JVM avancée        |

---

## 🧠 11. Spring et JMX

Spring permet d’exposer facilement des MBeans :

```java
@ManagedResource
public class CacheService {

    @ManagedAttribute
    public int getSize() {
        return 42;
    }

    @ManagedOperation
    public void clear() {
        System.out.println("Cache cleared");
    }
}
```

---

## 🧾 12. Résumé

### JMX permet de :

✔ surveiller une application Java
✔ exposer des objets (MBeans)
✔ modifier l’application à chaud
✔ accéder à des métriques JVM et métier

---

## 🎯 Phrase à retenir

👉 JMX est un système qui transforme une application Java en système **observable et contrôlable en temps réel**.

---

## 🚀 Bonus mental model

```
JVM
 ├── MBeans (tes objets + JVM)
 ├── MBean Server
 ├── JConsole / Mission Control
 └── Remote tools
```

---

## 👍 Conclusion

JMX est une technologie :

- puissante
- bas niveau
- très utile pour le monitoring JVM
- mais souvent remplacée par des outils modernes côté métier

---
