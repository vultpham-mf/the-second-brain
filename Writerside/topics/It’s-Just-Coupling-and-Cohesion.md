# It’s Just Coupling and Cohesion

<primary-label ref="draft"></primary-label>

## Preface

When we talk about writing **good software or software design**, buzzwords like **SOLID**, **Clean Architecture**, **DRY**, **CQRS**, **design
patterns**, and so on often dominate the conversation.
There’s endless debate about the latest architecture trend or which principle you should follow.
But beneath all the acronyms and guidelines, there are two ideas that almost never get enough attention: **coupling and
cohesion**.
Ironically, these are actually the **foundation** of all those other principles—almost every famous
guideline in software design is just a different perspective on how to achieve better coupling and cohesion.

This post aims to give coupling and cohesion the spotlight they deserve.
If you truly understand these two concepts, you no longer need SOLID, DRY, KISS... Let’s dig into why.

### Obligatory AI Reference to Stay Relevant

<img src="first-they-came-for-the-artists-v0-zku9xsa0ob6a1.png" alt="Soon AI will take over programming"/>

**~~It’s 2026, and not mentioning AI feels illegal.~~**

When using AI-assisted coding tools, we expect outputs that are predictable and dependable.
To achieve this, developers are moving away from random, ad-hoc **vibe coding** and toward structured context workflows
that give the AI clear, reproducible instructions.
But how can you be confident your AI prompts are actually producing good results?
The answer is always tied to fundamentals: strong software design—especially in terms of **coupling and cohesion**
—matters just as much with AI as without.

## Why Software Design Matters: The Three Symptoms of Complexity

Before we get into the details of coupling and cohesion, it’s worth pausing to consider:
**Why should we care about software design at all?** The structure and organization of code plays
a big part in how manageable, flexible, and enjoyable a codebase is to work with.
Good design helps teams move faster and with fewer mistakes, while poor design slows everyone down
and complicates even simple changes.

Bad design isn’t always about obvious bugs or failures—instead,
it shows up as persistent pain points that make working with the code harder than it needs to be,
no matter what language or framework you’re using:

- **Change amplification:** When a simple change requires updates in many different places,
  indicating that everything is entangled.

- **Cognitive load:** The second symptom of complexity is cognitive load,
  which refers to how much a developer needs to know in order to complete a task.
  A higher cognitive load means that developers have to spend more time learning the
  required information, and there is a greater risk of bugs because they have missed something important.

- **Unknown unknowns:** When it’s unclear what needs to change, or even what questions to ask,
  because the relevant code and information are hard to discover.

Recognizing these symptoms is the first step toward understanding why coupling and cohesion matter so much. These two
qualities are at the root of most design pain—and the key to avoiding it.

## What Are Coupling and Cohesion?

**Coupling and cohesion** are two foundational concepts in software design that shape how your codebase
evolves, scales, and survives over time. While they’re often mentioned in passing,
many developers struggle to define them clearly, or to recognize their impact on real-world decisions.

**Cohesion** describes how well the elements within a module or component belong together—how focused
and self-contained it is. **Coupling**, on the other hand, measures how much different modules or
components depend on each other—how entangled or isolated your components are.

Why does this matter? Because the level of coupling and cohesion in your codebase determines how painful
or painless it is to maintain, change, and understand. Mastering these concepts leads to cleaner,
more reliable, and more robust software—regardless of what language, framework, or architecture you use.

Let’s break down what each term really means, and why you should care.

### Coupling

To understand coupling, let’s use the example of **replacing the battery in different iPhone models**.

Consider two Apple devices: **iPhone 3** and **iPhone 17 Pro**.
The iPhone 3 is the easiest when it comes to battery replacement—you can simply
open the back and swap out the battery with almost no effort,
and without worrying about damaging anything else. With the iPhone 17 Pro, replacing the battery is far more difficult.
You need to start by removing the expensive display and then carefully separate additional layers,
increasing the risk of damaging the screen or other internal sensors along the way.

<img src="ip-battery-replacement.png" width="700" alt="iPhone Battery Replacement"/>

From an engineering perspective, the relationship between the iPhone and its battery during
replacement demonstrates different degrees of **coupling**. In this case, coupling describes how much
changing one module (the battery) affects others (the rest of the phone).
In the iPhone 3, battery replacement is easy, safe, and doesn’t impact other modules
—we call this **loose coupling**, which is ideal. In the 17 Pro,
accessing the battery risks interfering with or breaking other parts,
like the screen or sensors—this is **tight coupling**, which is lesss desirable because
a change to one part requires risky interaction with others.

In software engineering, **coupling** formally refers to the degree of interdependence
between software modules—a measure of how closely connected different modules or components are.
**Lower (looser) coupling is generally preferred**, as it makes systems easier to understand, modify, and test.

<img src="coupling.png" width="700" alt="Coupling"/>

#### Example: Tight vs. Loose Coupling in Code

Let’s see a practical example from API and web development.

Imagine you first build an API to display product information on a public product page. The frontend only needs the
product's **name**, **image**, and **price**, so you create a simple DTO:

<code-block lang="go">
type ProductDTO struct {
    Name  string  `json:"name"`
    Image string  `json:"image"`
    Price float64 `json:"price"`
}
</code-block>

Later, you receive a requirement to build an API for the **admin product page**. At that time, the admin also needs only
the product's name, image, and price, so you reuse the existing `ProductDTO` to avoid duplication (following the DRY
principle).

However, the problem comes when one of those pages needs to change. Suppose the admin page now needs two new fields: *
*supplier** and **inventory count**. You update `ProductDTO` to satisfy the admin requirement:

<code-block lang="go">
type ProductDTO struct {
    Name          string  `json:"name"`
    Image         string  `json:"image"`
    Price         float64 `json:"price"`
    Supplier      string  `json:"supplier"`
    InventoryCount int    `json:"inventory_count"`
}
</code-block>

Now, both the user-facing and admin APIs use the same DTO, but only the admin API actually needs the new fields. The
user API becomes coupled to changes it doesn't care about, and you suddenly have to test both APIs whenever a new admin
requirement appears. This is **tight coupling**—a change meant for one consumer risks unintended impact for another.

A **better, loosely coupled** design would separate the DTOs **from the very beginning**, even when both APIs initially
need the same fields. That is, instead of sharing a single `ProductDTO` between the user and admin APIs, define distinct
response objects for each use case right away—even if they look identical at first:

<code-block lang="go">
type UserProductResponse struct {
    Name  string  `json:"name"`
    Image string  `json:"image"`
    Price float64 `json:"price"`
}

type AdminProductResponse struct {
    Name string  `json:"name"`
    Image string  `json:"image"`
    Price float64 `json:"price"`
    // Add admin-specific fields later as requirements change
}
</code-block>

<note>
Just because two pieces of code look identical now doesn't mean they should be shared—especially if they're meant 
for different purposes or consumers. Reusing code in these situations can lead to unnecessary coupling and future
headaches as requirements diverge. Sometimes, it's better to keep similar code separate to keep your solutions
flexible and maintainable.
</note>

By anticipating that these APIs will evolve separately, you avoid tight coupling from the start. Later, when the
admin API needs extra fields like **supplier** or **inventory count**, you add them only to `AdminProductResponse`:

<code-block lang="go">
type AdminProductResponse struct {
    Name           string  `json:"name"`
    Image          string  `json:"image"`
    Price          float64 `json:"price"`
    Supplier       string  `json:"supplier"`
    InventoryCount int     `json:"inventory_count"`
}
</code-block>

By keeping these response objects separate, changes made for admin requirements never unintentionally impact the user API.
Each endpoint consistently delivers only what its consumer actually needs—nothing extra, nothing missing. This clear 
separation not only prevents cross-impact, but also demonstrates the first two **SOLID** principles in action: 
**Single Responsibility** and **Open/Closed**.

<note>
Tight coupling—such as shoehorning multiple consumers into a shared DTO—makes your code brittle:
every change increases the risk of unintended side effects and extra testing.  
Loose coupling—separating outward-facing data models for different use cases—lets features 
and requirements evolve independently and safely. Even if it seems less DRY, 
this is almost always the more maintainable and robust approach in the long term.
</note>

### Cohesion

**Cohesion** refers to how closely the operations inside a single module (like a class or function) relate to each
other.
High cohesion means a module does one well-defined thing. Low cohesion means it handles unrelated tasks,
making code harder to maintain and understand.

Unlike **coupling**, cohesion is all about how tightly related the elements
within a single module (such as a class, function, or feature) are. The higher the cohesion among its components,
the better—the module’s elements should work together to achieve one clear, focused purpose.

<img src="cohesion.png" width="700" alt="Cohesion"/>

Think of a **waste sorting system** with **Recycling** bins (for items like plastic bottles and cans) and **Compost**
bins (for things like food scraps). Each bin is designed to handle one specific type of waste, which makes sorting and
processing much easier. This is **high cohesion**, which is **good** because everything in the bin belongs together.

<note>
With <b>cohesion</b>, <b>higher is better</b>: the more closely related a module's responsibilities are, 
the more focused and maintainable it becomes. <b>High cohesion</b> means each part works together for one clear purpose.  
In contrast, with <b>coupling</b>, <b>lower is better</b>: loosely coupled components interact minimally, 
making code more flexible and easier to change.  
Aim for high cohesion within modules and loose coupling between them for the most robust design.
</note>

<img src="waste-sorting-system.png" width="700" alt="Waste sorting system"/>

If someone throws a plastic bottle into the compost bin, problems start to appear. The compost bin now contains
unrelated
items—this is low cohesion. Worse, the composting process is forced to “deal with” plastic, something it was never
designed for. The bins become coupled in a bad way, since what should be a clean separation now creates confusion and
extra work.

<img src="waste-sorting-system-wrong.png" width="700" alt="Waste sorting system wrong"/>

Just like in code, mixing unrelated things together (low cohesion) or forcing unrelated systems to depend on each other
(tight coupling) leads to a mess that’s hard to fix. Clean separation keeps everything easier to manage and understand.

#### Example: Low vs. High Cohesion in Code

Referring back to the **waste sorting system** analogy, a common mistake with cohesion is grouping elements by their "type" 
instead of their actual purpose. For example, we don't put a bottle in the **Recycling** bin just because it's made of
plastic—we put it there because it's **recyclable**. 

The important lesson for cohesion is to group things by **what they are meant to accomplish**, not just by their superficial
traits. The same rule applies in software: achieve high cohesion by organizing modules, components or functions around their true
responsibilities, not simply by technical type or category. Here’s a code example to show what that means:

// TODO example

## Common Violations
// TODO I'll try to complete this section later

### The "Class and Interface" problem

### The "MVC Starter" problem (A guide to "Clean Architecture")
~~I hate to mention "Clean Architecture" as a actual arch, but anyway~~

### The "Everything is a Service" problem

## Conclusion

<note>
// TODO fill content

Coupling and Cohesion isn't just about structure, it is everywhere, represent the relationship of modules, component,
class, or even lines of code..
</note>

## Reference
// TODO add reference