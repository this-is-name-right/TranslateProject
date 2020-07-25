
[#]: collector: (lujun9972)
[#]: translator: (this-is-name-right)
[#]: reviewer: ( )
[#]: publisher: ( )
[#]: url: ( )
[#]: subject: (Detecting CPU steal time in guest virtual machines)
[#]: via: (https://opensource.com/article/20/1/cpu-steal-time)
[#]: author: (Jamie Fargen https://opensource.com/users/jamiefargen)

检测虚拟机窃取CPU的时间
======
你的虚拟机是否获得了它所需要的CPU时间？使用GNU的top命令来找出客户机的性能问题。
![and old computer and a new computer, representing migration to new software or hardware][1]

 CPU窃取时间被定义在[GNU **top**][2]命令手册中是这么定义的“时间被虚拟机的管理程序偷走了”。CPU窃取时间通常发生当虚拟机管理程序进程和客户机尝试同时利用同一虚拟机管理程序物理核心（pCPU）时。这导致客户机的的虚拟CPU（vCPU）可用的处理器时间更少，并且客户机的性能下降。

在今天虚拟的环境（随着公共云和私有云的采用，它们几乎已经普及）一个客户机实例可以在以下几种情况下遭遇能CPU窃取时间：

  * 虚拟机管理程序分配过多的资源和多个CPU利用率高的客户机的vCPU在同一pCPU上运行。
  * 客户机的vCPU及其模拟器线程固定在同一pCPU上，导致虚拟主机进程在处理I/O时从客户机vCPU窃取CPU时间。
  * 虚拟机管理程序进程，例如监视，记录和I/O进程同时使用客户机的vCPU也正在在使用的pCPU。



 通常情况下，请来调查应用程序或系统性能问题的系统工程师会发现，由于客户机窃取了CPU时间，导致系统性能下降。客户机的性能问题通常表现为低磁盘或网络I/O性能，网络数据包丢失以及其他应用程序性能异常。

 即使系统管理员正在观察客户机和虚拟机系统管理程序，也可能很难检查出由于CPU窃取时间而导致客户机实例性能下降的原因。 有一些导致难检测困难的原因。 首先，任何日常监控的日志文件都不会记录CPU窃取时间。 可以预期观察到虚拟机管理程序处于重负载下的窃取时间，但是在正常负载下的虚拟机管理程序上发生的窃取时间不好观察到。最后，管理员可能不知道可以使用GNU的top命令从客户机实例中观察到管理程序CPU争用。

幸运的是，GNU的top命令确实可以非常容易检测客户机实例上的CPU窃取时间。 窃取的时间会显示在top程序的输出的第三行的行末，并且以**%Cpu(s)**开头， 就像是下面的截图展示一样（用**st**来标识的值）。第一个例子展示了客户机只偷取了一点时间：

![Output of the top command showing low CPU steal time][3]

客户机的输出显示了客户机正在进行很低的CPU窃取时间只有0.2st。

这张截图展示了客户机正在进行很严重的CPU窃取时间：

![Output of the top command showing high CPU steal time][4]

客户机的输出表明，客户机正在进行很严重的CPU窃取时间，高达0.9st。

 这两个例子，这个压力工具是用四个进程同时执行的，这些进程消耗了客户机实例的所有四个vCPU。 在第一个例子，虚拟机管理程序相对比较闲，因此客户机窃取扽e时间仅为0.2。但是第二个例子中，压力工具在虚拟机管理程序上同时执行，其中有八个进程消耗了所有八个虚拟机管理程序的vCPU，这产生高达9.0CPU窃取时间。

 第二个例子中还有窃取时间的另一种迹象：压力进程无法消耗客户机100%的vCPUs;  它只能分别消耗99.3％，99.3％，86.4％和74.4％。 总的来说，这相当于客户机被盗取的40.3%。这是因为管理程序在客户机的vCPU进程正在使用的同一pCPUs上消耗周期。

### 使用top缓解性能不佳

此示例展示了如何在虚拟机管理程序上与客户机实例和其他进程进行超额分配资源，以及如何使用GNU的top命令来检测客户机上的CPU窃用时间百分比。

 在客户机中检测这种类型的性能下降很重要，这样可以减少造成系统和应用程序性能不佳的罪魁祸首。 在公共云中，唯一的解决方案可能是迁移实例或更改为具有保证的pCPU服务级别协议（SLA）的实例类型。 在私有云中，存在更多选择，但同样，最简单的方法可能是将实例迁移到利用率较低的虚拟机管理程序中。然而，如果许多客户机实例有着很高的CPU窃取时间，那么你将需要更改管理程序管理客户机的管理方式，以达到最好的客户机实例的性能SLA。

David Both解释了保持硬件凉爽的重要性，并分享了一些可以使用的Linux工具。

--------------------------------------------------------------------------------

via: https://opensource.com/article/20/1/cpu-steal-time

作者：[Jamie Fargen][a]
选题：[lujun9972][b]
译者：[this-is-name-right](https://github.com/译者ID)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]: https://opensource.com/users/jamiefargen
[b]: https://github.com/lujun9972
[1]: https://opensource.com/sites/default/files/styles/image-full-size/public/lead-images/migration_innovation_computer_software.png?itok=VCFLtd0q (and old computer and a new computer, representing migration to new software or hardware)
[2]: https://en.wikipedia.org/wiki/Top_(software)
[3]: https://opensource.com/sites/default/files/uploads/cpu-steal-time_1.png (Output of the top command showing low CPU steal time)
[4]: https://opensource.com/sites/default/files/uploads/cpu-steal-time_2.png (Output of the top command showing high CPU steal time)
