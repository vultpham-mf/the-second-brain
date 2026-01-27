# It’s Just Coupling and Cohesion

<primary-label ref="opinion"/>

## Preface
When we talk about writing **good software or software design**, buzzwords like **SOLID**, **DRY**, and **design patterns** often dominate the conversation. 
There’s endless debate about the latest architecture trend or which principle you should follow.
But beneath all the acronyms and guidelines, there are two ideas that almost never get enough attention: **coupling and cohesion**. 
Ironically, these are actually the **foundation** of all those other principles—almost every famous 
guideline in software design is just a different perspective on how to achieve better coupling and cohesion.

This post aims to give coupling and cohesion the spotlight they deserve. 
If you truly understand these two concepts, you no longer need SOLID, DRY, KISS... Let’s dig into why.

### Obligatory AI Reference to Stay Relevant

<img src="first-they-came-for-the-artists-v0-zku9xsa0ob6a1.png" alt="Soon AI will take over programming"/>

~~It’s 2026, and not mentioning AI feels illegal.~~

When using AI-assisted coding tools, we expect outputs that are predictable and dependable. 
To achieve this, developers are moving away from random, ad-hoc **vibe coding** and toward structured context workflows that give the AI clear, reproducible instructions.
But how can you be confident your AI prompts are actually producing good results?
The answer is always tied to fundamentals: strong software design—especially in terms of **coupling and cohesion**—matters just as much with AI as without.

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

Recognizing these symptoms is the first step toward understanding why coupling and cohesion matter so much. These two qualities are at the root of most design pain—and the key to avoiding it.

## What Are Coupling and Cohesion?
Before you can improve your code, you have to know what you’re aiming for. Coupling and cohesion are the two most fundamental qualities of software architecture.
**Basically, they describe the relationships between each module or component in the system.** Let’s define them and see why they matter so much.

### Coupling
To understand coupling, let’s use the example of **replacing the battery in different iPhone models**.

Consider three Apple devices: **iPhone 3**, **iPhone 14**, and **iPhone 17 Pro**.
The iPhone 3 is the easiest when it comes to battery replacement—you can simply
open the back and swap out the battery with almost no effort,
and without worrying about damaging anything else. For the iPhone 14,
it’s a bit more complicated: you have to carefully disconnect other
components before you can safely access and remove the battery.
Finally, with the iPhone 17 Pro, replacing the battery is the most difficult.
You need to start by removing the expensive display and then carefully separate additional layers,
increasing the risk of damaging the screen or other internal sensors along the way.

<img src="ip-battery-replacement.png" width="750" alt="iPhone Battery Replacement"/>

From an engineering perspective, the relationship between the iPhone and its battery during
replacement demonstrates different degrees of `coupling`. Coupling describes how much
changing one component (the battery) affects others (the rest of the phone). 
In the iPhone 3, battery replacement is easy, safe, and doesn’t impact other components
—we call this “loose coupling,” which is ideal. In the iPhone 14 and especially the 17 Pro,
accessing the battery risks interfering with or breaking other parts, 
like the screen or sensors—this is “tight coupling,” which is less desirable because 
a change to one part requires risky interaction with others.

In software engineering, **coupling** formally refers to the degree of interdependence between software modules—a measure of how closely connected different components or classes are. Lower (looser) coupling is generally preferred, as it makes systems easier to understand, modify, and test.

[//]: # (//TODO fix the format)
// TODO fix the format

// TODO fix image scale

<img src="coupling.png" width="750" alt="Coupling"/>

#### Example: Tight vs. Loose Coupling in Code

Suppose we need a program that sends notifications to users.  

<tabs>
    <tab title="Tightly coupled example">
<code-block lang="python">
class EmailService:
    def send(self, message):
        print(f"Sending email: {message}")

class NotificationManager:
    def __init__(self):
        self.email_service = EmailService()  # Directly creates dependency

    def notify(self, message):
        self.email_service.send(message)
</code-block>
    </tab>
</tabs>

Here, `NotificationManager` directly depends on `EmailService`. If we want to replace emails with SMS or push notifications, we have to modify `NotificationManager`. This is **tight coupling**.

<tabs>
    <tab title="Loosely coupled example">
<code-block lang="python">
class Notifier:
    def send(self, message):
        pass

class EmailService(Notifier):
    def send(self, message):
        print(f"Sending email: {message}")

class NotificationManager:
    def __init__(self, notifier: Notifier):
        self.notifier = notifier  # Uses abstraction

    def notify(self, message):
        self.notifier.send(message)
</code-block>
    </tab>
</tabs>

Now, `NotificationManager` can accept any notifier (Email, SMS, etc.) via constructor. This decoupling makes the code flexible and easier to maintain.

**In summary:**  
- *Tight Coupling* = components rely directly on each other (hard to change)
- *Loose Coupling* = components interact via interfaces/abstractions (easy to change)

Reducing coupling improves modularity and adaptability of your codebase.


### Cohesion
// TODO fix format

// TODO provide a better example, just like the iPhone example above

Cohesion refers to how closely the operations inside a single module (like a class or function) relate to each other. High cohesion means a class does one well-defined thing. Low cohesion means it handles unrelated tasks, making code harder to maintain and understand.

<img src="cohesion.png" width="750" alt="Cohesion"/>


<tabs>
    <tab title="Low cohesion example">
<code-block lang="python">
class Utils:
    def send_email(self, message):
        print(f"Sending email: {message}")

    def calculate_salary(self, hours, rate):
        return hours * rate

    def save_to_file(self, filename, content):
        with open(filename, 'w') as f:
            f.write(content)
</code-block>
    </tab>
</tabs>

In this example, the `Utils` class does unrelated things: sending emails, calculating salary, and file operations. This is **low cohesion**—there’s no clear responsibility or purpose to the class.

<tabs>
    <tab title="High cohesion example">
<code-block lang="python">
class SalaryCalculator:
    def calculate_salary(self, hours, rate):
        return hours * rate

class EmailService:
    def send_email(self, message):
        print(f"Sending email: {message}")

class FileSaver:
    def save_to_file(self, filename, content):
        with open(filename, 'w') as f:
            f.write(content)
</code-block>
    </tab>
</tabs>

Here, each class takes responsibility for one thing. `SalaryCalculator` focuses only on salary calculations, `EmailService` on emails, and `FileSaver` on file storage. This is **high cohesion**.

**In summary:**  
- *Low Cohesion* = a class/function does many unrelated things (hard to maintain & understand)
- *High Cohesion* = a class/function does one well-defined thing (clear, robust, maintainable)

Striving for high cohesion leads to code and products that are easier to reuse, refactor, and reason about.

<note>
// TODO fill content

Coupling and Cohesion isn't just about structure, it is everywhere, represent the relationship of modules, component, class, or even lines of code..
</note>

## Common Violations

### The "Class and Interface" problem

### The "MVC Starter" (A guide to "Clean Architecture")

### The Constants file (A place to store constant)

### The "Everything is a Service" problem

## Conclusion

## Reference