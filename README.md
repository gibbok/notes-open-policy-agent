# Notes on Open Policy Agent 

The Open Policy Agent (OPA, pronounced “oh-pa”) is an open source, general-purpose policy engine that unifies policy enforcement across the stack. OPA provides a high-level declarative language that lets you specify policy as code and simple APIs to offload policy decision-making from your software. You can use OPA to enforce policies in microservices, Kubernetes, CI/CD pipelines, API gateways, and more.

> OPA is a lightweight general-purpose policy engine that can be co-located with your service. You can integrate OPA as a sidecar, host-level daemon, or library.

> Services offload policy decisions to OPA by executing queries. OPA evaluates policies and data to produce query results (which are sent back to the client). Policies are written in a high-level declarative language and can be loaded dynamically into OPA remotely via APIs or through the local filesystem.

<https://www.openpolicyagent.org/>

OPA decouples policy decision-making from policy enforcement.

<img width="1022" alt="Screenshot 2024-06-21 at 12 18 53 PM" src="https://github.com/gibbok/notes-open-policy-agent/assets/17195702/3c79446e-7f45-4719-9f77-56a14b762ee1">

OPA generates policy decisions by evaluating the query input against policies and data. OPA and Rego are domain-agnostic so you can describe almost any kind of invariant in your policies. For example:

- Which users can access which resources.
- Which subnets egress traffic is allowed to.
- Which times of day the system can be accessed at.

Policy decisions are not limited to simple yes/no or allow/deny answers. Like query inputs, your policies can generate arbitrary structured data as output.

OPA policies are expressed in a high-level declarative language called Rego. Rego (pronounced “ray-go”) is purpose-built for expressing policies over complex hierarchical data structures.

To install:

```shell
brew install opa
```
You can load policy and data files into the REPL by passing them on the command line. By default, JSON and YAML files are rooted under data.

## Philosophy

A policy is a set of rules that governs the behavior of a software service. That policy could describe rate-limits, names of trusted servers or accounts a user can withdraw money from.

Authorization is a special kind of policy that often dictates which people or machines can run which actions on which resources.

Authorization is sometimes confused with Authentication: how people or machines prove they are who they say they are. Authorization and more generally policy often utilize the results of authentication (the username, user attributes, groups, claims), but makes decisions based on far more information than just who the user is. 

Today policy is often a hard-coded feature of the software service it actually governs. Open Policy Agent lets you decouple policy from that software service so that the people responsible for policy can read, write, analyze, version, distribute, and in general manage policy separate from the service itself.

### What is Policy?

Policies are set of rules essential to the long-term success of organizations because they encode important knowledge about how to comply with legal requirements, work within technical constraints, avoid repeating mistakes, and so on.

### What is Policy Decoupling?

Software services should allow policies to be specified declaratively, updated at any time without recompiling or redeploying, and enforced automatically.

### OPA Document Model

OPA policies (written in Rego) make decisions based on hierarchical structured data. Sometimes we refer to this data as a document, set of attributes, piece of context, or even just “JSON”.

- OPA policies can make decisions based on arbitrary structured data.
- OPA itself is not tied to any particular domain model.
- OPA policies can represent decisions as arbitrary structured data (e.g., booleans, strings, maps, maps of lists of maps, etc.)

Data can be loaded into OPA from outside world using push or pull interfaces that operate synchronously or asynchronously with respect to policy evaluation. We refer to all data loaded into OPA from the outside world as base documents. These base documents almost always contribute to your policy decision-making logic.
However, your policies can also make decisions based on each other. Policies almost always consist of multiple rules that refer to other rules (possibly authored by different groups). In OPA, we refer to the values generated by rules (a.k.a., decisions) as virtual documents. The term “virtual” in this case just means the document is computed by the policy, i.e., it’s not loaded into OPA from the outside world.

In Open Policy Agent (OPA), virtual documents are data structures that are  computed by the policies themselves, as opposed to being loaded from external sources [4].  They essentially represent the outcome of evaluating OPA rules.

Dynamically generated: Unlike base documents (which contain pre-loaded data), virtual documents are created on-the-fly during policy evaluation [1].
Structure: They can hold the same kind of information as base documents, including numbers, strings, lists, and maps [4].
Access: You can reference virtual documents using the same dot/bracket notation as base documents, making it easy to integrate them into your policies [4].
Location control: Policies themselves control where virtual documents are stored within the data tree using the package directive in the Rego policy language [4].

Rego lets you refer to both base and virtual documents through a global variable called data. Similarly, OPA lets you query for both base and virtual documents via the /v1/data HTTP API [3]. This is why queries for just data (or data.foo or data.foo.bar, etc.) return the combination of base and virtual documents located under that path.

Since base documents come from outside of OPA, their location under data is controlled by the software doing the loading. On the other hand, the location of virtual documents under data is controlled by policies themselves using the package directive in the language.

Base documents can be pushed or pulled into OPA asynchronously by replicating data into OPA when the state of the world changes. This can happen periodically or when some event (like a database change notification) occurs. Base documents loaded asynchronously are always accessed under the data global variable. On the other hand, base documents can also be pushed or pulled into OPA synchronously when your software queries OPA for policy decisions.


Data loaded asynchronously into OPA is cached in-memory so that it can be read efficiently during policy evaluation. Similarly, policies are also cached in-memory to ensure high-performance and high-availability. Data pulled synchronously can also be cached in-memory. For more information on loading external data into OPA, including tradeoffs, see the External Data page.


Simple example:

https://play.openpolicyagent.org/p/bw1179qOPn

Input:

```json
{
    "user": "bob"
}
```

Data:

```json
{
    "user_roles": {
        "alice": [
            "admin"
        ],
        "bob": [
            "employee",
            "billing"
        ],
        "eve": [
            "customer"
        ]
    }
}
```

Rego:

```
package app.test

import rego.v1

# By default, deny requests.
default allow_user := false

# Allow admins to do anything.
allow_user if user_is_admin

# user_is_admin is true if "admin" is among the user's roles as per data.user_roles
user_is_admin if "admin" in data.user_roles[input.user]
```

## What is Rego?

It is a  query language insiped by Datalog.

Rego queries are assertions on data stored in OPA. These queries can be used to define policies that enumerate instances of data that violate the expected state of the system.

Rego focuses on providing powerful support for referencing nested documents and ensuring that queries are correct and unambiguous.

## How to test policies?

OPA also gives you a framework that you can use to write tests for your policies.

## Bundle API
When external data changes infrequently and can reasonably be stored in memory all at once, you can replicate that data in bulk via OPA’s bundle feature. The bundle feature periodically downloads policy bundles from a centralized server, which can include data as well as policy. Every time OPA gets updated policies, it gets updated data too. You must implement the bundle server and integrate your external data into the bundle server–OPA does NOT help with that–but once it is done, OPA will happily pull the data (and policies) out of your bundle server.

## Evaluating Policies
OPA supports different ways to evaluate policies.

- The REST API returns decisions as JSON over HTTP.
- The Go API (GoDoc) returns decisions as simple Go types (bool, string, map[string]interface{}, etc.)
- WebAssembly compiles Rego policies into Wasm instructions so they can be embedded and evaluated by any WebAssembly runtime
- Custom compilers and evaluators may be written to parse evaluation plans in the low-level Intermediate Representation format, which can be emitted by the opa build command
- The SDK provides high-level APIs for obtaining the output of query evaluation as simple Go types (bool, string, map[string]interface{}, etc.)

https://blog.openpolicyagent.org/partial-evaluation-162750eaf422
