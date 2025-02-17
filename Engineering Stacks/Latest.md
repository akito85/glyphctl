## Devs

- Use **Pnpm** for as default package manager
- Use **Turbopack** for default development bundling
- Use **Webpack** for specific necessity only such as `module federation`
- Use **Prettier** for standardize code base
- Use **Semver** for standardize versioning number
- Use standardize **Commitizen** git commit
- Use **Jest** for unit automated testing
- Use **Typedocs** for code level documentation generator
---
## DevSecOps
- Use **envoy** for api gateway, with:
	- Compression **zstd**
	- Secure connection **HTTP 1, 2, 3 (If supported)** with SSL
	- **WAF** enabled
- Use **distroless** base image and **multi-stage**
- Use **standalone build** Nextjs
- Use **Kubernetes** with service account (if possible)
- Use **Cilium** or **Kalico** for **eBPF** container network interface
- Use **Jenkins** locally or use **Bitbucket pipelines and Vercel**
---
## DevDocs

- Use **Apidoc** for API documentation
- Use **Sparrow** to support API testing / documentation
- Use **Storybook** to support code documentation
---
## DevQA

#### Types of tests
- **Unit Testing** involves testing individual units (or blocks of code) in isolation. In React, a unit can be a single function, hook, or component. **(Jest)**
    - **Component Testing** is a more focused version of unit testing where the primary subject of the tests is React components. This may involve testing how components are rendered, their interaction with props, and their behavior in response to user events.
    - **Integration Testing** involves testing how multiple units work together. This can be a combination of components, hooks, and functions.
- **End-to-End (E2E) Testing** involves testing user flows in an environment that simulates real user scenarios, like the browser. This means testing specific tasks (e.g. signup flow) in a production-like environment. **(Cypress, Appium)**
- **Snapshot Testing** involves capturing the rendered output of a component and saving it to a snapshot file. When tests run, the current rendered output of the component is compared against the saved snapshot. Changes in the snapshot are used to indicate unexpected changes in behavior. **(Jest)**
