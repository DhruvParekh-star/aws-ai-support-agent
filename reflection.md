# Project Reflection: Customer Support AI Agent with Amazon Bedrock AgentCore

## Design Decision

I designed the customer-support agent around modular tools so that each capability has a clear responsibility. The AgentCore Gateway connects the agent to order-tracking and refund-processing functions, while the Knowledge Base provides product and policy information through retrieval-augmented generation. AgentCore Memory supports customer context across sessions, the Code Interpreter handles loyalty calculations, and the Browser tool retrieves live website information. This design makes the agent more flexible and easier to test than a chatbot that relies only on generated text.

## Challenge and Solution

One major challenge was deploying the agent successfully to Amazon Bedrock AgentCore Runtime. Although the local development server responded correctly, the deployed runtime encountered an initialization error. I inspected the CloudWatch logs and found that the command-line argument parser was running without its required payload. I then corrected the Python entrypoint to use `app.run()` for the deployed application and redeployed the project. Afterward, the deployed agent responded successfully. This experience taught me the importance of checking runtime logs and distinguishing local execution from cloud deployment behavior.

## Production Improvements

For a production deployment, I would strengthen authentication and authorization, introduce human approval for sensitive refund operations, and apply input validation and safety controls. I would also add automated regression tests, structured logging, monitoring, and evaluation metrics to track tool accuracy and response quality. Finally, I would improve error handling, manage configuration securely, and establish a deployment and cleanup process that controls operational costs.

## Validation

I tested the deployed agent against the project's functional scenarios for order tracking, refund processing, knowledge-base retrieval, cross-session memory, loyalty calculations, and browser access. The final test results and supporting evidence are included with the submission.
