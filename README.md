# KHub

> Introducing another Kafka management system for microservices applications running on Kubernetes.

What Khub does:

- Centralize configuration and management of all kafka resources (i.e. topics and sink/source connectors...)
- Continuously check the cluster desired status and, if they diverges, sends alarms (standard logs, Prometheus)
- Provides all required producer/consumer configurations  via rest api call

KHub is easy to adopt:

- Runs inside the application as every other micro-service.
- Requires (almost) no configuration and less resources

## Motivation

Modern systems are running hundreds of micro-services and most of them are connected to many topics.

Both consumers and producers need to read from the same topic and someone needs to create that topic.

## Who is calling `newTopic`?

In most of the enterprise environment `auto.create.topics.enable = true` is not a workable choice. Engineers wants control over the topic configurations.

We don't want dependencies between consumers and producers.

To fix this a usual approach is: the `newTopic("topic-X")` call for the same topic needs to be present in both consumers and producers. This leads to code duplication and unclear responsibility for that action.

Another drawback is, at startup, every micro-service for all topics in its configuration requests a new topic to be created. This leads unnecessary requests and not clear ownership over the action.

## Configuration consistency

The configurations needs to be consistent. Consumers need to read the same configuration as the producers to be in sync.

Usual approach is to centralize the configuration (single configmap).

That means: all micro-service have access to the configurations of all topics in the system.

Basic metrics and alarms are delegated to external systems. This requires extra effort.

Internal micro-services typically use standard configurations shared across the components. KHub take advantage of this approach.

### What is changing using KHub:

- Topics and their configuration are managed as standard configurations. The system is consistent across the environments. Environment-specific configurations are implemented through standard mechanisms, such as Helm template overrides
- All configurations are centralized. Only one service create topics and configure connectors
- Micro-service are fetching the configurations from a single central system via rest api (KHub keep the service in hold if the topics are not available)
- KHub offer a mechanism to document and inspect topics.
- KHub keep track (via configuration requests) of producers and consumers for each topic
- Configure sources and sinks
- Continuously check the cluster and check the consistence with its current and desired status

### what is not KHub:

- A centralized UI/rest based system to manage topics
- A system to inspect the content/executing query/filters....

## Minimal dependencies

Khub has no dependencies with external systems. Everything you need is

- Configuration (configmap)
- A persistent volume