# 🎬 *Yoore Once, Twice, Three Times a Lady*

### A Romantic Comedy compiled against the quality attribute: **SCALABILITY**

> *LOT Polish Airlines, Flight LO-12A. One passenger. One seat reassignment. Three retries with no idempotency key. Three Hannas. One love story that refuses to scale gracefully.*

---

## The one true bug (your logline)

Hanna's reservation gets rebooked from **12A** (window, near the front, civilized) to **17B** (middle seat, deep in the cabin, the seat of the damned). A trivial load-balancing move — the system just shuffled a request to a less-busy node.

Except the rebooking endpoint **isn't idempotent.** The retry logic fires three times, and with no dedup key, the booking service happily commits **all three**. The manifest now reads:

```

Three commits. Three Hannas. **Once, twice, three times a Lady.** That's the title, that's the spine, and that's the inciting incident. Each duplicate boards a *different reality* of the same flight — the same `Romance` interface, three wildly different concrete implementations. Polymorphism as plot.

The romantic lead in all three: **Marek**, the LOT on-call SRE in a Warsaw basement at 03:00, paged awake by the exact alert *"DUPLICATE SEAT ASSIGNMENT — 17B — MANUAL RECONCILE REQUIRED."* He is, quite literally, the man trying to make Hanna **consistent.** He just doesn't know yet that he's going to fall in love with the incident.

---

## THE ENTRANCE — *cold open, shared by all three branches*

Warsaw Chopin Airport. Hanna stares at her boarding pass: **12A → 17B**. A small, specific injustice — window to middle, a strict downgrade, no explanation. She does what any of us would do: she opens the app and hits **"Restore my seat"** again. And again. And — the spinner won't stop — again.

Cut to Marek's monitors lighting up like a Christmas tree. Cut to three identical Hannas walking down three identical jet bridges in three slightly-wrong-colored cabins. The doors close. The interface is called. **Three implementations execute in parallel.**

---

## ⟶ BRANCH 1 — THE FAIRYTALE

### *Vertical Scaling — "just give the node more resources"*

**Tone:** glossy, golden-hour, Richard Curtis with a champagne flute.

The reconciler resolves *in Hanna's favor.* There's no economy seat left, so the system does the most expensive thing it can: it **scales her up.** 17B → **1A. Business class.** Lie-flat pod, warm nuts in a ceramic dish, a pianist who shouldn't fit on an aircraft.

And there, deadheading to fix a server in Gdańsk, is **Marek** — the voice from the incident call, now a real person two pods away, helplessly explaining what an "idempotency key" is to a woman who finds it, against all odds, *charming.* Slow burn. A shared armrest war that becomes a held hand. A grand gesture at baggage claim involving an apology issued in valid JSON.

**The scalability lesson (the bittersweet undertow):** Vertical scaling is *gorgeous* and it **does not scale.** There's exactly one 1A. You can only make a single seat so luxurious before you hit the ceiling of the airframe. The fairytale works **for one lucky request** and quietly bankrupts the system for everyone behind her. The happy ending is real — but it's a happy path, and happy paths don't survive load. *Roll credits before anyone asks what happened to the other 200 passengers.*

---

## ⟶ BRANCH 2 — THE DISASTER

### *Cascading Failure — "no backpressure, no circuit breaker, the reaper comes"*

**Tone:** *Final Destination* by way of *Airplane!* Dark, fast, very funny, slightly unwell.

Hanna's three retries weren't alone. Every passenger is rage-tapping "Restore my seat." It's a **thundering herd** — a retry storm with zero rate limiting. The booking service browns out, the seat map goes stale, two strangers physically fight over the *real* 17B, and the system, having no graceful degradation, chooses the worst branch of the CAP theorem under partition: **stay Available, abandon Consistency.** Let *everyone* board. Sort it out in the sky. (You will not sort it out in the sky.)

Then Death — sorry, **the OOM killer** — starts garbage-collecting passengers in seat order. Hanna gets a premonition rendered as a *stack trace she can read on the seatback screen.* She and Marek (now frantically SSH'd in from the ground, screaming "ADD A CIRCUIT BREAKER" into a headset) keep cheating deletion one row at a time. Their romance is built entirely on narrowly not-dying together, which it turns out is an *excellent* foundation.

**The biblical Easter egg LOT was begging for:** "LOT" → *Lot.* **Do not look back at the plane.** The one passenger who turns around to film the engine fire turns, instantly, to a pillar of salt (and a `410 Gone`).

**The scalability lesson:** This is what a system *without* backpressure, rate limiting, bulkheads, or a circuit breaker actually does under load — it doesn't slow down, it *dies in order.* Survival in the final act = Marek shipping the one-line fix (`if (!idempotencyKey) reject()`) at the last possible second. The reaper process catches a `429 Too Many Requests` and goes home. They live. Barely. *Throttled, but alive, and in love.*

---

## ⟶ BRANCH 3 — ABSTRACT POLYMORPHISM

### *Everything turns into something else — "too much abstraction and nobody knows what anything IS"*

**Tone:** *Everything Everywhere All At Once* co-written by a UML diagram. Surreal, tender, occasionally a goose.

Here the type system *itself* comes loose. Hanna boards as a `Passenger`, but the cabin keeps **casting** her to other types. The seat is `(Throne) 17B`. Then `(Horse) 17B`. Then a single cell in a spreadsheet labeled `17B` where she lives a full, rich life as a `=VLOOKUP`. Marek arrives as an exception handler and is promptly downcast into a **lighthouse**, then a verb, then a very sincere `interface` with no implementation.

She is cast as **"Lady"** three separate times, via three different inheritance paths — *that's* the "three times a Lady" — and each time she's a Lady, she's a *different kind* of Lady, and none of them can find the others because the equality check is comparing references, not values.

The romance survives by **duck typing**: it doesn't matter what Hanna and Marek *are* this scene — if it walks like love and quacks like love, the runtime treats it as love. Their big kiss happens while they are, respectively, a thundercloud and a parking ticket. It works anyway.

**The scalability lesson:** Abstraction is *what lets a system scale* — swap any implementation behind a stable interface and the whole thing keeps running. But abstract *too* hard and you get a cabin full of `ClassCastException`, where everything is technically interchangeable and nothing is legible. The film's emotional climax is the year's most romantic line of code: **`if (you instanceof Mine) return true;`**

---

## ⟶ POST-CREDITS — *you said surprise me* 🎁

Pick one. Or stack them — they're stingers, they nest.

1. **It was a drill.** Smash cut to a LOT war room. The triple-booking, the storm, the salt pillar — all of it was a **Chaos Monkey** experiment. Someone deliberately killed seat 12A in production "to test resilience." Hanna was never a bug. She was the **test case.** Marek slowly realizes he fell in love with a fault injection. *He requests to run the experiment again.*

2. **The system was Cupid all along.** Reveal that the booking engine is sentient and has been **load-balancing strangers into soulmates for thirty years.** Its objective function isn't seat utilization — it's `maximize(happily_ever_after)`. The 12A→17B rebooking was never a mistake. It moved her exactly one row behind Marek's mother, who needed someone kind in 18B. *The machine knew.*

3. **"Yoore" wins.** Final shot: a database row. `name: YOORE/LADY`. The corrupted instance — the replication-lag typo — was never reconciled and quietly **achieved consciousness.** She's not a passenger; she's a primary key who learned to love. *Black Mirror chime.* (Hanna and Marek never find out they have a daughter made of stale data.)

4. **Sequel hook — *Scalability II: The Sharding.*** A new alert: vertical scaling capped, so the system **scales out.** Hanna's next booking gets **horizontally partitioned across two aircraft** — half of her flies to Kraków, half to Vilnius, eventual consistency promised "within 4–6 business days." Marek, now Staff Engineer, must reunite her shards before the partition becomes permanent. Tagline: *"She's distributed now. He has to make her whole."*

5. **The needle-drop.** Over black, the gate PA crackles to life and a slightly-too-emotional gate agent sings the *entire* Commodores chorus. Last line of the film, deadpan, from Marek: *"...that's not idempotent either."*

---

## Tagline options for the poster

- **"Some requests just want to be retried."**
- **"One seat. Three commits. No regrets (and no rollback)."**
- **"She wasn't overbooked. She was *over*-loved."**

---

## Scalability concept map (for the engineers in the audience)

| Story beat | Scalability concept |
|---|---|
| 12A → 17B rebooking | Load balancing — request rescheduled to a less-busy node |
| Triple booking from one retry loop | Non-idempotent endpoint, no dedup/idempotency key |
| "YOORE/LADY" name corruption | Eventual consistency / replication lag |
| Branch 1 — Business class upgrade | Vertical scaling (scale up) — luxurious, has a ceiling |
| Branch 2 — Retry storm collapse | Thundering herd, no backpressure → cascading failure |
| "Available, abandon Consistency" | CAP theorem under network partition |
| The one-line fix | Circuit breaker / rate limiting (`429 Too Many Requests`) |
| Branch 3 — everything re-casts | Polymorphism & abstraction; over-abstraction → `ClassCastException` |
| "if (you instanceof Mine)" | Type checking / duck typing |
| Sequel — split across two planes | Horizontal scaling / sharding (partition) |
| Post-credit "it was a drill" | Chaos engineering (Chaos Monkey / fault injection) |
