# Navigating Complexity: From Confusion to Clarity through clear perspectives

No matter who we are — developers, architects, or consultants — in most IT projects we throw around the word “complexity” like confetti at a New Year’s party. “This is too complex,” we say. “Let’s reduce complexity here.” But have you ever paused mid-sentence and wondered: *What exactly are we talking about?*

Through research and reflection on this question, I’ve discovered two perspectives that have transformed how I approach complexity in software development: understanding **where** complexity lives in our systems (local versus global) and **why** it exists in the first place (essential versus accidental). These frameworks provide a more structured way to tackle complexity discussions and decisions.

However, this post isn’t meant to be a compact exploration of just these two perspectives. Instead, it’s a broader reflection on my learnings about complexity — including digressions into related ideas and prevention strategies. My goal is to share insights that not only help you approach complexity more systematically — but also encourage deeper reflection on the topic.

## 🔍 Why I Started Questioning “Complexity”

I’ve spent countless hours in retros, pair programming sessions, and architectural discussions where everyone nods knowingly about complexity. Yet I suspect we’re often talking about entirely different things. It’s like a group of people describing an elephant while blindfolded — each touching a different part and drawing wildly different conclusions.

Think about the last time your team got stuck in an endless loop of “this is complex” statements without making progress. Sound familiar?

I noticed that without a more precise way to discuss complexity, we kept having surface-level conversations that went nowhere. Whether we were debating code refactoring or planning architecture, the same vague complaints kept surfacing while the real issues remained hidden.

This pattern became impossible to ignore:

- We’d identify something as “complex” but couldn’t agree on what made it complex
- Solutions often missed the mark because we were solving different problems
- Team discussions felt frustratingly circular, with everyone talking past each other

These weren’t isolated incidents — they were symptoms of a deeper communication problem that sparked my curiosity to dig deeper into what complexity actually means in our context.

## 📖 A Brief Digression: Complex vs. Complicated

Before diving deeper, there’s an important distinction worth clarifying that often muddles our complexity discussions: the difference between the terms “complex” and “complicated.” While we use these terms interchangeably in everyday conversation, they describe fundamentally different phenomena.

When things like software systems are **complicated**, they have many parts, but the relationships between those parts are predictable and well-understood. Think of a mechanical watch. It has hundreds of components, but each part has a clear, deterministic function.

On the other hand, when things are **complex**, they exhibit emergent behaviors from the interactions of their parts. The behavior of the whole cannot be predicted simply by understanding the individual components. Consider a traffic jam during rush hour. Even if you understand every driver’s behavior, you can’t predict exactly how traffic will flow or where bottlenecks will form.

### Why this matters for navigating complexity

This distinction is crucial because it shapes our solution approach. Complicated problems respond well to decomposition and systematic analysis. Complex problems instead require different strategies like experimentation, adaptation, and accepting that some uncertainty cannot be eliminated.

The perspectives I’ll share below primarily address complexity in software systems. Those unpredictable properties that emerge when components interact in ways difficult to fully anticipate or control. Understanding this distinction helps us select the right tools for each type of problem.

## 🎯 Two Ways of Looking at Complexity

Through my exploration, I discovered several ways to categorize and think about complexity. While many valuable perspectives exist, I found myself drawn to two particular dimensions that cut through most of the confusion I was experiencing. These perspectives proved especially useful because they complemented each other. They include:

- **Local & Global complexity**: Answering the question of where complexity lives in our systems
- **Essential & Accidental complexity**: Considering why the complexity exists in the first place

Let me illustrate these two concepts with a concrete example we can explore together.

## 📫 Our Example: An e-commerce notification system

Imagine you’re working on a microservice architecture for an e-commerce platform. Your team manages the notification service that sends order confirmations, shipping updates, and promotional emails. Over the time, this seemingly simple service has become a source of constant headaches.

Let’s use this scenario to explore how these perspectives might help us think differently.

## 📍 The “Where” Question: Locating Complexity in Your System

The first dimension focuses on where complexity manifests in our systems: Whether it’s concentrated within individual components or emerges from their interactions. This perspective is explored in detail by Vlad Khononov in his work on [untangling microservices](https://vladikk.com/2020/04/09/untangling-microservices/#:~:text=In%20our%20context%2C%20local%20complexity,and%20dependencies%20between%20the%20services) and his book [Learning Domain-Driven Design](https://www.oreilly.com/library/view/learning-domain-driven-design/9781098100124/).

### Local Complexity: The Devil in the Details

When most developers hear the term “complexity,” this is likely what first comes to mind. It’s the type we encounter daily in code reviews and debugging sessions, making it the most familiar and immediate form of complexity.

Local complexity resides within individual components, functions, or services. It’s the gnarly conditional logic, the nested loops processing data, or the state machines handling workflows. Consider this function from our notification service:

```kotlin
// Local complexity example
fun formatNotification(user: User, event: Event, preferences: Preferences) {
  if (event.type == "order" && user.tier == "premium") {
    if (preferences.email && preferences.promotions) {
      // 15 more lines of conditional logic...
    }
  }
  // You get the idea...
}
```

Functions like this are locally complex — difficult to understand, test, and modify in isolation. Yet this inward focus on individual components tells only half the story.

### Global Complexity: The Web We Weave

Global complexity emerges from the interactions between components, services, and systems. It’s not about any single piece being complicated; it’s about how the pieces fit together and influence each other.

To illustrate this perspective more clearly, consider how our notification service orchestrates a typical order confirmation:

```kotlin
fun sendOrderConfirmation(orderId: String) {
  val order = orderService.getOrder(orderId)
  val user = userService.getUser(order.userId)
  val preferences = userService.getNotificationPreferences(order.userId)

  if (preferences.email) {
    emailService.send(formatEmail(order, user))
  }
  if (preferences.sms) {
    smsService.send(formatSMS(order, user))
  }
  auditService.logNotification(orderId, "confirmation_sent")
}
```

While each individual service call might be simple, the coordination between them creates global complexity through:

- Dependencies on multiple services (like user-, order-, inventory-, and payment-service)
- Complex choreography for handling failed notifications across multiple channels
- Data consistency challenges when user preferences change mid-notification-flow

### The Complexity Trade-Off

When comparing these two types of complexity, an interesting pattern emerges: complexity behaves like a conserved quantity. You might have perfectly clean code in each service (low local complexity) but still struggle with the system as a whole (high global complexity).

Think of it like this: When you break down a monolithic application into microservices, you’re not eliminating complexity — you’re relocating it from inside components to between them. The business logic doesn’t disappear — it splits up. This creates a trade-off: individual services become easier to understand, but you gain new complexity from aspects like network communication and coordination challenges.

The insight? We’re often not reducing complexity — we’re choosing where to place it. And that choice matters because different locations require different management strategies. This brings us to our second crucial question: why does complexity exist in the first place? This is where the second perspective comes into play.

## 🤔 The “Why” Question: Understanding Complexity’s Origins

The second perspective examines the origin of complexity. It originates from Fred Brooks’ influential essay [“No Silver Bullet — Essence and Accident in Software Engineering”](https://en.wikipedia.org/wiki/No_Silver_Bullet) published in 1986. Like the “where” perspective, it also divides complexity into two types. It examines complexity based on root causes — whether it comes from the inherent nature of the problem we’re solving (essential) or from the choices we make in our solutions (accidental).

### Essential Complexity: The Problem’s True Nature

Essential complexity is inherent to the problem you’re solving. It cannot be eliminated — only managed. This complexity exists because the real world is inherently complex.

In our notification system, essential complexity includes:

- Business rules about when and how to notify different customer segments
- Legal requirements for email preferences and data protection
- The inherent complexity of coordinating across multiple communication channels

**A small denote on temporal complexity
Despite the fact that it cannot be eliminated, it’s also worth noting that essential complexity isn’t always static. Some complexity remains stable over time (like core business rules), while other aspects are dynamic (like regulatory requirements that may change frequently).

This temporal dimension should influence how you approach complexity. For dynamic requirements, you might invest in more flexible patterns despite their initial complexity. For stable requirements, you can choose solid, well-understood solutions instead of over-engineering for changes that may never come.

### Accidental Complexity: The Problems We Create

Accidental complexity arises from our solutions, tools, and approaches. This is complexity we’ve introduced through our choices — and therefore one we can potentially eliminate.

Examples in our notification service might include:

- Over-engineered abstractions that seemed justified “for future flexibility”
- Technical debt from quick fixes that became permanent
- Fragmented data models across services
- Infrastructure decisions, like decisions for a database, that is inappropriate for the use-case

### The Reducibility Question

In summary, the “why” perspective also shows us an important truth: complexity comes from different sources, and not all of it can be eliminated.

Essential complexity is the core complexity that comes from the business problem itself. You can’t get rid of it without changing what you’re trying to accomplish. Accidental complexity, however, comes from our implementation choices and offers real opportunities for simplification.

## 🗂️ When Perspectives Combine

When I finished exploring each perspective on its own, I wondered whether combining them might unlock additional insights that neither could provide alone. This experiment in connecting the “where” and “why” dimensions revealed four logical combinations, each with different characteristics of complexity and specific strategies for addressing it:

### **1. Local + Essential: Core Domain Logic
When business rules are implemented in code, you typically get complex domain logic within individual functions or classes. The goal isn’t elimination but proper encapsulation and clear expression. I’d suggest considering [domain-driven design principles](https://martinfowler.com/bliki/DomainDrivenDesign.html), rich models, and clear abstractions as strategies for managing this inherent complexity.

### **2. Local + Accidental: Unnecessary Code Complexity
When we create unnecessary complexity within components, we end up with code that’s more complex than it needs to be. This typically results from premature optimization or misapplied patterns. Refactoring, simplification, and removing unnecessary abstractions might be worth exploring as approaches to address this complexity.

### **3. Global + Essential: Inherent System Coordination
When domain requirements demand coordination across multiple components, you face the inherent complexity of orchestrating services or managing distributed state. Clear boundaries, event-driven architectures, and eventual consistency patterns could help manage the coordination challenges that naturally arise in distributed systems.

### **4. Global + Accidental: Architectural Debt
When poor design decisions create unnecessary system-wide complications, you get complexity introduced by flawed service boundaries, shared databases, or accumulated technical decisions. Service redesign, clear interfaces, and systematic decoupling strategies might be approaches to consider for addressing complexity that stems from architectural choices rather than domain requirements.

![An approach to combine & categorize complexity perspectives](https://cdn-images-1.medium.com/max/800/1*HbzrkcWK9Zz58_afDfTjog.png)

An approach to combine & categorize complexity perspectives

## 🎯 What These Perspectives Have Changed for Me

Since exploring these concepts, I think one can be more precise when thinking and talking about complexity. In my opinion & experience, this can be helpful in several contexts.

**Architectural and design discussions** become more targeted. Instead of vaguely stating “this service design seems too complex,” one might explore deeper questions like:

> “Is this complexity coming from the actual business problem, or did we create it by drawing service boundaries in the wrong places?”
> 

Or when evaluating design alternatives, teams can consider:

> “If we consolidate these services, are we reducing the total complexity — or are we just shifting global & local complexity around?”
> 

**Code reviews could benefit as well.** Rather than simply flagging a function as “complex,” one might provide more constructive feedback:

> “This function handles multiple business rules, which creates some local complexity. Should we consider using patterns like strategy-pattern to make this more manageable?”
> 

## 🌟 The Broader Benefits I See

To sum up: Based on my exploration, reflection, and usage of these perspectives, I believe several benefits emerge when teams tackle complexity more systematically. They include:

- **Sharper conceptual clarity.** Understanding the distinction between “complex” and “complicated,” as well as the different types and perspectives of complexity, provides a clearer view of what we’re actually dealing with. This conceptual foundation is essential for making better decisions.
- **More strategic effort and realistic expectations.** Teams can focus their energy where it matters most — reducing accidental complexity while accepting that essential complexity is part of the problem domain. This approach helps with evaluating trade-offs more systematically and helps teams recognize whether they’re truly reducing complexity or merely redistributing it throughout the system.
- **Better communication leading to targeted solutions.** Having specific vocabulary for different types of complexity helps teams discuss what they’re actually trying to solve and decide whether complexity is worth documenting, refactoring, or accepting as-is, naturally leading to more targeted architectural and code-level solutions.

## 🤔 Keeping Expectations Realistic

Let’s be realistic — will everyone magically adopt to this way of thinking? Probably not. At least not immediately. Getting teams to change how they talk about problems is notoriously difficult. And not everyone will find these distinctions as useful as I have. Team dynamics, time pressures, and existing knowledge as well as communication patterns all influence whether approaches like this actually stick.

But even if it doesn’t transform the approach of your whole team overnight, this framework might help you ask better questions or suggest more targeted solutions. Sometimes that’s enough to shift a conversation in a more productive direction.

## 🛡️ A Note on Prevention: Beyond Managing Complexity

While these perspectives help us understand and discuss existing complexity, it is equally important to prevent unnecessary complexity from being introduced in the first place. The frameworks I’ve shared focus on navigating complexity once it’s present, but how do we avoid creating accidental complexity from the beginning?

Here are some proactive strategies — though certainly not an exhaustive list:

**Decision Transparency and Documentation
Making architectural decisions explicit helps teams avoid repeating past mistakes and provides context for future changes. Architecture Decision Records (ADRs) are particularly valuable here. My colleague has written [a post about ADRs](https://medium.com/miragon/how-to-document-your-software-architecture-decisions-3e357f5bcfd4) that covers their fundamentals. When decisions are documented as ADRs, teams can better evaluate whether they’re introducing essential or accidental complexity — or if it’s even necessary, considering the alternatives.

**Continuous Architectural Improvement
Regular discussion and review processes prevent complexity from accumulating unnoticed. The [Thoughtworks Technology Podcast](https://www.thoughtworks.com/de-de/insights/podcasts/technology-podcasts) series on the [Architecture Advice Process](https://www.thoughtworks.com/en-de/radar/techniques/architecture-advice-process) describes a methodology that involves all team members in architectural decisions, not just a select few. This approach fosters a culture of continuous improvement instead of reactive complexity management.

**Fitness Functions and Automated Guardrails
Automated tools can prevent many forms of accidental complexity. Architectural testing tools like [ArchUnit](https://www.archunit.org/) encode design decisions as executable tests, catching violations before they become entrenched. Similarly, tools like [Konsist](https://github.com/LemonAppDev/konsist), [ktLint](https://pinterest.github.io/ktlint/latest/), [ESLint](https://eslint.org/), or [Prettier](https://prettier.io/) ensure teams stick to agreed-upon coding standards, preventing the gradual drift that often leads to unnecessary complexity.

This entire topic of automated complexity prevention deserves deeper exploration — something I’m planning to cover in a future post.

These prevention strategies complement the perspectives we’ve discussed and encourage thinking beyond just managing existing complexity. The goal isn’t to eliminate all complexity — essential complexity will always exist — but to be more intentional about the complexity we introduce through our choices.

## 🚀 Worth a Try?

I’m curious if these perspectives resonate with your experience. Maybe try looking at a system your team considers “complex” through these lenses:

1. **Pick something** your team struggles with
2. **Try categorizing** the complexity you see
3. **Notice** if it changes how you think about potential solutions

## 🤝 What’s Your Experience?

I’m genuinely curious about how others think about and discuss complexity. While these concepts have been valuable in my work, I recognize that every team and context brings different challenges.

What mental models do you use when approaching complexity? Have you discovered effective ways to discuss complexity with your teams — or are you facing complexity challenges that seem to defy categorization?

I’d love to hear your thoughts — whether they expand on these ideas or offer completely different perspectives.