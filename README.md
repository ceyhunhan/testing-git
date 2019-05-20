# Zenoo Hub

Zenoo Hub provides a generic platform for defining and orchestrating Digital Onboarding (DO) processes. A process configuration resides within the Zenoo Hub and is used for the process orchestration.

At the heart of Zenoo Hub is a workflow engine. Each DO process is defined by a workflow using a custom DSL (Domain Specific Language). The workflow engine is used for orchestrating the corresponding DO process, i.e. a series of pages (routes), external calls, etc. This approach makes it possible to change a DO process configuration on-the-fly without rebuilding and redeploying the Zenoo Hub.

The Zenoo Hub is designed with the following goals in mind
- **Flexibility**, easy to customize and make changes on per-customer basis
- **Extensibility**, easy to add new functionality and integrate 3rd party services
- **Scalability**, easy to scale to handle increased traffic
- **Resilience**, responsive in case of failure
- **Transparency**, facilitate monitoring and troubleshooting

The Zenoo Hub consists of several components

- **Hub Backend**, contains the workflow engine, provides API for Hub Clients and Back-office, handles monitoring and logging

- **Hub Client**, renders DO pages using common best-practice components, uses Hub Backend API for processing Customer input and DO process orchestration (what to do next), a Hub Client is implemented as a web or mobile app

- **Hub Back-office**, an admin web UI, provides dashboard for Hub monitoring, let you filter and view DO process execution details (execution log, workflow definition)

More details can be found under specific sections below

- [Modules](docs/modules.md)
- [Workflow engine and DSL](docs/workflow.md)
- [Connectors](docs/connectors.md)
- [Testing](docs/testing.md)
- [API and security](docs/api.md)
- [Building and running](docs/build.md)
- [Customization, configuration and deployment](docs/customization.md)
