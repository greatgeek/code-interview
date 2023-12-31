# Docker核心技术相关

## 内核与文件系统

Linux操作系统包括两个主要部分：内核（Kernel）和文件系统。

1. **内核**：内核是操作系统的核心，控制计算机硬件资源并提供上层应用程序接口。例如，它负责调度并管理CPU的进程和任务、管理内存、硬盘、外设等硬件资源。此外，它还提供了系统调用接口，应用程序可以通过这些接口请求内核提供的服务。Linux内核是开源的，可以在不同的硬件平台上运行，如个人电脑、服务器、移动设备等。
2. **文件系统**：文件系统是操作系统用来控制如何将数据存储到磁盘上，以及如何检索这些数据的一种机制。Linux文件系统对所有存储设备使用一种统一的目录结构，即使在网络上或者插入新的设备时仍然保持相同的结构，为用户和程序提供了方便的数据管理方式。Linux系统支持许多不同类型的文件系统，如ext3、ext4、XFS、Btrfs等。

在Linux的设计中，所有设备都被视为文件，这种设计使得用户和程序可以采用统一且简单的方式来访问各种资源。这是通过设备文件和特殊文件来实现的，它们在文件系统中有自己的路径，但实际上是内核中设备驱动程序的一种接口。

所以，内核和文件系统是Linux操作系统中的关键部分，满足了用户对于硬件设备的控制需求和对文件的组织、管理需求。

## cgroup 与 chroot

**cgroup**：cgroup (控制组)是Linux内核的功能，用于限制并隔离一组进程对系统资源（如CPU，内存，磁盘I/O等）的使用。例如，Docker等容器技术就使用cgroup来限制和隔离容器的资源使用。通过cgroup，系统管理员可以精细地控制资源的分配，提高系统整体的稳定性和效率。

**chroot**：chroot是一种在Unix和类Unix操作系统中改变进程运行时根目录的操作。该操作的效果是建立一种虚拟环境，在该环境中进程无法访问实际文件系统中位于其根目录之外的文件和目录。它经常被应用在系统安全和系统隔离方面，例如，创建一种"沙盒"环境来运行不信任的程序。

总的来说，内核和文件系统是操作系统的基础组成部分，而cgroup和chroot是内核提供的一些可用于进程控制和资源隔离的高级功能。

### PID 为 1 有什么用？

在传统的 UNIX 系统中，PID 为 1 的进程是 init，地位非常特殊。他作为所有进程的父进程，有很多特权（比如：屏蔽信号等），另外，其还会检查所有进程的状态，我们知道，如果某个子进程脱离了父进程（父进程没有 wait 它），那么 init 就会负责回收资源并结束这个子进程。所以，要做到进程空间的隔离，首先要创建出 PID 为 1 的进程，最好就像是 chroot 那样，把子进程的 PID 在容器内变成 1。

## Docker 的 NAT 模式与 overlay 模式

Docker在容器网络中使用了多种网络模式，其中NAT（Network Address Translation）模式和Overlay模式是最常见的两种。以下是它们的详细描述和对比：

### 1. NAT模式（通常称为bridge模式）：

- **工作原理**：当容器启动时，它将连接到一个默认的网络桥（docker0）。每个新容器都会在这个桥上获得一个内部IP地址，然后Docker使用NAT将这些容器的流量转发到主机的外部IP地址。
  
- **用途**：NAT或bridge模式是Docker的默认网络设置，适合于单机上的容器通信。

- **限制**：
  - 因为它是基于主机的网络桥，所以跨多个主机的容器间通信可能会变得复杂。
  - 需要手动管理端口映射，以允许外部访问容器服务。

### 2. Overlay模式：

- **工作原理**：Overlay网络使用网络层2（数据链路层）的覆盖技术，允许在多个Docker宿主机上的容器之间创建一个虚拟网络，即使这些宿主机在物理上是分隔的。它通常使用VXLAN技术在主机之间封装网络流量。

- **用途**：当需要在跨多个宿主机上的容器之间进行通信时，overlay网络是首选。这在使用像Docker Swarm这样的集群管理工具时特别有用。

- **特点**：
  - 可扩展性：允许在跨多个主机、数据中心和云提供商的容器之间通信。
  - 解耦：Overlay网络与底层物理或云网络的细节解耦，为容器提供一致的网络接口。
  
- **限制**：
  - Overlay网络会引入一些额外的网络开销，因为它在宿主机之间封装和解封装网络流量。
  - 需要跨主机的数据存储和网络通信支持（例如，Swarm需要K/V存储如Consul、Etcd或ZooKeeper来维护网络状态）。

### 总结：
选择使用NAT模式还是Overlay模式取决于您的需求。对于单机上的简单Docker部署，NAT或bridge模式可能就足够了。但是，对于跨多个宿主机或数据中心的复杂部署，Overlay模式提供了一个强大和灵活的解决方案。

## Cgroup 相关

`cgroup`（control groups） 是一个 Linux 内核特性，用于限制、监控和隔离进程组的资源使用。下面是`cgroup`的工作原理和它是如何实现的概述：

实现细节：

`cgroup`在内核中实现，它为用户空间提供了一组文件系统接口（通常挂载在`/sys/fs/cgroup`）。当进程请求资源（如 CPU 时间、内存或 I/O）时，内核会检查进程所在的 `cgroup`,并基于该`cgroup`的配置决定是否授予请求。

1. 层次结构和控制器：
   * `cgroup`使用称为“控制器”或“子系统”的组件来对各种资源进行管理。常见的控制器有：`cpu`，`cpuacct`，`memory`,`blkio`，`net_cls`等。
   * `cgroup`在文件系统中（通常在`/sys/fs/cgroup/`）提供一个层次结构，该层次结构用于组织和管理进程组。
2. 进程管理：
   * 每个`cgroup`都包含一组进程。一个进程只能属于一个 `cgroup`，但同一进程可以在不同的控制器子系统中属于不同的`cgroup`。例如，进程`A`可能在`memory`子系统的 `cgroup X`中，而在 `cpu`子系统的`cgroup Y`中。
   * 当进程被创建时，它会继承其父进程所在的 `cgroup`。
3. 资源限制：
   * 通过控制器，`cgroup`可以限制、监控和隔离资源。例如，`memory`控制器可以用于设置内存使用的上限，而`cpu`控制器可以用于限制 CPU 使用时间。
   * 控制器提供的接口通常以文件的形式存在。例如，要设置特定`cgroup`的内存限制，可以写入其`memory.limit_in_bytes`文件。

### cgroup 控制器的原理

`cgroup`控制器的实现原理涉及对系统调用的拦截和处理、资源计量以及资源使用限制的强制实施。以下是各个常用控制器如何在内核中实现其功能的简要概述：

1. **cpu 控制器**：
   * 该控制器通过调整进程在 cpu 调度队列中的位置来影响其 CPU 访问。`cpu.shares`文件用于指定相对于其他进程组的 CPU 配额。
   * 还有`cpu.cfs_period_us`和`cpu.cfs_quota_us`可以设置时间周期和时间限制，从而限制在一个给定周期中进程可以使用的 CPU 时间。
2. **memory 控制器**:
   - 当进程尝试分配内存时，`cgroup` 会检查其所在的控制组是否超出了其限制。如果超出了，内存分配可能会失败，或者系统可能会触发 OOM  kill。
   - `memory.usage_in_bytes` 文件提供了当前控制组的内存使用情况，而 `memory.limit_in_bytes` 文件允许设置最大内存使用量。
   - 控制器还跟踪缓存和缓冲区的使用，并允许设置相关的限制。
3. **blkio 控制器**:
   - 该控制器限制块设备（通常是硬盘）的 I/O。它可以限制读/写的 IOPS 或吞吐量。
   - 通过调整请求的优先级或直接限制请求，它可以影响给定控制组的 I/O 访问。
4. **net_cls 和 net_prio 控制器**:
   - 这些控制器影响网络数据包的处理。`net_cls` 控制器可以为控制组中的包分配特定的类别标识符，而 `net_prio` 控制器可以设置不同网络接口的优先级。
5. **freezer 控制器**:
   - 它允许暂停和恢复控制组中的所有任务。当进程被 "冻结" 时，它们不会运行，直到它们被 "解冻"。
6. **devices 控制器**:
   - 该控制器允许或拒绝控制组的进程访问特定的设备。

这些控制器的工作方式是：当进程发出**系统调用请求资源**或执行某些操作时，`cgroup`子系统会介入并检查请求与该进程所在`cgroup`的配置之间是否存在冲突。例如，当进程请求内存分配时，`memory` 控制器会检查是否已经达到或超出 `memory.limit_in_bytes` 的值。

### 详细说明一下 cgroup中 cpu 控制器的原理

`cgroup`中的`cpu`控制器主要用于管理和限制进程组在 CPU 上的时间分配。它提供了一系列的机制来调整进程组的 CPU 使用。以下是`cpu`控制器的核心概念和实现原理：

1. cpu.shares
   * 这是一个相对权重，决定了一个`cgroup`与其他`cgroup`之间在竟争 CPU 时的权重。
   * 默认情况下，每个`cgroup`的`cpu.shares`值设置为 1024。如果一个`cgroup`的`cpu.shares`设置为 2048，与一个设置为 1024 的`cgroup`相比那么它将获得双倍的 CPU 时间。
   * 这是一个相对度量，不是绝对的 CPU 切片或百分比。实际的 CPU 时间分配取决于所有活动`cgroup`的`cpu.shares`的相对值。
2. cpu.cfs_period_us 和 cpu.cfs_quota_us：
   * 这两个参数一起定义了`cgroup`在特定时间段内可以使用的 CPU 时间的上限。
   * `cpu.cfs_period_us` 定义了一个时间周期，通常默认为100000us，即100ms。
   * `cpu.cfs_quota_us` 定义了在这个周期内 `cgroup` 可以使用的 CPU 时间。例如，如果它设置为50000us（或50ms），那么在每100ms的周期内，`cgroup` 最多只能使用50ms的 CPU 时间。
   * 如果 `cpu.cfs_quota_us` 设置为-1，表示该 `cgroup` 没有 CPU 使用的上限。
3. 实现原理：
   * 当进程在其分配的`cgroup`中运行并请求 CPU 时，Linux 调度器会考虑该`cgroup`的`cpu.shares`、`cpu.cfs_period_us`和`cpu.cfs_quota_us`的设置。
   * Linux 的调度器使用完全公平调度（CFS）算法，基于权重（由`cpu.shares`表示）和已使用的时间（与`cpu.cfs_quota_us `和`cpu.cfs_period_us`相关）来决定如何分配 CPU。
   * 如果`cgroup`超过了它的`cpu.cfs_quota_us`值， 则在该周期的其余时间内，该`cgroup`中的进程将不会被调度。

通过上述机制，`cpu` 控制器允许系统管理员有细粒度的控制权，决定不同的进程组如何分享系统的 CPU 资源。这对于多租户环境、容器化应用以及任何需要资源隔离的场景都非常有用。

### 如何在编程中使用 cgroup

在编程中使用 `cgroup` 通常涉及以下几个步骤：

1. **创建新的 cgroup**:
   通过文件系统接口，你可以在 `/sys/fs/cgroup/<controller_name>/` 下创建新的目录。每个新目录代表一个新的 `cgroup`。

   ```bash
   mkdir /sys/fs/cgroup/cpu/my_cgroup
   ```

2. **配置 cgroup 参数**:
   你可以通过写入值到特定的 `cgroup` 参数文件来配置它。例如，设置 `cpu.shares`：

   ```bash
   echo 512 > /sys/fs/cgroup/cpu/my_cgroup/cpu.shares
   ```

3. **添加进程到 cgroup**:
   你可以通过写入进程ID (PID) 到 `cgroup.procs` 文件来将进程添加到 `cgroup`。

   ```bash
   echo 12345 > /sys/fs/cgroup/cpu/my_cgroup/cgroup.procs
   ```

4. **从编程语言中操作**:
   多数编程语言都提供了文件操作的接口，你可以使用这些接口来读写 `cgroup` 相关的文件。

   例如，使用 Go 语言操作 `cgroup`：

   ```go
   package main
   
   import (
       "fmt"
       "io/ioutil"
   )
   
   func main() {
       // 设置 cpu.shares
       err := ioutil.WriteFile("/sys/fs/cgroup/cpu/my_cgroup/cpu.shares", []byte("512"), 0644)
       if err != nil {
           fmt.Println("Error:", err)
           return
       }
   
       // 添加进程到 cgroup
       err = ioutil.WriteFile("/sys/fs/cgroup/cpu/my_cgroup/cgroup.procs", []byte("12345"), 0644)
       if err != nil {
           fmt.Println("Error:", err)
           return
       }
   }
   ```

5. **使用库和工具**:
   多数现代编程语言都有库或工具可简化 `cgroup` 的操作。例如，Go 有 `libcontainer`，它提供了 `cgroup` 操作的高级API。

6. **清理**:
   当你不再需要一个 `cgroup` 时，确保移除与之相关的所有进程，并删除对应的目录：

   ```bash
   rmdir /sys/fs/cgroup/cpu/my_cgroup
   ```

7. **权限考虑**:
   操作 `cgroup` 通常需要特定的权限。如果你从应用中进行操作，确保应用有必要的权限，或者考虑使用如 `sudo` 等机制提升权限。

注意：虽然通过文件系统接口操作 `cgroup` 看起来很简单，但在生产环境中，通常更倾向于使用更高级的工具和库，如 Docker、Kubernetes 和 systemd，这些工具在内部管理 `cgroup`，并为用户提供了更友好的接口。

### cgroup 的子系统有多少种

1. **blkio**：这个子系统设置了每个 cgroup 的 I/O 访问块设备的限制。
2. **cpu**：管理 cgroup 中任务的 CPU 使用。
4. **cpuset**：为 cgroup 中的任务分配特定的 CPU 和内存节点。
5. **devices**：允许或拒绝 cgroup 中的任务访问设备。
6. **freezer**：暂停或继续执行 cgroup 中的任务。
7. **memory**：设置 cgroup 的内存使用限制，并自动生成 cgroup 中任务的内存资源使用统计信息。
8. **net_cls**：标记 cgroup 中任务生成的网络数据包，这样可以在 tc（Linux 的流量控制工具）中使用这些标记。
9. **perf_event**：允许 cgroup 中的任务使用 perf 工具监控。
10. **pids**：限制 cgroup 中的进程数量。
11. **rdma**：为 RDMA 设备上的 RDMA/InfiniBand 设备设置资源限制。
12. **hugetlb**：允许 cgroup 中的任务设置大页的使用限制。

这是 cgroup v1 的子系统列表。cgroup v2 对这些子系统进行了调整和合并，并引入了一些新特性。例如，`cpu` 和 `cpuacct` 在 cgroup v2 中被合并，`net_cls` 和 `net_prio` 也被合并。

## namespace 相关

### 什么是 Linux 命名空间（namespace）？

Namespace 是 linux 内核功能，允许隔离和更改进程看到的系统资源的视图，如 PID、网络、文件系统等。

### Namespace 的主要类型

在 Linux 中，Namespace 是用于隔离系统资源的关键技术。至今，Linux 内核提供了以下七种不同的 Namespaces：

1. Mount（mnt）：Mount Namespace 隔离了文件系统挂载点视图，允许每个命名空间内可以有不同的文件系统视图。
2. Process ID （pid）：PID Namespace 隔离了进程 ID，允许每个 Namespace 不同的进程有相同的 PID。
3. Network（net）：Network Namespace 隔离了网络设备、IP 地址、路由表等网络栈的状态，允许每个命名空间内的进程拥有独立的网络栈、网络设备和网络配置，使得每个命名空间内可以有独立的网络环境。
4. InterProcess Communication（ipc）：允许每个命名空间内的进程拥有独立的进程间通信机制，如System V IPC和POSIX消息队列。
5. **UTS**：UTS Namespace允许每个Namespace有自己的主机名和域名。
6. **User ID**（user）: User Namespace隔离了用户和用户组ID，它可以将一个Namespace的用户和组ID映射到别的Namespace的用户和用户组ID。
7. **Cgroup Namespace**：隔离 cgroup 根目录，使得在不同 Cgroup Namespace 中的进程看到的 cgroup 层次结构不同。
8. **Time Namespace**（自 Linux 5.6 起）：隔离时间命名空间，用于隔离时间相关属性，如系统时间、定时器等。

| 分类               | 系统调用参数  |
| ------------------ | ------------- |
| Mount namespaces   | CLONE_NEWNS   |
| UTS namespaces     | CLONE_NEWUTS  |
| IPC namespaces     | CLONE_NEWIPC  |
| PID namespaces     | CLONE_NEWPID  |
| Network namespaces | CLONE_NEWNET  |
| User namespaces    | CLONE_NEWUSER |
| Cgroup Namespace   |               |
| Time Namespace     |               |

### 为什么说命名空间对容器技术如 Docker 和 Kubernetes 是核心的？

命名空间为容器提供了隔离，确保容器内的进程看到的是其自己的局部视图，而不是主机的全局视图。这为容器的独立性和安全性提供了基础。命名空间为容器提供了资源的隔离，从而防止容器内的恶意进程影响到主机或其他容器。

### `PID`命名空间是如何工作的？

`PID` 命名空间隔离进程 ID 号。在新的 PID 命名空间中，进程可以有一个在外部不可见的 PID。

### `Network`命名空间有什么特性？

`Network`命名空间隔离了网络接口、路由表、iptables 规则等，确保容器有自己独立的网络栈。

`unshare`、`clone`和`nsenter`命令

`unshare`允许从 shell 创建新的命名空间，`clone`允许从编程接口创建命名空间，`nsenter`允许进入已存在的命名空间。

### 仅依赖命名空间来隔离进程的安全风险是什么？

仅依赖命名空间 (namespaces) 进行进程隔离可能会导致以下的安全风险：

1. **资源耗尽**: 
   - 命名空间本身并不限制资源使用。如果一个进程在其命名空间内消耗过多的资源（如 CPU、内存或磁盘 I/O），它可能会对主机或其他命名空间中的进程产生负面影响。
   - 例如，在没有 `cgroups` 的帮助下，一个容器进程可能会耗尽所有可用内存，导致其他进程或整个系统受到影响。

2. **容器逃逸**:
   - 命名空间仅仅为进程提供了一个隔离的视图，但进程仍然在相同的内核上运行。如果内核存在漏洞，恶意进程可能利用这些漏洞从其命名空间逃逸到主机或其他命名空间。
   - 这种逃逸可能允许攻击者获得对整个系统的控制，而不仅仅是他们的命名空间。

3. **难以限制特定系统调用**:
   - 某些系统调用，尤其是那些可能影响安全性的，可能需要在命名空间中受到限制。
   - 虽然命名空间可以限制进程看到的视图，但不会限制它们可以执行的系统调用。例如，`seccomp` 需要用来限制系统调用。

4. **权限提升**:
   - 在某些情况下，特定的命名空间可能允许权限提升。例如，在一个新的用户命名空间中，一个普通用户可能被视为“root”，尽管该命名空间外部的权限可能受到限制。
   - 如果与其他隔离机制（如 `capabilities`）不正确结合，这可能会导致安全隐患。

5. **网络攻击面**:
   - 即使进程在网络命名空间中，如果没有正确的防火墙和安全策略，它仍然可能面临网络攻击。

6. **文件系统安全性**:
   - 使用命名空间时，文件系统的挂载点和视图可以被隔离，但如果不使用如 `chroot` 或其他层技术，文件系统仍然可能受到风险。

总之，虽然命名空间为容器和其他轻量级虚拟化技术提供了重要的基础隔离，但仅仅依赖它们并不足以确保安全。通常，命名空间需要与 `cgroups`、`seccomp`、`capabilities`、`SELinux`、`AppArmor` 等其他技术结合使用，以实现更全面的隔离和安全性。

### namespace 的实现原理

Linux 命名空间 (namespace) 是一个强大的内核特性，它允许隔离和改变多个系统资源的视图，如进程 IDs、主机名、文件系统挂载点等。这些隔离的视图允许运行在命名空间内的进程认为它们是在一个独立的、清洁的环境中运行。

以下是命名空间的实现原理：

1. **数据结构**:
   - 在内核中，命名空间是通过结构体来表示的。每种类型的命名空间（例如 `pid`, `net`, `mnt` 等）都有其特定的结构体。
   - 每个进程都有一个指向其所属命名空间的指针，在 `task_struct` 结构中。

2. **隔离**:
   - 当进程请求一个系统资源（如获取其 PID 或查看文件系统），内核会基于该进程的命名空间来决定如何响应。
   - 例如，在 PID 命名空间中，内核可能会返回一个与全局 PID 不同的本地 PID。

3. **创建与进入**:
   - 可以使用特定的系统调用，如 `clone()` 和 `unshare()`, 来创建新的命名空间或使现有进程进入一个新的命名空间。
   - 进入新的命名空间通常涉及到更改进程的 `task_struct` 中的命名空间指针。

4. **命名空间嵌套与层次**:
   - 命名空间可以嵌套。例如，可以在一个 PID 命名空间内创建另一个 PID 命名空间。这在容器技术中非常有用，它允许为容器创建多层的隔离。
   - 每个命名空间都有一个父命名空间的引用，除了最顶层的命名空间，它们指向自己。

5. **生命周期与垃圾收集**:
   - 命名空间的生命周期与它们的引用计数相关。当没有任何进程属于一个特定的命名空间时，该命名空间被销毁。
   - 内核会跟踪哪些进程属于哪个命名空间，并确保不会过早地销毁任何命名空间。

6. **与其他系统组件的交互**:
   - 命名空间与其他内核组件紧密集成，如 `cgroups`。例如，`net` 命名空间会与网络堆栈一起工作，确保正确的网络隔离。

7. **文件系统表示**:
   - `/proc` 文件系统可以用来查询和修改命名空间。例如，`/proc/[pid]/ns/` 目录包含了一个进程的所有命名空间的引用。

总的来说，命名空间通过改变进程对系统资源的视图来提供隔离，而这些改变是在内核级别实现的，对进程是透明的。这使得命名空间非常适合容器技术，因为它们可以提供轻量级、高效的隔离，而无需像传统虚拟机那样完全模拟整个操作系统。
