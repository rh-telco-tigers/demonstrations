# OpenShift Demonstrations
This repository is intended to be a public one-stop resource for various use-case based demonstrations. This will be broken up into various use-case "patterns". A lot of these guides will be based on OpenShift, but this will also include patterns for OpenStack and Ansible as well.

# Repository Structure
The repository will be broken up into two primary formats.

## Use Cases
Use cases will be one-and-done demonstrations for a given use case. These are quick formatted guides to demonstrate a very targeted action item. An example of could be something like "pod autoscaling". With a "pod autoscaling" demonstration, a use-case for scaling pods is documented along with instructions for implementing that functionality, finishing with sample output from the demonstrated use case.

## Progressive Guides
A "progressive guide" takes a collection of "use cases", and expands this for a full end-to-end implementation reference guide. A small example of this is user management. You may want to take several use case guides such as configuring an IDP integration, assigning users to groups, assigning those groups to roles (with associated role-bindings), removing the kube-admin password, and then demonstrating an end to end application deployment with multi-team engagement during an example SDLC. "Use cases" are narrow in focus, whereas a "progressive guide" tells a complete story.

## Contributing and Style Guides
For questions, please open an Issue. Pull requests will be reviewed and accepted, but please see our [contributing guidelines](./docs/CONTRIBUTING.md) (more to come soon).

## Guides

## Progressive Guides

Progressive guides are full, end-to-end demonstrations that target specific end-users or use cases.

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [N/A](guides/README.md)             | [Overview](guides/README.md)            |

## Installation Guides

Installation guides go above and beyond our official Red Hat documentation. Examples of this could be setting up Zero Touch Provisioning, or installation of upstream or new operators.

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [0.00](./000-installation/README.md)       | [Overview](./000-installation/README.md)       |

## Standard Kubernetes Functionality

These guides should be focused on demonstrating OpenShift and Kubernetes parity.

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [1.00](./100-platform/README.md)   | [Overview](./100-platform/README.md)         |

## Infrastructure Guides

Infrastructure guides will be focused on hardware-specific deployments, such as performance add-on operator, NVIDIA DPU (when used as an OpenShift endpoint), etc.

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [2.00](./200-infrastructure/README.md)          | [Overview](./200-infrastructure/README.md)        |

## Networking Guides

Unlike the infrastructure guides, networking is specifically focused on network-specific hardware like Intel 810/710 feature enablement, SR-IOV operator, or NVIDIA DPUs (when used for networking acceleration).

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [3.00](./300-network/README.md)        | [Overview](./300-network/README.md)      |

## Storage Guides

These guides will be focused on storage deployments, integration, and operation.

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [4.00](./400-storage/README.md)        | [Overview](./400-storage/README.md)      |

## Security Guides

Security guides will focus on many areas of security. This could be RHCOS security, ACS, ACM, native Kubernetes security features, or general OpenShift security enhancements.

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [5.00](./500-security/README.md)       | [Overview](./500-security/README.md)     |

## Self-Service Guides

Anything related to self-service within the OpenShift platform should be included in this section.

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [6.00](./600-self-service/README.md)   | [Overview](./600-self-service/README.md) |

## User Experience

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [7.00](./700-user/README.md)          | [Overview](./700-user/README.md)        |

## Developer Experience

Developer-focused use cases and overall developer experience.

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [8.00](./800-developer/README.md)    | [Overview](./800-developer/README.md)    |

## Logging, Alerting, and Monitoring

|             |  Section    | Title/Goal  |
| ----------- | ----------- | ----------- |
| :heavy_check_mark: | [9.00](./900-lma/README.md)           | [Overview](./900-lma/README.md)           |