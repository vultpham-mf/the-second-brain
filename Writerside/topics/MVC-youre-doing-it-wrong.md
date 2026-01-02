# MVC: You’re Doing It Wrong.

<primary-label ref="research"/>
<secondary-label ref="software-arch"/>
<secondary-label ref="mvc"/>

<img src="mvc-meme.jpg" alt="Just use MVC"/>

## Overview
// TODO

## The "Pluto Problem": Why Definitions Evolve
For 76 years, **Pluto** (Sao Diêm Vương) was a planet. In 2006, it wasn't.

The change didn't happen because Pluto moved or shrank; 
it happened because our definition of a planet matured.
As we discovered more objects in deep space, the old category became too broad to be useful.

<img src="pluto.jpeg" alt="Pluto" width="700"/>

<procedure title="Definitions will change...">
   <p>
    Definitions aren't static truths—they are tools we use to organize our current understanding of the world.
    When our knowledge expands or our technology improves,
    we must "resize" our tools to fit a new reality.
    </p>
    <p>
    Much like the reclassification of Pluto, the definitions of technical terminology can be updated to incorporate new findings from our engineers.
    </p>
</procedure>

## What isn't quite right with how we think about MVC?

Searching for keywords like "MVC" on Google, you'll find a lot of articles about MVC, I pick some of them from the popular sources:
- [MVC on GeeksforGeeks](https://www.geeksforgeeks.org/software-engineering/mvc-framework-introduction/)
- [MVC on codecademy](https://www.codecademy.com/article/mvc-architecture-model-view-controller)

You don't need to read all of them, here is a short summary of how they describe MVC:
<table>
    <tr>
        <td>Model</td>
        <td>View</td>
        <td>Controller</td>
    </tr>
    <tr>
        <td>
            <list>
                <li>Enforcing business rules.</li>
                <li>Manages database interactions.</li>
            </list>
        </td>
        <td>two</td>
        <td>three</td>
    </tr>
</table>


## The history of MVC

### First official documentation: Smalltalk-80

**Smalltalk-80** (A programming language and interactive application development environment, see [Wikipedia](https://en.wikipedia.org/wiki/Smalltalk)) 
was known as the early documenting the MVC pattern in their book Smalltalk-80: The Language and its Implementation, released in around **1983**.

