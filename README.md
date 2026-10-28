# AWS AI Support Agent with Amazon Bedrock AgentCore

An AI-powered customer support agent built with Amazon Bedrock AgentCore
and Strands Agents. It combines tool use, retrieval-augmented generation
(RAG), customer memory, loyalty calculations, and browser access to
answer common customer-support questions.

## Features

-   **Order tracking:** Retrieves order and customer information through
    the AgentCore Gateway.
-   **Refund processing:** Routes refund requests to the
    refund-processing Lambda tool.
-   **Knowledge-base retrieval (RAG):** Answers product and policy
    questions using the configured knowledge base.
-   **Long-term memory:** Stores and retrieves customer facts and
    preferences across separate sessions.
-   **Loyalty calculations:** Uses the AgentCore Code Interpreter to
    calculate point redemption and tier discounts.
-   **Browser access:** Uses the AgentCore Browser tool to retrieve
    information from public websites.
-   **Cloud deployment:** Runs as an Amazon Bedrock AgentCore Runtime
    application.

## Architecture

``` text
Customer prompt
      |
      v
Amazon Bedrock AgentCore Runtime
      |
      v
Strands support agent (main.py)
      |
      +--> AgentCore Gateway
      |       +--> Order tracking (API Gateway / Lambda)
      |       +--> Refund processing (Lambda)
      |
      +--> Knowledge Base (RAG)
      |
      +--> AgentCore Memory
      |
      +--> AgentCore Code Interpreter
      |
      +--> AgentCore Browser
      |
      v
Customer-support response
```

## Repository structure

``` text
aws-ai-support-agent/
├── starter/
│   ├── main.py
│   ├── product_catalog.txt
│   ├── pyproject.toml
│   ├── uv.lock
│   ├── setup_permissions.py
│   ├── .bedrock_agentcore.yaml   # Review before publishing
│   └── lambda/
│       ├── order_tracker.py
│       ├── refund_processor.py
│       └── lambda_schema
├── submission-evidence/
│   ├── 01-order-tracking.png
│   ├── 02-refund-processing.png
│   ├── 03-knowledge-base.png
│   ├── 04-memory-session-a.png
│   ├── 05-memory-session-b.png
│   ├── 06-loyalty-calculation.png
│   └── 07-browser-tool.png
├── README.md
├── reflection.md
└── .gitignore
```

## Technology stack

-   Python 3.13+
-   `uv` for dependency management
-   Amazon Bedrock AgentCore Runtime, Gateway, Memory, Code Interpreter,
    and Browser
-   Amazon Bedrock Knowledge Base
-   Strands Agents and MCP tools
-   AWS Lambda and API Gateway
-   Amazon Nova 2 Lite (`global.amazon.nova-2-lite-v1:0`)
-   CloudWatch Logs for runtime troubleshooting

## Prerequisites

1.  An AWS account with access to the required Amazon Bedrock and
    AgentCore services in `us-east-1`.
2.  AWS CLI v2 configured with credentials for your own account.
3.  Python 3.13+ and `uv`.
4.  The AgentCore Starter Toolkit CLI version used by this project.
5.  Access to the Amazon Nova 2 Lite inference profile.
6.  The AWS resources configured for this application: Gateway and
    targets, Knowledge Base, Memory resource, and deployed Runtime.

Service availability, IAM permissions, model access, and console
workflows can vary. Use your own resource IDs and follow current AWS
documentation.

## Setup and configuration

### 1. Get the project and install dependencies

``` bash
git clone https://github.com/DhruvParekh-star/aws-ai-support-agent.git
cd aws-ai-support-agent/starter
uv sync --python 3.13
```

### 2. Configure AWS resources

Create or configure the AWS resources required by the project:

-   Deploy the order-tracking and refund-processing Lambda functions.
-   Configure the API Gateway routes and AgentCore Gateway targets.
-   Configure the Knowledge Base with `product_catalog.txt` as its data
    source and wait for synchronization to finish.
-   Create and activate the AgentCore Memory resource with the memory
    strategies expected by the application.
-   Ensure the AgentCore Code Interpreter and Browser tools are
    available.
-   Configure the Runtime execution role with the minimum permissions
    required by the application.

The gateway target names in this project may be `order-tracker` and
`refund-processor` because target-name constraints can disallow
underscores. Ensure the configured names match the deployed resources
and tool names used by the application.

### 3. Set resource configuration

Review `main.py`, `.bedrock_agentcore.yaml`, and any deployment
configuration. Set the correct region and resource identifiers for your
own AWS account, including the Gateway URL, Knowledge Base ID, and
Memory ID wherever the code expects them.

Do not publish AWS credentials or secret keys. Prefer an AWS CLI profile
or an IAM role for authentication. If you share this repository
publicly, remove or parameterize account-specific values where
practical.

### 4. Configure permissions and deploy

From `starter/`, use the AgentCore CLI commands appropriate to the
toolkit version installed for this project. The typical workflow is:

``` bash
uv run python -m py_compile main.py
uv run agentcore deploy
uv run setup_permissions.py
```

If your Runtime has already been configured, do not blindly rerun
initial configuration commands. Review the existing configuration and
CLI help first. Wait for IAM permission changes to propagate before
testing the deployed agent.

## Functional testing

Run the following examples from `starter/`. Replace customer IDs,
session IDs, and resource-specific values as appropriate.

### 1. Order tracking

``` bash
uv run agentcore invoke '{"prompt":"Can you track order ORD-001?","customer_id":"CUST-123","session_id":"t1"}'
```

Verify that the response contains order details returned by the
order-tracking tool.

### 2. Refund processing

``` bash
uv run agentcore invoke '{"prompt":"I want to return my Kindle Paperwhite (ORD-002). Please initiate a refund.","customer_id":"CUST-123","session_id":"t2"}'
```

Verify the refund ID and status against the tool's actual response.
Refund status and processing-time messages may change as the workflow
progresses.

### 3. Knowledge-base retrieval

``` bash
uv run agentcore invoke '{"prompt":"What are the benefits of the Platinum loyalty tier?","customer_id":"CUST-123","session_id":"t3"}'
```

Check the response against the configured Knowledge Base content rather
than relying only on the model's wording.

### 4. Long-term memory

First session:

``` bash
uv run agentcore invoke '{"prompt":"Hi, I am Jane. I prefer concise responses.","customer_id":"CUST-123","session_id":"s-A"}'
```

Wait for memory extraction to complete (at least 30 seconds, or longer
if required by the service). Then use a different session ID:

``` bash
uv run agentcore invoke '{"prompt":"Do you remember my name and communication preference?","customer_id":"CUST-123","session_id":"s-B"}'
```

Verify that the agent recalls the name and preference accurately. Check
for stale or contradictory historical memories.

### 5. Loyalty discount calculation

``` bash
uv run agentcore invoke '{"prompt":"Use calculate_loyalty_discount to calculate a $150 standard order for a Gold member with 4250 points. Redeem points in multiples of 500, at 100 points per dollar, with redemption capped at 50% of the original order. Apply the 10% Gold discount after points redemption and report all calculation fields.","customer_id":"CUST-123","session_id":"t5"}'
```

Under these stated rules, the expected arithmetic is 4,000 points
redeemed (\$40), followed by a \$11 Gold discount on the \$110 subtotal,
for a \$99 final total. If the earning rule is 1% of the final total, 99
points are earned and the resulting balance is 349. Confirm the Code
Interpreter actually ran; do not treat a fallback result as proof that
the interpreter succeeded.

### 6. Browser tool

``` bash
uv run agentcore invoke '{"prompt":"Go to https://www.udacity.com and tell me the page title.","customer_id":"CUST-123","session_id":"t6"}'
```

Verify that the page title is retrieved by the browser tool. Website
content and availability may change.

## Troubleshooting

-   **Syntax errors:** Run `uv run python -m py_compile main.py` and
    correct indentation or syntax before deploying.
-   **Runtime startup or invocation errors:** Inspect the CloudWatch log
    group for your deployed AgentCore Runtime.
-   **Gateway tool errors:** Confirm target names, schemas, Lambda
    permissions, API Gateway routes, and the Gateway URL.
-   **Knowledge Base answers are missing or outdated:** Check the
    Knowledge Base ID, S3 data source, and synchronization status.
-   **Memory is inconsistent:** Verify the customer/actor identifier,
    namespaces, extraction delay, and whether stored memories conflict.
-   **Loyalty calculation uses a fallback:** Inspect logs for
    `Code Interpreter unavailable` and resolve the underlying error
    before accepting the result.
-   **Permission errors:** Check the Runtime execution role and resource
    policies. Avoid granting broad permissions as a quick fix.

    ## Security and cost considerations

-   Do not commit AWS access keys, tokens, `.env` files with secrets, or
    private keys.
-   Review `.bedrock_agentcore.yaml`, screenshots, logs, and deployment
    scripts before publishing.
-   Use least-privilege IAM permissions and secure authorization for any
    production Gateway.
-   The project may use a `NONE` Gateway authorizer in its learning
    configuration. Do not expose an unauthenticated Gateway for
    production or sensitive data.
-   Monitor AWS usage and delete resources you no longer need to avoid
    unexpected charges.


## Learning references

-   [Amazon Bedrock AgentCore
    documentation](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html)
-   [Strands Agents documentation](https://strandsagents.com/)
-   [Model Context Protocol
    Inspector](https://github.com/modelcontextprotocol/inspector)
-   [uv documentation](https://docs.astral.sh/uv/)
