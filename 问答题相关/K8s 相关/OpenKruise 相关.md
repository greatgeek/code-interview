# Open Kruise 相关

## 常见面试题

OpenKruise 是一款针对云原生应用的自动化工程管理平台，主要用于处理部署和运维等相关问题。因此，对于关于 OpenKruise 的面试题来说，问题通常涉及它的功能、使用场景、特性、和其他相关的技术知识。以下是一些可能的面试题：

1. OpenKruise 是什么？你可以谈谈 OpenKruise 的主要特性吗？

OpenKruise 是一个开源的 Kubernetes 自动化管理平台，它为 Kubernetes 提供了一系列的自动化能力，帮助使用者更好地管理其容器化的应用，包括应用的发布升级、扩缩容、QoS 等。

OpenKruise 的主要特性包括：

1. 高级应用部署能力：OpenKruise 提供了更强大的应用发布和滚动更新能力，我们可通过 CloneSet 或者 Advanced StatefulSet进行灵活的发布和滚动升级策略控制。

2. 自动化应用伸缩：OpenKruise 提供了自动化伸缩的能力，可根据用户设置的指标（例如CPU、内存等）对Pod等进行智能化的水平和垂直伸缩。

3. 灰度发布和流量调度：OpenKruise 支持灰度发布，能通过灰度发布的方式逐渐将新版本应用推向生产。

4. 应用质量管理：OpenKruise 提供了一些工具和机制来确保应用程序的可用性和服务质量。

5. 提供 Sidecar 的管理能力: OpenKruise 提供了 Sidecar 容器的生命周期管理，如注入、升级等。

6. 工作负载资源保护：OpenKruise 提供了对 Kubernetes 工作负载资源的保护机制，减少了运维操作对集群资源的影响。

1. 你可以描述一下在使用 OpenKruise 进行部署的过程中，一款应用的生命周期是如何的吗？

OpenKruise 在 Kubernetes 中引入了新的 APIs 和 控制器，以更好地管理应用的生命周期。以下是使用 OpenKruise 进行部署时，一款应用可能经历的生命周期：

1. 应用定义和创建：首先，你需要使用 OpenKruise 提供的 CRD (Custom Resource Definitions)，例如 CloneSet，来定义你的应用。这通常包括应用的 Docker 镜像，环境变量，副本数量等。定义好之后，你可以提交这个定义到 Kubernetes 集群，由 OpenKruise 控制器创建应用。

2. 应用部署：在创建应用的过程中，OpenKruise 控制器会确保按照你设定的部署策略（例如滚动更新策略，分区策略等）来部署应用。例如，如果你的应用有 10 个副本，你可以通过设定，让它们按照一定的顺序和速度被创建和启动，而不是全部同时创建。

3. 应用运行：当应用的所有副本部署完成后，应用进入运行阶段。在这个阶段，你的应用提供服务，并可能会需要处理各种运维操作，例如扩缩容，更新配置等。OpenKruise 提供了自动化伸缩，侧车容器管理等能力，来帮助你更好地管理运行中的应用。

4. 应用更新：当应用需要更新时，例如更新 Docker 镜像或者环境变量，你可以更新应用的定义，然后提交到 Kubernetes 集群。OpenKruise 控制器会根据你设定的滚动更新策略，逐步更新副本，而不影响提供的服务。

5. 应用析构：当应用需要停止时，你可以删除应用的定义。OpenKruise 控制器会逐步删除副本，然后清理和回收资源。

以上步骤都可以通过 Kubernete 的 kubectl 对接 OpenKruise 控制器完成，并可以通过 Kubernetes APIs 和 kubectl 命令来监控和管理应用的生命周期。

以上就是使用 OpenKruise 进行部署的应用的一般生命周期，实际生命周期可能会根据具体的部署策略和环境有所差异。

1. OpenKruise 如何实现云原生应用的集群部署？
2. 对于运行在 Kubernetes 集群中的应用如何做到滚动更新？OpenKruise 是如何支持的？

Kubernetes的原生Deployment对象已经支持滚动更新，但是OpenKruise提取了RollingUpdate的策略作为一个独立的策略对象供用户自定义，并且在此基础上增加了更多高级特性。

在Kubernetes中滚动更新主要是依靠控制Pod的创建和删除实现的。例如，一个服务由3个Pod提供，更新时，Kubernetes将逐个停止运行旧版本Pod，并启动新版本Pod。如果一个新版本Pod启动并运行无误，Kubernetes将继续停止第二个旧版本Pod，以此往复，直到所有旧版本Pod全部被新版本Pod替代。

相较于Kubernetes原生的RollingUpdate策略，OpenKruise提供了一个CloneSet控制器，它提供更加灵活的滚动升级、更新策略。如指定滚动升级Speed，支持指定每批升级版本的数量或百分比；支持按Pod副本数量动态调整批次更新数量；支持在更新过程中原地重建(replace)Pod，避免频繁的迁移和调试带来的资源浪费；支持指定并发更新的最大不可用数量等等。

此外，OpenKruise的CloneSet控制器在滚动升级过程中支持Pause的功能，可以随时暂停或者继续滚动更新，让整个过程可以更好地符合用户的需要。它还支持基于分区策略（inPlaceUpdateStrategy）进行灰度升级，能够分步骤、分批次地更新应用，很好地避免了版本一致性问题，减小了升级过程中的风险。

因此，对于在Kubernets集群中运行的应用而言，OpenKruise提供滚动更新的策略更加灵活高效，这不仅仅为应用的升级提供了新的可能，也为管理者带来更多便利。

1. OpenKruise 提供哪些类型的应用控制器，它们之间有何区别？

OpenKruise 提供了一组增强版的工作负载资源控制器，它们之间主要区别在于使用的场景和管理的对象不同，以下详细介绍：

1. CloneSet：CloneSet 是一个用于管理无状态或有状态应用的控制器。相较于 Deployment 和 StatefulSet 它在滚动更新，扩缩容等方面有更多丰富的特性。比如，它支持最大不可用（maxUnavailable）和分区（partition）的更新，支持原地重建一个 Pod，支持并行扩缩容，等等。

2. Advanced StatefulSet：这是对 Kubernetes 原生 StatefulSet 的增强版本，主要新增了无需 Headless Service，多维度按需缩容，下线优先缩容等功能。

3. BroadcastJob：BroadcastJob 是执行单次任务的控制器，它会在集群的所有节点或者某个特定节点上运行一个 Pod，直到该 Pod 结束。适用的场景包括数据预热，日志清理等。

4. UnitedDeployment：它提供一个跨多个不等的子集进行部署的能力。比如，一个应用可以按照设定的比例，分布在全局的网格环境中的多个可用区。

5. SidecarSet：用于管理 Sidecar 容器的控制器，可以实现 Sidecar 容器的生命周期管理，如注入，升级等。

6. WorkloadSpread：这是一个管理跨集群工作负载的控制器，可以将一个 Deployment／StatefulSet／CloneSet 之类的 workload 的 replica 在多个集群或者多个 namespace 中进行划分。

上述控制器用于不同的场景，可以根据实际的需求选择合适的控制器进行应用的管理。

1. 谈谈你理解的 OpenKruise 中的 SidecarSet 概念，它在什么场景下会被使用？

SidecarSet 是 OpenKruise 中一个重要的概念，它提供了对应用内 Sidecar 容器的管理。

在微服务架构中，Sidecar 模式非常常见。Sidecar 指的是和应用主业务容器（主容器）一起运行在一个 Pod 中的帮助容器。这个 Sidecar 容器通常用来执行那些和主业务逻辑无关，但对整体运行又很重要的任务，比如网络代理、日志收集、监控数据采集等。

然而，原生的 Kubernetes 对 Sidecar 容器的支持并不完善。比如，当我们需要对 Sidecar 容器进行升级或者配置修改的时候，可能需要重启整个 Pod，包括主容器，这可能会影响到应用的性能和可用性。

这就是 OpenKruise 提供 SidecarSet 的原因。SidecarSet 可以用来描述同一类 Sidecar 的各种属性，比如镜像版本、环境变量等。当我们需要对 Sidecar 进行升级或者修改的时候，只需要更改对应的 SidecarSet，由 OpenKruise 自动地完成对应 Pod 中 Sidecar 容器的更新，而不影响主容器的运行。

这个功能在很多场景下都非常有用。比如，我们在使用 Istio 服务网格的时候，常常需要把 Envoy 代理作为 Sidecar 注入到每个 Pod 中。当需要更新 Envoy 的版本或者配置的时候，使用 SidecarSet 就可以做到无缝更新，不影响应用的性能和可用性。

1. OpenKruise 中的 Advanced StatefulSet 与 Kubernetes 中的标准 StatefulSet 相比有什么区别？

Advanced StatefulSet 是 OpenKruise 中的一种自定义资源控制器，与 Kubernetes 中的标准 StatefulSet 相比，它增强了一些功能和特性。

以下是它们之间的主要区别：

1. 缩容优先级：在 Kubernetes 的 StatefulSet 中，其缩容操作默认是按逆创建顺序进行。也就是说，最新创建的 Pod 会首先被删除。然而，Advanced StatefulSet 具有更为灵活的缩容机制，它支持通过 annotations 对每个 Pod 的优先级进行定义，以此控制缩容的顺序。

2. 最大不可用率：在 Kubernetes 的 StatefulSet 中，升级操作默认是逐个更新 Pod。在很多场景下，为了保证服务的高可用，尤其是在集群规模较大的情况下，这种逐个更新的机制可能效率极低。相比之下，Advanced StatefulSet 支持定义最大不可用率，从而允许在更新期间有多个 Pod 同时不可用，可以显著提高更新的速度。

3. 无需 Headless Service：在 Kubernetes 的 StatefulSet 中，它依赖 Headless Service 来控制 Pod 的网络标识。然而在某些情况下，可能并不需要这样的网络标识。Advanced StatefulSet 可以不依赖 Headless Service，实现更为灵活的部署。

4. 灵活的扩容方式：与 Kubernetes 的 StatefulSet 相比，Advanced StatefulSet 提供了更加灵活的扩容机制。它支持同时扩容多个 Pod，并且可以给出同时扩容的数量。

以上还只是一部分功能和特性，Advanced StatefulSet 实际上还添加了很多其他强大的功能，以更好满足生产环境下的各种复杂需求。

1. OpenKruise 如何支持在 Kubernetes 集群中的应用的冗余和自我修复？

OpenKruise 在 Kubernetes 集群中的应用冗余和自我修复的支持主要通过其丰富的扩展应用控制器实现。比如，它的 CloneSet 模块以及 Advanced StatefulSet 模块都提供了强大的能力用于应对应用的故障和进行故障恢复。

OpenKruise 的 CloneSet 提供了对应用实例的复制和冗余能力。一旦某个 pod 发生错误或者无法正常工作，CloneSet 可以自动地创建一个新的 pod 来替代挂掉的 pod，实现自我修复的功能。

在运行有状态服务的时候，OpenKruise 的 Advanced StatefulSet 可以满足对数据一致性和网络标识的要求。对于有状态的应用，Advanced StatefulSet 提供了强大的数据备份和恢复能力，以应对可能的数据丢失和节点故障。

此外，OpenKruise 的其他模块如 WorkloadSpread、SidecarSet 等也在特定应用场景下提供了冗余和自我修复支持。比如 WorkloadSpread 模块提供了跨集群的应用副本调度，SidecarSet 模块支持在应用发送故障时动态替换修复 sidecar 容器等。

总的来说，OpenKruise 主要通过提供丰富的应用控制器，以达到在发生故障时快速修复和恢复应用，保障 Kubernetes 集群中的应用的稳定运行。

1. 能否描述一下在使用 OpenKruise 进行应用扩展时，会出现的一些问题？您是如何处理这些问题的？
2. 你是否有 OpenKruise 的实际使用经验？你在使用过程中解决过什么问题？