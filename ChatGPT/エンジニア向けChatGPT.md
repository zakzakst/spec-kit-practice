# エンジニア向けChatGPT

## ChatGPT for engineering teams
Engineers focus on building, fixing, and optimizing systems. They care about writing clean code, solving technical problems efficiently, and reducing bugs or downtime. ChatGPT can help by generating code snippets, explaining complex concepts, reviewing logic, writing documentation, and speeding up repetitive tasks so engineers can focus on shipping quality work faster.

---

## Research & benchmarking
Use ChatGPT to provide comparative analyses of technologies, tools, and industry practices.

### Evaluate cloud providers for migration
I’m an infrastructure engineer evaluating cloud migration options. Context: We’re moving from on-prem to the cloud for a fintech backend. Output: Compare AWS, GCP, and Azure for scalability, pricing, compliance, and developer tooling. Include citations.

### Research frameworks for real-time apps
I’m building a real-time collaboration tool. Context: We need low-latency and scalability. Output: Compare top frameworks (e.g., SignalR, Socket.io, WebRTC) with use cases, pros/cons, and current usage by other SaaS companies. Include sources.

### Benchmark observability tools
Benchmark the top observability tools. Context: We want to move from basic logging to full-stack monitoring. Output: Create a comparison table of features, pricing, integrations for Datadog, New Relic, Prometheus, and OpenTelemetry. Include sources.

### Analyze AI/ML trends in logistics
I’m researching AI/ML adoption in logistics systems. Context: Our company is considering integrating predictive routing. Output: A 5-paragraph summary on current trends, vendors, and implementation patterns. Include citations and links.

### Investigate compliance best practices
Research best practices for GDPR/CCPA compliance so we can help kick off discussions with our legal team. Context: Our app stores sensitive user data in the EU and US. Output: A compliance checklist with citations, sorted by regulation. Include links to documentation and regulations.

---

## Technical reviews & documentation
Use ChatGPT for engineering documentation, design reviews, and planning materials. Use canvas for real-time editing.

### Review system design doc
I’ve drafted a technical design document for [insert project or feature]. Review it for clarity, architectural soundness, and completeness. Highlight any missing considerations or questions reviewers may raise.

### Document internal API behavior
I need to document how this internal API works for other developers. Here’s the relevant code, schema, and usage examples: [insert materials]. Create clear documentation including endpoints, input/output formats, and expected behavior.

### Draft runbook for on-call engineers
I need to create a runbook for on-call engineers supporting [insert system]. Draft one that includes sections for system overview, common alerts, diagnostic steps, and escalation procedures.

### Draft onboarding guide for new hires
I need to write an onboarding guide for new engineers joining [insert team]. Create a draft with sections for required tools, access setup, codebase overview, and first tasks. Make it suitable for self-service onboarding.

### Write JIRA ticket from spec
Based on this engineering spec for [insert task or feature], write a JIRA ticket that includes the problem statement, context, goals, acceptance criteria, and technical notes for implementation.

---

## Debugging & optimization
ChatGPT can help diagnose, troubleshoot, and improve system performance and reliability.

### Debug failing system in production
A system in production is intermittently failing, and we’re struggling to isolate the root cause. Based on the following logs, metrics, and recent changes: [insert context], help identify the most likely causes and suggest next steps for mitigation.

### Analyze performance bottlenecks
Our service is experiencing latency and degraded performance during peak usage. Here are metrics, logs, and relevant traces: [insert context]. Help identify the bottlenecks and recommend specific optimizations.

### Analyze a data pipeline failure
A critical data pipeline failed in yesterday’s run. Here are the logs, data volume trends, and error outputs: [insert context]. Analyze what likely went wrong and provide recommendations to prevent recurrence.

### Suggest observability improvements
We currently use [insert tools] for monitoring [insert service]. Review our observability setup and suggest improvements across metrics, logging, alerting, and dashboards to improve issue detection and debugging.

### Brainstorm edge cases for testing
We’re preparing test cases for [insert feature/system]. Brainstorm potential edge cases and failure scenarios that may not be covered by standard testing, including unusual user inputs, system state changes, and concurrency issues.

---

## Data analysis & reporting
ChatGPT supports with analyzing datasets, identifying trends, and creating visual reports.

### Identify trends in product usage logs
Analyze this CSV of product usage logs. Context: We want to identify usage trends over time and across user segments. Output: Summary stats + line or bar charts highlighting key trends.

### Visualize system error rates over time
Plot error rates over time from this dataset. Context: It contains application logs from the last month. Output: A time-series chart with callouts for error spikes and a short interpretation.

### Analyze performance test results
Analyze this set of performance test results. Context: It compares two versions of our backend service. Output: Side-by-side comparison charts + text summary of improvements or regressions.

### Prioritize bugs based on impact
Analyze this bug report dataset. Context: Each row includes severity, frequency, and affected users. Output: A prioritized list of top bugs with charts showing frequency vs. severity.

### Summarize feedback from user surveys
Summarize this user feedback CSV. Context: It includes ratings and open text responses from a recent survey. Output: Key themes, sentiment scores, and charts showing distribution of ratings.

---

## System architecture & visualization
ChatGPT can support generating diagrams, flowcharts, and visual representations of complex systems and processes.

### Create a component diagram
I need to visualize the architecture of [insert system or service]. Generate a component diagram showing key services, data flows, and third-party integrations. Use clear labels and group components logically.

### Visualize system architecture
Create an image of the system architecture. Context: It’s a microservices-based e-commerce platform with services for payments, catalog, and user profiles. Output: Diagram with labeled services and data flow arrows.

### Explain CI/CD pipeline to stakeholders
Create an image that explains our CI/CD process. Context: This is for a presentation to business stakeholders. Output: Diagram showing dev → build → test → deploy steps with basic icons and short descriptions.

### Model data flow in ML pipeline
Create an image showing data flow in a machine learning pipeline. Context: We collect raw user data, clean it, train models, and serve predictions. Output: A labeled flowchart from raw data to inference.

### Diagram customer journey through app
Create a customer journey map through our mobile banking app. Context: Steps include onboarding, account linking, transactions, and support. Output: A visual flowchart with steps, screens, and decision points.
