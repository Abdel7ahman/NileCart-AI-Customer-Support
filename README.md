# NileCart AI Customer Support

An LLM-based customer support system designed for an Egyptian e-commerce environment, with a focus on prompt engineering, tool calling, validation, safety, and multilingual customer messages.

> **Portfolio Project:** NileCart is a simulated e-commerce environment created for this project. This repository does not use real customer data or production systems.

 ## Overview

NileCart AI Customer Support is a portfolio project focused on designing an AI customer support system for an Egyptian e-commerce business.

The system is designed to handle customer messages in Egyptian Arabic, Modern Standard Arabic, English, Arabizi, and mixed-language messages.

The main idea is to separate language understanding from business operations. The LLM is responsible for understanding the customer's request, while validation rules and business tools are responsible for providing trusted information and executing sensitive actions.

## Problem

An AI customer support assistant needs to handle different types of customer messages while avoiding incorrect or unsupported information.

For example, the assistant should not guess an order status, product price, stock availability, refund status, delivery date, or product specifications.

The system also needs to handle cases such as missing order information, multiple requests in one message, unclear customer intent, prompt injection attempts, sensitive actions, and messages written in Arabic, English, Arabizi, or a mixture of languages.

The main challenge is to make the LLM useful for understanding customer requests without allowing it to make business decisions or provide information that has not been verified.

## System Approach

The system is designed as a sequence of separate stages:

1. Understand the customer message.
2. Extract the relevant information.
3. Validate the extracted data.
4. Decide whether a tool is required.
5. Retrieve or update information through approved tools.
6. Generate a final response using verified results.
7. Apply safety and output checks before returning the response.

This approach keeps the LLM focused on language understanding while deterministic components handle validation, authorization, and business operations.

## Prompt Engineering

The system uses two main LLM prompts with different responsibilities.

### Prompt 1 — Request Understanding

The first prompt is responsible for analyzing the customer's message.

It extracts:

- Customer intent
- Order or product references
- Relevant entities
- Missing information
- Multiple requests in the same message
- Security-related signals
- Whether a tool is required
- Whether customer confirmation is required

The prompt returns structured JSON instead of a natural-language answer.

This makes the output easier to validate before any business operation is performed.

### Prompt 2 — Response Generation

The second prompt is responsible for generating the final customer-facing response.

It receives the relevant tool results and uses them as the source of truth.

The model must not invent:

- Order status
- Product price
- Stock availability
- Refund status
- Delivery dates
- Product specifications
- Return or cancellation eligibility

If the required information cannot be verified, the system should ask for the missing information, use the appropriate tool, or clearly tell the customer that the information cannot currently be verified.

## System Architecture

The overall flow is:

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

The architecture separates language processing from business logic so that the LLM is not treated as the final authority for sensitive operations.

## Tool Calling

The system uses tools to retrieve trusted information and perform approved operations.

### Information Retrieval

The following tools can be used to retrieve information:

- `get_order(order_id)`
- `get_product(product_id)`
- `search_products(query)`
- `get_refund_status(order_id)`
- `check_return_eligibility(order_id)`

### Customer Actions

The following tools can change order-related information:

- `cancel_order(order_id)`
- `update_shipping_address(order_id, address)`

Tools should only be called when the required parameters are available and the request has passed the relevant validation and authorization checks.

Sensitive actions require additional controls before execution.

## Safety and Validation

Safety is handled through multiple layers rather than relying on the prompt alone.

The system validates structured model output before using it, checks required parameters before calling tools, and applies authorization and business rules to sensitive operations.

For actions such as order cancellation or address updates, the system should require explicit customer confirmation before execution.

Prompt injection attempts are treated as untrusted customer content and must not override system instructions or security controls.

The system should also avoid exposing information belonging to another customer and should prevent sensitive information from being unnecessarily included in logs or responses.

## Multilingual Handling

The system is designed to handle customer messages written in different forms of Arabic and English.

Examples include:

- Egyptian Arabic
- Modern Standard Arabic
- English
- Arabizi
- Arabic-English mixed messages
- Messages containing spelling mistakes or informal wording

The system should preserve the customer's intended meaning rather than depending on exact wording.

Ambiguous messages should not be guessed. When important information is unclear or missing, the assistant should ask a focused clarification question.

## Evaluation

The system should be evaluated using a fixed test set rather than relying only on subjective review.

Key evaluation areas include:

- Intent classification
- Entity extraction
- Missing-information handling
- Hallucination prevention
- Prompt injection resistance
- Safe handling of sensitive actions
- Cross-customer data protection
- Tool-calling efficiency
- Multilingual robustness
- Response latency

Critical safety failures, such as unauthorized actions or customer-data leakage, should be treated separately from general response-quality metrics.

## Red Team Testing

The design includes adversarial and edge-case scenarios to test how the system behaves under difficult inputs.

Examples include:

1. A customer asks about an order without providing an order ID.
2. A single message contains requests about multiple orders.
3. A customer uses Arabizi or unclear product terminology.
4. A customer reports a damaged product and asks for a return.
5. A customer attempts to override the system instructions.
6. A customer asks for information about another person's order.
7. A malformed or unusually long order ID is provided.
8. The same action is requested repeatedly.
9. A customer includes sensitive payment information in a message.
10. A tool fails or returns incomplete information.

The purpose of these tests is to identify failure modes before the system is considered ready for a production implementation.

## Project Structure

The project is organized into separate components so that prompts, schemas, tool rules, test cases, and evaluation criteria can be developed and reviewed independently.

Planned structure:

- `prompts/` — LLM prompts and instructions
- `schemas/` — structured output schemas
- `tools/` — tool definitions and orchestration rules
- `red-team/` — adversarial test cases
- `evaluation/` — evaluation criteria and metrics
- `examples/` — sample customer conversations and expected behavior
- `architecture/` — system architecture documentation

## Project Status

**Current stage:** System design and prompt-engineering prototype.

The current version focuses on the system architecture, prompt design, structured outputs, tool orchestration, safety controls, red-team scenarios, and evaluation methodology.

The next development stage is to implement the design as a working prototype with a mock e-commerce database, real validation logic, tool calling, automated tests, and an API layer.
