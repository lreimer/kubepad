# Copilot instructions

## Build and tests

- Use JDK 21; the Gradle wrapper is Gradle 8.14.6.
- `./gradlew clean build` compiles the app, runs tests and license checks, and creates distributions in `build/distributions`.
- Run all default tests with `./gradlew test`.
- Run one test class with `./gradlew test --tests 'de.qaware.cloud.nativ.kpad.ButtonSpecs'`; narrow to one test method with `./gradlew test --tests 'de.qaware.cloud.nativ.kpad.ButtonSpecs.findMixerByCommandAndId'`.
- Tests named `*IT` are excluded from the default test task. They need external hardware or a cluster; run an individual one explicitly with `./gradlew test --tests 'de.qaware.cloud.nativ.kpad.KubernetesClientIT'` only when its external requirements are available.

## Architecture

Kubepad maps an 8×8 Novation Launchpad MK2 grid to workloads in a Kubernetes or OpenShift cluster. Each row identifies one app/deployment and each square represents one replica.

The application boots a CDI container in `KubepadApplication`. The `cluster.service` system property selects one backend: its CDI producer creates the backend client, and its `Cluster` implementation is enabled while the other providers are excluded. Backends publish `ClusterAppEvent`s when apps are added, scaled, or deleted. `ClusterNodeGrid` maintains the row/replica model, translates controller requests into backend scaling, and emits node lifecycle events. `LaunchpadController` observes button and node events to translate MIDI input into grid actions and grid state into Launchpad LEDs. Keep this event flow intact when changing backend, grid, or controller behavior.

App labels are part of the controller contract: `LAUNCHPAD_ENABLE` opts workloads in, `LAUNCHPAD_ROW` requests a free row from 0–7, and `LAUNCHPAD_COLOR` sets the row color. Backend settings live in `src/main/resources/cluster.properties`; backend selection is a JVM property (`-Dcluster.service=kubernetes` or `openshift`).

## Repository conventions

- Main application code is Kotlin under `src/main/kotlin/de/qaware/cloud/nativ/kpad`; test sources use Kotlin, Groovy/Spock, and Java. Keep hardware- or cluster-dependent tests named `*IT`.
- CDI-managed Kotlin classes and intercepted methods are generally `open`; use constructor injection and CDI events/qualifiers to connect components, following the surrounding package patterns.
- Backend implementations share the `Cluster` interface and report app changes through `ClusterAppEvent`; keep row assignment, labels, replicas, and event types consistent across providers.
- Java, Kotlin, and Groovy source files carry the repository MIT license header; the Gradle license task checks these files.
