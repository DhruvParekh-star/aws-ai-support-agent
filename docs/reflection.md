# Project Reflection – AWS AI Customer Support Agent

## 1. Design Decisions

I designed the Customer Support Agent using a modular architecture built around Amazon Bedrock AgentCore. The application uses a Gateway to connect order-tracking and refund-processing tools, allowing the agent to retrieve order information and handle refund requests. Amazon Bedrock Knowledge Bases provides retrieval-augmented generation (RAG) for answering product and policy-related questions. AgentCore Memory enables persistent customer context across separate sessions. The loyalty discount calculation uses AgentCore Code Interpreter, while AgentCore Browser enables access to live web content.

This architecture separates individual capabilities into specialized components, making the application easier to maintain, test, and extend.

## 2. Challenges and Solutions

One significant challenge occurred during runtime initialization. The application encountered an error because the argument parser attempted to process command-line arguments without the required invocation payload. I investigated the CloudWatch runtime logs, identified the entry-point issue, and corrected the application initialization to use the appropriate BedrockAgentCoreApp execution flow.

I also worked through Gateway configuration and tool integration, including configuring API Gateway and Lambda targets and ensuring that the agent could discover and invoke the required tools.

## 3. Testing and Validation

I tested the major application capabilities through separate invocation scenarios. These included order tracking, refund processing, Knowledge Base retrieval, cross-session memory recall, loyalty discount calculation, and browser-based webpage access.

CloudWatch logs were used to inspect runtime execution, Gateway tool discovery, Memory retrieval, and Code Interpreter execution. The loyalty test returned the expected calculation values, and the Memory test demonstrated retrieval of previously stored customer information in a separate session.

## 4. Future Improvements

For production deployment, I would implement stronger authentication and authorization, input validation, and human approval for sensitive refund operations. I would also improve structured logging, automated regression testing, monitoring, and error handling. Additional improvements would include secure configuration management, independent validation of refund amounts, and resource-cost monitoring.

Overall, this project provided practical experience in building, integrating, testing, and evaluating an AI-powered customer support application using AWS services.