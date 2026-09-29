# NileCart Tool Rules

This document defines when the AI customer support system may request a tool, what information is required, and which operations require additional validation or confirmation.

The AI must treat tool results as the source of truth for customer-specific and product-specific information.

The AI must never invent a tool result or assume that an operation succeeded without a successful tool response.

## Tool Selection Rules

A tool may be requested only when the customer's intent is clear enough to identify the required operation and all required parameters are available.

Before requesting a tool, the system should verify:

- The requested operation matches a supported tool.
- All required parameters are present and valid.
- The customer is authorized to access or modify the relevant information.
- Any required business rules have been satisfied.
- Any required confirmation has been obtained before a sensitive action.

If a required parameter is missing, do not request the tool. Ask the customer for the missing information instead.

Never invent, infer, or modify tool parameters just to make a tool call possible.

## Supported Tools

The system may use the following tools when their required conditions are satisfied.

### Order Tools

- `get_order(order_id)`
  - Retrieves order information using a valid order ID.
  - Use when the customer needs order status, delivery information, or other order-specific details.

- `get_refund_status(order_id)`
  - Retrieves the current refund status for an order.
  - Use only when a valid and authorized order ID is available.

- `check_return_eligibility(order_id)`
  - Checks whether an order is eligible for return.
  - Use when return eligibility needs to be verified before proceeding.

### Product Tools

- `get_product(product_id)`
  - Retrieves product information using a valid product ID.

- `search_products(query)`
  - Searches for products when the customer provides a product description or name but does not provide a product ID.

### Action Tools

- `cancel_order(order_id)`
  - Cancels an order when the operation is permitted by the system's business rules.
  - Requires explicit customer confirmation before execution.

- `update_shipping_address(order_id, address)`
  - Updates the shipping address for an order when the operation is permitted.
  - Requires explicit customer confirmation before execution.

  ## Sensitive Actions and Confirmation

Sensitive operations must not be executed based only on the model's interpretation of the customer's message.

The application layer must validate authorization, business rules, and confirmation state before executing a sensitive tool.

The following operations require explicit customer confirmation before execution:

- `cancel_order`
- `update_shipping_address`

Confirmation must be tied to the specific action, order, and parameters being confirmed.

For example, confirmation for cancelling order `12345` must not automatically authorize changing the shipping address of the same order.

A confirmation should not be reused for a different action or materially different parameters.

If confirmation has not been obtained, the system must not execute the sensitive tool.

## Tool Execution Safety

Tool calls must be treated as controlled application operations, not as direct extensions of the language model.

Before execution, the application should validate:

- Tool name and operation.
- Required parameter format.
- Customer authorization.
- Current business rules and operation eligibility.
- Confirmation state for sensitive actions.

If a tool returns an error, an empty result, or an unexpected response, do not infer the missing information.

The system should report that the information could not be verified or that the operation could not be completed.

Tool results must not be modified to make them appear more favorable or complete.

When multiple independent read-only tools are required, they may be executed in parallel.

When one operation depends on the result of another operation, execute them sequentially and use the verified result as the input for the next operation.

## Multiple Requests and Error Handling

A customer message may contain multiple independent requests.

Each request should be evaluated separately and only the tools required for those requests should be called.

Independent read-only requests may be processed in parallel when this does not create a dependency between them.

If one request fails, the system should not assume that the other requests also failed.

If a tool response is unavailable, incomplete, or inconsistent with the expected schema, the system must not guess the missing result.

The final response should clearly distinguish between information that was successfully verified and information that could not be verified.

The system should avoid duplicate tool calls for the same request and parameters unless a retry is required because of a temporary tool failure.

All tool execution should remain subject to the application's authorization, validation, business-rule, and safety controls.
