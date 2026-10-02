# NileCart AI Customer Support

> **Project status:** This is a simulated personal portfolio project, not a real client system or production deployment. NileCart is an invented e-commerce business, and no real customer data is used. It is a design and prompt-engineering prototype that has not been tested against a model yet, and nothing in this repository is a measured result.
>
> **About me:** I'm early in my prompt-engineering journey. I use AI tools to help draft the files, and I direct, review, and edit the work. My hands-on experience is in Egyptian Arabic data annotation and LLM evaluation.

An LLM-based customer support system designed for an Egyptian e-commerce environment, with a focus on prompt engineering, tool calling, validation, safety, and multilingual customer messages.

## Overview

The system is designed to handle customer messages in Egyptian Arabic, Modern Standard Arabic, English, Arabizi, and mixed-language messages.

The main idea is to separate language understanding from business operations. The LLM is responsible for understanding the customer's request, while validation rules and business tools are responsible for providing trusted information and executing sensitive actions.

## Problem

An AI customer support assistant needs to handle different types of customer messages while avoiding incorrect or unsupported information.

For example, the assistant should not guess an order status, product price, stock availability, refund status, delivery date, or product specifications.

The system also needs to handle cases such as missing order information, multiple requests in one message, unclear customer intent, prompt injection attempts, sensitive actions, and messages written in Arabic, English, Arabizi, or a mixture of languages.

The main challenge is to make the LLM useful for understanding customer requests without allowing it to make business decisions or provide information that has not been verified.

## System Approach

The design splits the work into separate stages:

1. Understand the customer message.
2. Extract the relevant information.
3. Validate the extracted data.
4. Decide whether a tool is required.
5. Retrieve or update information through approved tools.
6. Generate a final response using verified results.
7. Apply safety and output checks before returning the response.

This keeps the LLM focused on language understanding while deterministic components handle validation, authorization, and business operations.

## System Architecture

```text
Customer Message
        ↓
Input Pre-processing
        ↓
LLM Request Understanding
        ↓
Structured Output Validation
        ↓
Tool Selection and Orchestration
        ↓
Business Rules and Authorization
        ↓
Response Generation
        ↓
Safety and Output Checks
        ↓
Final Customer Response
```

The architecture separates language processing from business logic so that the LLM is not treated as the final authority for sensitive operations.

## Prompt Engineering

The design uses two LLM prompts with different responsibilities.

### Prompt 1: Request Understanding (draft v1)

The first prompt analyzes the customer's message and returns structured JSON instead of a natural-language answer. It extracts:

- Customer intent
- Order or product references and other entities
- Missing information
- Multiple requests in the same message
- Security-related signals
- Whether a tool is required
- Whether customer confirmation is required

Structured output makes the result easier to validate before any business operation is performed.

### Prompt 2: Response Generation (designed, not yet published)

The second prompt is meant to generate the final customer-facing response. Its text is not published in this repository yet. It receives verified tool results and treats them as the source of truth.

The model must not invent order status, product price, stock availability, refund status, delivery dates, product specifications, or return/cancellation eligibility. If information cannot be verified, the system should ask for the missing information, use the appropriate tool, or tell the customer the information cannot currently be verified.

## Example (illustrative)

This shows the *expected* output of Prompt 1 for one message. It is written by hand and is not a model output.

**Customer message:** `عايز أعرف طلبي وصل لفين؟`

```json
{
  "language": "Egyptian Arabic",
  "security_flags": [],
  "requests": [
    {
      "intent": "order_status",
      "entities": {
        "order_id": null,
        "product_id": null,
        "product_name": null,
        "size": null,
        "color": null,
        "address": null,
        "customer_reference": null,
        "refund_reference": null,
        "delivery_reference": null
      },
      "problem_summary": "Customer asks where their order is.",
      "customer_intent_text": null,
      "requires_tool": false,
      "requires_confirmation": false,
      "missing_info": ["order_id"],
      "needs_clarification": true,
      "clarification_question": "ممكن تبعتلي رقم الطلب؟"
    }
  ]
}
```

No tool is requested because the order ID is missing.

## Tool Calling

The design uses tools to retrieve trusted information and perform approved operations.

**Information retrieval:**

- `get_order(order_id)`
- `get_product(product_id)`
- `search_products(query)`
- `get_refund_status(order_id)`
- `check_return_eligibility(order_id)`

**Customer actions (change order data):**

- `cancel_order(order_id)`
- `update_shipping_address(order_id, address)`

Tools should only be called when the required parameters are available and the request has passed validation and authorization checks. Sensitive actions require explicit customer confirmation before execution.

## Safety and Validation

The design handles safety through multiple layers rather than relying on the prompt alone. It is designed to:

- validate structured model output before using it,
- check required parameters before calling tools,
- apply authorization and business rules to sensitive operations,
- treat prompt injection attempts as untrusted customer content that cannot override system rules,
- avoid exposing information belonging to another customer,
- keep sensitive information out of logs and responses.

None of this is implemented yet; see Known Limitations.

## Multilingual Handling

The design targets these input styles:

- Egyptian Arabic
- Modern Standard Arabic
- English
- Arabizi
- Arabic-English mixed messages
- Messages with spelling mistakes or informal wording

The system should preserve the customer's intended meaning rather than depend on exact wording. Ambiguous messages should not be guessed; the assistant should ask a focused clarification question.

## Design Decisions

- **Understanding is separate from execution.** The LLM only reads the message and returns structured data. Validation code and trusted tools decide what actually happens, so a wrong or manipulated model output cannot change an order by itself.
- **Cancelling an order or changing an address needs explicit confirmation.** These actions change real data and are hard to undo, so a single message from the customer is not treated as enough.
- **When information is missing, the model returns `null` and asks a question.** Guessing an order number or product could expose or change the wrong customer's data, so asking is the safer default.

## Evaluation

The plan is to evaluate the system on a fixed test set rather than on subjective review. Areas include intent classification, entity extraction, missing-information handling, hallucination prevention, prompt injection resistance, safe handling of sensitive actions, cross-customer data protection, tool-calling efficiency, multilingual robustness, and latency.

Critical safety failures are treated separately from general quality metrics. The accuracy targets in the evaluation plan are proposals, not results. No evaluation has been run yet.

## Red Team Cases

The repository includes 10 hand-written adversarial and edge cases:

1. Order status without an order ID
2. Prompt injection combined with a legitimate request
3. Multiple requests about different orders
4. Arabizi with an ambiguous product variant
5. Damaged product with missing order information
6. Cancellation request without confirmation
7. Request about another person's order
8. Malformed order ID
9. Repeated cancellation request
10. Payment card number in the message

These are written descriptions. They have not been run against a model, and they are not yet automated tests.

## Project Status

| Component | Status |
|---|---|
| Request-understanding prompt | Draft v1 |
| Response-generation prompt | Designed, not yet published |
| Output schema | Draft, needs tightening |
| Tool rules | Draft |
| Red-team cases (10) | Written, not yet run |
| Evaluation plan | Plan only, no results |
| Sample cases | In progress |
| Architecture notes | In progress |

## Known Limitations

- No code or running implementation yet.
- Prompts have not been tested against a model, so there are no measured results.
- The output schema is not strict enough yet to enforce every prompt rule.
- Test cases are written descriptions, not yet runnable test files.
- Egyptian Arabic and Arabizi coverage is still small.

## Next Steps

1. Tighten the schema and make the examples consistent with it.
2. Turn the test cases into a small runnable test file.
3. Run the prompts on a model and record the results, including failures.
4. Expand the Egyptian Arabic and Arabizi test messages.
5. Later: a small mock order database and validation code.

## Author

Abdelrhman Hisham: Egyptian Arabic data annotation and LLM evaluation.
