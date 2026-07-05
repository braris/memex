---
title: "JPA Methods for Faster Spring Boot Applications"
source: "https://medium.com/@kaurharjeet122/i-ignored-these-jpa-methods-for-years-now-spring-boot-application-10-faster-4e9ff820a964"
type: articles
ingested: 2026-07-05
tags: [spring-boot, jpa, hibernate, database-performance, batching, entitymanager, lazy-loading]
summary: "Article on Spring Boot and JPA performance pitfalls, including save in loops, saveAll batching, EntityManager flush and clear, avoiding findAll for API DTOs, @Modifying update queries, and LAZY fetching defaults."
fetched: 2026-07-05
---


# [I Ignored These JPA Methods for Years â€” Now Spring Boot Application 10Ã— Faster](https://medium.com/@kaurharjeet122/i-ignored-these-jpa-methods-for-years-now-spring-boot-application-10-faster-4e9ff820a964)

For years, I blamed **Spring Boot**.

> *â€œHibernate is slow.â€  
> â€œProduction databases are weird.â€  
> â€œWe need better servers.â€*

I was wrong.

Painfully wrong.

The real performance problem was hiding in plain sight â€”  
inside **JPA methods I had been using *incorrectly* for years**.

And fixing them didnâ€™t require:

- âŒ new infrastructure
- âŒ database tuning
- âŒ rewriting the app

Justâ€¦ **using JPA the way it was meant to be used**.

Hereâ€™s what changed everything.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Ffzbim4oApyohdxXO6xgyg.png)

## ðŸ’€ The Lie We All Believe About JPA

We assume:

> *â€œHibernate is smart.  
> It will optimize things automatically.â€*

Reality?

Hibernate is obedient.  
It does **exactly what you tell it to do** â€” even if that means destroying your performance.

Let me show you how I unknowingly did just that.

## 1ï¸âƒ£ save() in a Loop â€” The Performance Time Bomb

I wrote code like this for years:

```c
for (Order order : orders) {
    orderRepository.save(order);
}
```

Looks innocent.  
Works perfectly in dev.

Then traffic hits.

## What Hibernate actually does

- One SQL `INSERT` per iteration
- Multiple flushes
- Zero batching

Your database quietly cries in the corner.

## The one-line fix

```c
orderRepository.saveAll(orders);
```

Boom.

âœ” single persistence context  
âœ” batch support  
âœ” fewer round trips

ðŸ”¥ **Instant 3â€“4Ã— speedup** with *one method change*.

## 2ï¸âƒ£ flush() â€” The Method I Was Afraid to Touch

I avoided `flush()` like it was radioactive.

> *â€œHibernate flushes automaticallyâ€¦ right?â€*

Yes.  
At the **worst possible moment**.

## What happened in production

- Memory spikes
- Long GC pauses
- Random timeouts

## The fix that saved us

```c
entityManager.flush();
entityManager.clear();
```

Why this works:

- Forces SQL execution early
- Prevents persistence-context bloat
- Keeps memory under control

ðŸ”¥ **Batch jobs stopped crashing overnight.**

## 3ï¸âƒ£ clear() â€” The Most Underrated JPA Method Ever

Hereâ€™s a question:

â“ Why does a batch job slow down *the longer it runs*?

Answer:  
Hibernate is still tracking every single entity you ever touched.

## One forgotten line

```c
entityManager.clear();
```

Thatâ€™s it.

âœ” Detaches thousands of objects  
âœ” Frees JVM memory  
âœ” Stops Hibernate from babysitting old entities

ðŸ”¥ **Processing time dropped dramatically.**

## 4ï¸âƒ£ findAll() â€” The Method That Quietly Kills APIs

Confession time:  
I used `findAll()` in REST APIs.

A lot.

## The problem

```c
List<User> users = userRepository.findAll();
```

Hibernate:

- Loads everything
- Tracks everything
- Maps relationships you donâ€™t need

When all I wanted was:

- name
- email

## The smarter way

```c
@Query("SELECT new com.dto.UserDTO(u.name, u.email) FROM User u")
List<UserDTO> findUsers();
```

ðŸ”¥ **API response time:**  
**1200ms â†’ 150ms**

Same database.  
Same data.  
Different JPA usage.

## 5ï¸âƒ£ @Modifying Queries â€” Update Without Waking Hibernate

The classic mistake:

```c
User user = userRepository.findById(id).get();
user.setStatus("ACTIVE");
userRepository.save(user);
```

Hibernate:

- Loads entity
- Tracks changes
- Runs dirty checking

Allâ€¦ for a single update.

## The grown-up version

```c
@Modifying
@Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
int updateStatus(Long id, String status);
```

âœ” No entity loading  
âœ” No dirty checking  
âœ” Direct database update

ðŸ”¥ **10Ã— faster updates under load.**

## âš ï¸ Bonus Mistake: EAGER Fetching (Yes, This One Hurt)

```c
@OneToMany(fetch = FetchType.EAGER)
```

This single annotation can:

- explode query size
- create duplicate data
- wreck performance

ðŸ’¡ ==Default to== ==`LAZY`====  
Fetch== ==**only when you actually need it**====.==

## ðŸ§  The Moment Everything Clicked

JPA isnâ€™t slow.

**Lazy developers (past-me included) are.**

Once I stopped treating JPA like:

> *â€œMagic CRUD abstractionâ€*

and started treating it like:

> *â€œA powerful ORM that needs respectâ€*

My Spring Boot applications became:  
âœ” faster  
âœ” cheaper to run  
âœ” predictable under load

## ðŸš€ If You Remember One Thing

> ***Performance doesnâ€™t come from frameworks.  
> It comes from understanding them.***

If your Spring Boot app feels slow â€”  
start with **how youâ€™re using JPA**, not with blaming Hibernate.
