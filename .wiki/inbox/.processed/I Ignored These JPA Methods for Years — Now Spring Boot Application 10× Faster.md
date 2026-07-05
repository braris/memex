
# [I Ignored These JPA Methods for Years — Now Spring Boot Application 10× Faster](https://medium.com/@kaurharjeet122/i-ignored-these-jpa-methods-for-years-now-spring-boot-application-10-faster-4e9ff820a964)

For years, I blamed **Spring Boot**.

> *“Hibernate is slow.”  
> “Production databases are weird.”  
> “We need better servers.”*

I was wrong.

Painfully wrong.

The real performance problem was hiding in plain sight —  
inside **JPA methods I had been using *incorrectly* for years**.

And fixing them didn’t require:

- ❌ new infrastructure
- ❌ database tuning
- ❌ rewriting the app

Just… **using JPA the way it was meant to be used**.

Here’s what changed everything.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Ffzbim4oApyohdxXO6xgyg.png)

## 💀 The Lie We All Believe About JPA

We assume:

> *“Hibernate is smart.  
> It will optimize things automatically.”*

Reality?

Hibernate is obedient.  
It does **exactly what you tell it to do** — even if that means destroying your performance.

Let me show you how I unknowingly did just that.

## 1️⃣ save() in a Loop — The Performance Time Bomb

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

✔ single persistence context  
✔ batch support  
✔ fewer round trips

🔥 **Instant 3–4× speedup** with *one method change*.

## 2️⃣ flush() — The Method I Was Afraid to Touch

I avoided `flush()` like it was radioactive.

> *“Hibernate flushes automatically… right?”*

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

🔥 **Batch jobs stopped crashing overnight.**

## 3️⃣ clear() — The Most Underrated JPA Method Ever

Here’s a question:

❓ Why does a batch job slow down *the longer it runs*?

Answer:  
Hibernate is still tracking every single entity you ever touched.

## One forgotten line

```c
entityManager.clear();
```

That’s it.

✔ Detaches thousands of objects  
✔ Frees JVM memory  
✔ Stops Hibernate from babysitting old entities

🔥 **Processing time dropped dramatically.**

## 4️⃣ findAll() — The Method That Quietly Kills APIs

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
- Maps relationships you don’t need

When all I wanted was:

- name
- email

## The smarter way

```c
@Query("SELECT new com.dto.UserDTO(u.name, u.email) FROM User u")
List<UserDTO> findUsers();
```

🔥 **API response time:**  
**1200ms → 150ms**

Same database.  
Same data.  
Different JPA usage.

## 5️⃣ @Modifying Queries — Update Without Waking Hibernate

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

All… for a single update.

## The grown-up version

```c
@Modifying
@Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
int updateStatus(Long id, String status);
```

✔ No entity loading  
✔ No dirty checking  
✔ Direct database update

🔥 **10× faster updates under load.**

## ⚠️ Bonus Mistake: EAGER Fetching (Yes, This One Hurt)

```c
@OneToMany(fetch = FetchType.EAGER)
```

This single annotation can:

- explode query size
- create duplicate data
- wreck performance

💡 ==Default to== ==`LAZY`====  
Fetch== ==**only when you actually need it**====.==

## 🧠 The Moment Everything Clicked

JPA isn’t slow.

**Lazy developers (past-me included) are.**

Once I stopped treating JPA like:

> *“Magic CRUD abstraction”*

and started treating it like:

> *“A powerful ORM that needs respect”*

My Spring Boot applications became:  
✔ faster  
✔ cheaper to run  
✔ predictable under load

## 🚀 If You Remember One Thing

> ***Performance doesn’t come from frameworks.  
> It comes from understanding them.***

If your Spring Boot app feels slow —  
start with **how you’re using JPA**, not with blaming Hibernate.