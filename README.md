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
