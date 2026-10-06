# MCP-RM

**Status: design proposal.** This repository does not yet contain an executable proxy or a demonstrated enforcement boundary.

MCP-RM explores a security proxy for Model Context Protocol (MCP) tool invocations. Its design goal is to apply the formal **Reference Monitor** requirements—complete mediation, tamper-proofness, and verifiability—to an agent's calls to MCP tools. These are requirements to be engineered and tested, not capabilities already delivered by this repository.

## Problem

An MCP tool call can turn an agent's decision into an action against a file system, service, or other resource. Untrusted prompts, retrieved content, and tool results may influence that decision. If authorization is only advisory, or if the client can reach a tool server through another route, neither a policy decision nor an audit trail can reliably cover every invocation.

MCP-RM's proposed scope is the **invocation boundary**: decide whether a particular client may call a particular tool with particular arguments before the request reaches the MCP server. The project does not treat a model's intent or a tool's description as an authorization decision.

## Reference Monitor requirements

The project uses the Reference Monitor concept described by [Anderson's 1972 study](https://csrc.nist.gov/files/pubs/conference/1998/10/08/proceedings-of-the-21st-nissc-1998/final/docs/early-cs-papers/ande72a.pdf) and summarized in the [NIST definition](https://csrc.nist.gov/glossary/term/reference_monitor):

| Property | Requirement for MCP-RM |
| --- | --- |
| **Complete mediation** | Every in-scope `tools/call`, including retries, must cross the enforcement point and receive a policy decision before forwarding. A deployment must prevent a direct client-to-server bypass. |
| **Tamper-proofness** | The agent and untrusted MCP servers must not be able to alter the proxy, its policy, its credentials, or the evidence of its decisions. This depends on deployment isolation and privilege separation as well as code. |
| **Verifiability** | The enforcement core must be small and explicit enough to analyze and test. Policy semantics, failure behavior, and bypass resistance must be demonstrable; logging alone is not proof of correct enforcement. |

## Proposed architecture

```text
                    policy administration
                            |
                            v
Agent host / MCP client -> MCP-RM proxy -> approved MCP server(s)
                            |
                            v
                      decision/audit record
```

The proposed proxy sits between an enrolled MCP client and its configured MCP servers. For each tool invocation, it would bind the caller and destination identity, inspect the requested tool and arguments, evaluate policy, then deny or forward the request. It would return the server's response through the same path and record the decision with enough context to investigate it.

The security boundary requires the client to have no usable direct route or credentials to those servers. Policy administration and audit storage must be outside the agent's control. Transport support, identity binding, policy language, and audit integrity mechanisms remain design decisions; the diagram is an intended topology, not a deployed system.

## Threat model

**In scope for the proposed design:** an agent influenced by malicious instructions in prompts, retrieved data, or tool output; a malicious or compromised MCP server that presents misleading tool metadata or responses; and attempts to invoke a tool outside policy, bypass the proxy, or change policy and decision records from the agent's trust domain.

**Assumptions and limits:** the host and policy administrator are trusted, and deployment controls can force enrolled clients through the proxy. A compromised host or administrator can defeat those assumptions. Calls made outside the enrolled MCP path—including direct APIs, local commands, or other tool systems—are not mediated by MCP-RM. MCP server behavior after an authorized call also remains outside the proxy's enforcement boundary.

## Current status

The repository currently publishes this design framing and an MIT license. It contains no proxy implementation, policy engine or format, deployment configuration, tests, or release. The architecture and controls above are **proposed work**, and the three Reference Monitor properties have **not** been established.

## Next steps

1. Define the protected operations, caller and server identity model, policy inputs, deny-by-default behavior, and failure handling.
2. Select an MCP transport and deployment topology, then document how direct server access and credential bypass are prevented.
3. Build a minimal `tools/call` mediation path with explicit allow/deny decisions and decision records.
4. Test unauthorized calls, retries, policy changes, identity spoofing, proxy failure, and direct-path bypass against the three requirements.
5. Publish a runnable example, supported-scope matrix, known limitations, and evidence from testing or review before making enforcement claims.
