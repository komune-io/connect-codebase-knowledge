# Komune.io Connect Codebase

This repository contains the codebase and documentation for Komune.io Connect, a suite of microservices designed for managing files and identity/access.

The project is composed of two main modules:

* **connect-fs**: A service for managing files, acting as an interface to S3-compatible storage.

* **connect-im**: A service for Identity and Access Management (IAM), built on Keycloak.

## Modules

### connect-fs

`connect-fs` provides a service for managing **files**. It acts as an interface to an *S3-compatible storage* (like MinIO), allowing users to upload, download, list, and delete files using a structured **FilePath**. Optionally, it can track file history using *event sourcing* (via S2 Automate/SSM) for audit trails. Another optional feature is integrating with a *Knowledge Base* to vectorize file content, enabling semantic search and question-answering on documents. Configuration, including S3 credentials and optional features, is managed through `FsProperties`.

For detailed documentation on `connect-fs`, please refer to the [connect-fs Tutorial](connect-fs/index.md).

### connect-im

`connect-im` is a service for **Identity and Access Management (IAM)**. It allows managing *digital identities* (Users, API Keys) and *groups* (Organizations) within isolated environments called **Spaces** (which are Keycloak Realms). The system uses **Keycloak** as its core identity engine. Functionalities are exposed via **F2 Functions** (an API layer). Access control is handled through **Privilege Management** (defining Roles, Permissions, and Features) and enforced by a **Policy Enforcement** layer. **Configuration & Initialization Scripts** automate the setup of Spaces and their initial state.

For detailed documentation on `connect-im`, please refer to the [connect-im Tutorial](connect-im/index.md).

## License

This project is licensed under the Apache License, Version 2.0. See the [LICENSE](LICENSE) file for details.

##
