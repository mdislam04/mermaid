flowchart TD
    A[ Product Goal<br/>Business Requirements<br/>User Stories] --> B[ Planning Agent<br/>Task Decomposition<br/>Architecture Design]

    B --> C[ Coding Agent<br/>(Copilot / Cursor)<br/>Code Generation<br/>Refactoring]

    C --> D[ Test Agent<br/>Unit & Integration Tests<br/>Automated Validation]

    D -->|Test Failures| C
    D --> E[ Review Agent<br/>Code Quality<br/>Security & Compliance]

    E -->|Review Feedback| C
    E --> F[ Release Agent<br/>Versioning<br/>Changelog & Release Notes]

    F --> G[ Deploy Agent<br/>CI/CD Pipeline<br/>Monitoring & Rollback]

    G -->|Production Feedback| B
