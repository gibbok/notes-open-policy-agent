# Notes on Open Policy Agent

The Open Policy Agent (OPA, pronounced “oh-pa”) is an open source, general-purpose policy engine that unifies policy enforcement across the stack. OPA provides a high-level declarative language that lets you specify policy as code and simple APIs to offload policy decision-making from your software. You can use OPA to enforce policies in microservices, Kubernetes, CI/CD pipelines, API gateways, and more.

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

