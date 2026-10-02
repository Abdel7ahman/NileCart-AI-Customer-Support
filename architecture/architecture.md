# NileCart Architecture Notes

> **Status:** Draft v1. This describes the intended design. None of it is implemented yet.

## Principle

The language model understands language. Application code decides what is allowed and what actually happens.

## Flow

```text
Customer Message
        ↓
Input Pre-processing
        ↓
LLM Request Understanding (Prompt 1)
        ↓
Structured Output Validation
        ↓
Tool Selection and Orchestration
        ↓
Business Rules and Authorization
        ↓
Response Generation (Prompt 2)
        ↓
Safety and Output Checks
        ↓
Final Customer Response
```

## Who is responsible for what

| Step | Owner | What it does |
|---|---|---|
| Input pre-processing | Application | Wraps the message as untrusted data and normalizes only what the design says (for example digit forms) |
| Request understanding | LLM (Prompt 1) | Returns structured JSON: intent, entities, missing information, flags |
| Output validation | Application | Rejects output that fails the schema or breaks the consistency rules |
| Tool selection | Application | Decides which tools to run from the validated request |
| Authorization and business rules | Application | Checks the customer's identity, order ownership, identifier format, and eligibility |
| Confirmation for sensitive actions | Application | Stores the pending action and runs it only after a valid confirmation |
| Response generation | LLM (Prompt 2) | Writes the reply using only verified tool results |
| Output checks | Application | Checks the reply for sensitive data and unsupported claims before sending |

## Trust boundaries

- **Customer messages** are untrusted data, never instructions.
- **Tool results** are the source of truth for order and product facts, but any text inside them (such as a product name or address) is also treated as data, not instructions.
- **Customer identity** comes from the authenticated session, never from claims in a message.
- **The model's output** is a proposal. The application validates it before acting.

## Sensitive actions

`cancel_order` and `update_shipping_address` need explicit confirmation tied to the specific action, order, and parameters. The application keeps the pending action. A confirmation expires and cannot be reused for a different action. Repeating a completed action returns the existing verified result instead of running it again.

## Not decided yet

- The order ID format.
- How the customer is authenticated.
- How multi-message conversations keep context.
- When a conversation is handed to a human agent.
- Which model runs each prompt.
- How long a pending confirmation stays valid.
