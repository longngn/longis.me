---
title: "Introducing DAG: A simple way to design backend application"
date: 2020-12-03
---

At some point in writing API application or microservice, we have probably struggled with code architecture and organization. Modern technologies for building backend applications like Node.js or Go don't even have any idiomatic way for project structure. That gives us the freedom of designing the most suitable architecture for our use case but also makes it easy to shoot ourselves in the foot.

<!--more-->

In this blog post, I am going to introduce a new approach: Directed Acyclic Graph, also known as DAG. For a formal definition, better to take it from [Wikipedia](https://en.wikipedia.org/wiki/Directed_acyclic_graph):

> A directed acyclic graph (DAG) is a directed graph with no directed cycles. That is, it consists of vertices and edges (also called _arcs_), with each edge directed from one vertex to another, such that there is no way to start at any vertex v and follow a consistently-directed sequence of edges that eventually loops back to v again.

{{< figure src="/img/dag-1.png" alt="DAG example" position="center" style="width: 50%; margin: auto;" >}}

In layman's terms, DAG is a graph that flows in one direction, where no element can reference back to itself (cycle). Computer scientists are bad linguists, acyclic graph doesn't mean graph with a cycle but it means graph without any cycle, but come on! [[1]](#notes)

I actually have this idea while reading about Apache Airflow which is a data ETL management and scheduling tool (it's awesome!). Airflow is built on the concept of defining data pipeline as DAG, where data can only flow in one direction. Most backend applications are handling data and I think that it's like two complementing ETL pipelines:

- One that receives data from requests, processes it (validate, calculate, business logic,...), and loads it somewhere else (other services, databases, message queues,...)
- And one that extracts data from somewhere (databases, other services,...), processes it and answers clients

Because if we invert all edges of a DAG, the resulting graph is still a DAG (proof below) [[2]](#notes). So why not applying DAG model to this problem? Even though the method is language-agnostic and paradigm-agnostic, I will use Java terms like class, constructor, method,... because they're the most familiar to us.

So a DAG-like program is compromised of nodes, each node here is a unit of data processing logic. Simply put, it's a class with methods. Its children (also called "dependencies") are injected by constructor method. On request, a method receives data from its arguments, processes data, and passes them down to its children. And then it receives some other data from its children and forms the correct data to return to its parent. No side-effect.

Side-effects are only allowed at root nodes (e.g. return data to clients) and leaf nodes (e.g. write data to databases or call other services). Because most of the time the node doesn't store data, we could use a singleton instance for each node.

Of course, there are classes that we want to import from everywhere like utilities or models classes. Treat them like external (third-party) dependencies, which means that any node can import them but they can't import any node. Your graph can either be wide or deep but personally, I prefer a wide graph over a deep graph as it expresses a more modular and composable design.

Dependency-injection can be done manually or by using a dependency injection tool. If you have been using DI tools your whole life, manual DI sounds like washing clothes by hand, but try it if you haven't. Your development server startup time will thank you later. Most DI tools are either reflection-based or code-generation-based and it would be overkill for small to medium-sized projects.

Here is an example design of a CMS backend application.

![CMS example](/img/dag-2.png)

I've found this method works well for me so far. It doesn't change my program structure that much, sometimes it made me write even more code, but it gives me peace of mind. I know that when I want to test any node, I could mock its dependant nodes safely because there is no transitive dependency leading to some faraway code that has side-effects. I know that when writing a node, I'd only need to take into account its dependencies and not need to care about anything above it. I know that when circular dependency appears, my code stinks and there's tight coupling somewhere.

Structuring your program like a DAG is nice and all until you want to break out of its yoke. You want to have a cycle. Let's say a child class wants to use some code from its parent class. There are a few solutions:

- Maybe the child and parent are not that distinct. If they have tight coupling, just merge them into one node.
- The code that both the child and parent need is something logically stand-out that could justify a node of its own. Refactor it to be a "grandchild" node and make it dependencies of both the child and parent node. (see picture)
- The code the child need is small and not something highly related to the parent class, so we can copy them to the child class and use it there.

Wait, isn't duplication bad? In my opinion, duplication is better than premature abstraction (i.e. abstraction that doesn't really abstract anything).

{{< figure src="/img/dag-3.png" alt="refactor circular dependencies" position="center" caption="Yeah I know, my drawing is not the best." >}}

This approach is not intended to be a replacement for your current design, but more like a complement. You don't need to create new abstractions or rulesets for this, just take it as a mindset, or an architecture blueprint. If you find yourself struggling with a messy, tangled, out-of-control codebase, maybe you should consider giving this a try. I know that your mileage may vary, so let me know what you think and your experience in the comments section below!

### Notes

_[1] Correction: Actually, a- prefix has Greek roots and it means without, like atheism, asexual, asocial,... So acyclic means without any cycle, ironically._

_[2] Informal proof: If G is a valid DAG, G has topological ordering with every edge of G directs from a lower-order vertex to a higher-order vertex. If G' is G with its edges inverted, the topological ordering of G' is also an inverted version of the topological ordering of G. Which means every edge of G' also directs from a lower-order vertex to a higher-order vertex. Hence G' is also a DAG. QED._
