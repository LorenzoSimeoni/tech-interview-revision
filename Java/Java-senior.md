# 📘 Cours Java Senior Backend (JVM / Spring / Architecture / Perf)

# 1. 🧠 JVM Deep Dive (ce qu’un senior doit vraiment comprendre)

## 1.1 Memory Model (JMM)

Le Java Memory Model définit :

> comment les threads voient la mémoire partagée

### Problèmes couverts :

- reordering CPU
- visibility issues
- race conditions invisibles

---

## 1.2 Happens-Before

Un des concepts les plus importants en concurrence.

✔ garantit :

> si A happens-before B → B voit A

Ex :

- unlock → lock
- volatile write → read
- thread start / join

---

## 1.3 Volatile (version réelle)

✔ garantit :

- visibilité entre threads
  ❌ pas d’atomicité

---

## 1.4 CPU vs JVM optimisation

La JVM optimise via :

- JIT compilation
- escape analysis
- lock elimination
- scalar replacement

👉 un objet peut ne jamais exister en mémoire réelle

---

## 1.5 GC avancé

### G1 GC (standard)

- region-based heap
- pause time target

### ZGC / Shenandoah

- ultra low latency
- pauses < 10ms

---

## 1.6 Memory issues réels

- heap leaks (références persistantes)
- off-heap leaks (Netty, direct buffers)
- metaspace leaks (classloaders)

---

# 2. ⚙️ ClassLoaders & Runtime dynamique

## 2.1 Architecture réelle

- Bootstrap
- Platform
- Application

---

## 2.2 Problème senior : ClassLoader leak

Très courant en prod :

👉 reload application = memory leak

Causes :

- static references
- thread pools non stoppés
- caches globaux

---

## 2.3 Spring + ClassLoader

Spring Framework repose sur :

- classpath scanning
- reflection
- proxy generation
- bytecode enhancement

---

# 3. 🔥 Spring Internals (niveau senior)

## 3.1 IoC Container réel

Spring ne “crée pas des objets”.

Il gère :

- BeanDefinition
- BeanFactory
- ApplicationContext

---

## 3.2 Bean lifecycle complet

1. Instantiation
2. Dependency injection
3. Aware callbacks
4. BeanPostProcessor (before init)
5. @PostConstruct
6. Proxy wrapping (AOP)
7. Bean ready

---

## 3.3 Proxies (TRÈS important)

Deux types :

### JDK dynamic proxy

- interface-based

### CGLIB proxy

- subclass-based

👉 utilisé pour :

- @Transactional
- @Async
- @Cacheable

---

## 3.4 AOP réel

Spring AOP = proxy-based AOP

⚠ pas du vrai bytecode weaving complet

---

## 3.5 @Transactional (piège senior)

⚠ fonctionne uniquement si :

- appel via proxy
- pas self-invocation

---

## 3.6 Spring Boot internals

Spring Boot :

- auto-configuration via classpath
- conditional beans (@ConditionalOnClass)
- starters = dependency aggregation
- embedded server

---

# 4. 🗄 Hibernate / JPA (deep internals)

## 4.1 Entity lifecycle

- transient
- persistent
- detached
- removed

---

## 4.2 Persistence Context

👉 cœur de Hibernate

- 1st level cache
- identity guarantee
- dirty checking

---

## 4.3 Dirty checking

Hibernate :

> détecte automatiquement les modifications

👉 pas besoin de update SQL explicite

---

## 4.4 Lazy Loading (proxy magique)

- objets remplacés par proxies
- requête SQL déclenchée à la demande

⚠ piège :

- LazyInitializationException

---

## 4.5 N+1 problem

Problème classique :

- 1 requête parent
- N requêtes enfants

Solutions :

- fetch join
- entity graph
- batch size

---

## 4.6 1st vs 2nd level cache

- 1st level = session (obligatoire)
- 2nd level = shared cache (optional)

---

# 5. ⚡ Concurrency avancée

## 5.1 Thread model réel

- OS threads
- virtual threads (Project Loom)

---

## 5.2 Virtual Threads (Java moderne)

Java :

- lightweight threads
- scalable blocking I/O
- change le modèle server

---

## 5.3 Locking strategies

- synchronized (coarse)
- ReentrantLock (fine control)
- ReadWriteLock
- StampedLock (optimistic locking)

---

## 5.4 Lock-free programming

- CAS (Compare And Swap)
- Atomic classes
- concurrent collections

---

## 5.5 Thread safety patterns

- immutability (best)
- confinement
- stateless services

---

# 6. 🌐 Architecture backend (senior mindset)

## 6.1 Monolith vs Microservices

### Monolith

✔ simple
❌ scaling limité

### Microservices

✔ scalable
❌ complexité réseau + observabilité

---

## 6.2 Distributed systems problems

- network latency
- partial failure
- eventual consistency
- retries
- idempotency

---

## 6.3 Idempotency (critique en prod)

Ex :

- payment API
- retry safe endpoints

---

## 6.4 CAP theorem

Impossible d’avoir les 3 :

- Consistency
- Availability
- Partition tolerance

---

# 7. 🌐 API design (niveau senior)

## 7.1 REST maturity

- resource modeling
- proper HTTP semantics
- HATEOAS (rare mais important conceptuellement)

---

## 7.2 API pitfalls

- overfetching / underfetching
- chatty APIs
- lack of pagination
- missing idempotency keys

---

## 7.3 HTTP internals

- caching headers
- ETags
- compression
- connection pooling

---

# 8. 🧱 Design & Architecture

## 8.1 SOLID (niveau réel)

👉 souvent violé volontairement en prod

---

## 8.2 Design patterns senior

- Proxy (Spring core)
- Decorator (IO, Spring filters)
- Adapter (legacy integration)
- Strategy (business rules)
- CQRS (architecture)
- Event-driven architecture

---

## 8.3 Clean Architecture

Layers :

- domain
- application
- infrastructure
- adapters

👉 indépendance framework

---

## 8.4 Hexagonal architecture

Ports & Adapters :

- business core isolé
- infra interchangeable

---

# 9. 📦 Build & Dependency Management

Apache Maven

## 9.1 Dependency hell

- transitive conflicts
- version mismatch
- shading (fat jars)

---

## 9.2 Maven internals

- dependency resolution graph
- nearest wins strategy
- lifecycle plugins

---

# 10. 🚀 Performance engineering

## 10.1 Bottlenecks

- CPU
- memory
- I/O
- DB (most common)

---

## 10.2 Profiling tools

- Java Flight Recorder
- Java Mission Control
- async-profiler

---

## 10.3 GC tuning

- heap sizing
- pause time tuning
- allocation rate reduction

---

## 10.4 Database performance

- indexes
- query plans
- connection pool (HikariCP)

---

# 11. 📊 Observability (très senior)

## 11.1 Logging

- structured logs
- correlation IDs

---

## 11.2 Metrics

- latency
- throughput
- error rate

---

## 11.3 Tracing

- distributed tracing (OpenTelemetry)

---

# 12. 🔐 Security (souvent oublié)

- authentication vs authorization
- JWT pitfalls
- session vs stateless
- CSRF / CORS
- OWASP top 10

---

# 13. 🧩 Advanced Spring ecosystem

- Spring Security
- Spring Cloud
- Spring Batch
- Spring WebFlux (reactive)

---

# 🎯 Résumé mental senior

Un senior Java backend ne pense pas :

> “comment coder la feature”

mais :

- impact mémoire / GC
- comportement multi-thread
- scalabilité réseau
- coûts DB
- observabilité
- failure handling
- coupling architectural

---

# ⚡ Différence junior vs senior

| Junior          | Senior               |
| --------------- | -------------------- |
| “ça marche”     | “comment ça scale”   |
| code local      | système distribué    |
| objets Java     | JVM + mémoire        |
| Spring usage    | Spring internals     |
| SQL simple      | perf + locks + cache |
| design patterns | architecture globale |
