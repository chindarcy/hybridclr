
# huatuo

[![license](http://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)

<br/>

![icon](docs/images/logo.png)

<br/>

huatuo是一个**特性完整、零成本、高性能、低内存**的**近乎完美**的c#热更新方案。

huatuo为il2cpp之类的纯AOT CLR提供了interpreter模块，使基于它们的app不仅能在Android平台，也能在IOS、Consoles等限制了JIT的平台上高效地以**AOT+interpreter**混合模式执行，从底层彻底支持了热更新。

## 特性

- 特性完整。 近乎完整实现了[ECMA-335规范](https://www.ecma-international.org/publications-and-standards/standards/ecma-335/)，除了 下文中"限制和注意事项" 之外的特性都支持。
- 零学习和使用成本。 huatuo将纯AOT的运行时增强为完整的CLR运行时，使得热更新代码与AOT代码无缝工作。脚本类与AOT类在同一个运行时内，即使反射、多线程(volatile、ThreadStatic、Task、async)之类的代码都能够正常工作。不需要额外写任何特殊代码、没有代码生成，也没有什么特殊限制。
- 执行高效。实现了一个极其高效的寄存器解释器，所有指标都大幅优于其他热更新方案。[性能测试报告](docs/benchmark.md)
- 内存高效。 热更新脚本中定义的类跟普通c#类占用一样的内存空间，远优于其他热更新方案。[内存占用报告](docs/memory.md)
- 原生支持hotfix修复AOT部分代码。几乎不增加任何开发和运行开销。

## 工作原理

huatuo从mono的[Hybrid mode execution](https://developpaper.com/new-net-interpreter-mono-has-arrived/)技术中得到启发，为unity的il2cpp之类的AOT runtime额外提供了interpreter模块，将它们由纯AOT运行时改造为"AOT + Interpreter"混合运行方式，从底层彻底支持了热更新。

![icon](docs/images/architecture.png)

更具体地说，huatuo做了以下几点工作：

- 实现了一个高效的元数据(dll)解析库
- 改造了元数据管理模块，实现了元数据的动态注册
- 实现了一个IL指令集到自定义的寄存器指令集的compiler
- 实现了一个高效的寄存器解释器
- 额外提供大量的instinct函数，提升解释器性能
- 提供hotfix AOT的支持 （进行中）

## 文档

- [wiki](docs/home.md)
- [快速上手](docs/start_up.md)
- [FAQ](docs/FAQ.md)
- [常见错误](docs/common_errors.md)
- [最佳实践](docs/best_practices.md)
- [源码结构与跟踪调试](docs/source_inspect.md)
- [示例项目](https://github.com/focus-creative-games/huatuo_trial)
- [知乎专栏](https://www.zhihu.com/column/c_1489549396035870720)
- [==>致谢名单<==](docs/donate.md)
- 支持与联系
  - QQ群: 651188171（huatuo c#热更新 开发交流群）
  - 邮箱: taojingjian#gmail.com

## 稳定性状况

技术评估上目前稳定性处于Alpha版本与Beta版本之间。

- 完成了绝大多数指令的单元测试。只有calli、initblk等少数难以直接在c#里构造的指令未添加对应的单元测试。
- 对照c#规范完成各项语法测试。
- 测试了游戏常用库和框架的兼容性，兼容性良好。 参见[兼容性报告](docs/compatible.md)
- 可以正确地加载 [luban](https://github.com/focus-creative-games/luban) 配置
- 可以正确地运行 [2048](https://github.com/dgkanatsios/2048)这样的轻量小游戏
- 可以运行 [StarForce](https://github.com/EllanJiang/StarForce) 这样的小型游戏
- 正在帮助一些小中大型（其中有一个重度MMORPG项目）进行huatuo的迁移和测试工作。预计本月可以顺利运行MMORPG这样的大型游戏项目。

## 限制和注意事项

**不在限制事项中的特性都是huatuo支持**。请不要再问huatuo是否支持某个功能。

- 支持2019-2022全系列版本，但不是每个小版本都支持。具体请查看[现在支持的Unity版本](docs/support_versions.md)。
- 无法创建出普通AOT泛型(**delegate、Nullable、数组(包括多维)不受限制，热更新泛型也完全不受限**)的**热更新非枚举值类型**的实例化类型的实例。 例如不支持List&lt;HotUpdateValueType&gt;但支持List&lt;int&gt;、List&lt;HotUpdateClass&gt;和List&lt;HotUpdateEnum&gt;。 具体原因见[AOT泛型限制及原理介绍](docs/generic_limit.md)。这个会在7月份版本比较彻底地解决，此后不再有限制。
- 暂时不支持返回**自定义值类型**的默认async task，原生值类型如int及枚举及class类型不受限制。原因是编译器会为async默认生成 AsyncTaskMethodBuilder&lt;T&gt;的泛型类实例化，如果像ETask那样使用自定义Task及AsyncTaskMethodBuilder，则不受限制。原生async这个限制后续版本会解决。
- **注意使用link.xml或者代码引用的方式避免unity裁减代码。避免开发期能调用，发布后却找不到函数的错误**。我们后续会提供默认模板。
- 不支持delegate的BeginInvoke, EndInvoke。纯粹是觉得没必要实现。
- 热更新的MonoBehaviour通过AddComponent可以正常工作。如果要挂载到热更新资源上，还需要打包时略微做一点额外操作（文档正在补充），才能正常工作。
- 暂不支持增量式gc。由于时间紧凑，来不及仔细处理增量式gc的memory barrier细节。这个问题很快会解决。
- 暂时不支持打包后真机的c#源码调试，但能打印错误堆栈（只能精确到函数），也可以在Build出的Debug工程中跟踪调试。Editor下开发期调试使用平时的mono调试就可以了。

## RoadMap

huatuo虽然与il2cpp相关，但绝大多数核心代码独立于il2cpp，很容易移植（预计一个月）到其他不支持AOT+Interpreter的CLR平台。无论unity如何版本变迁，哪怕废弃了il2cpp改用.net 6+，huatuo会持续跟进，稳定地提供跨平台的CLR热更新服务，直至某天.net官方直接支持AOT+Interpreter，则huatuo完成其历史使命。

- 持续修复bug，让一个中大型游戏正常运行 (2022.4)
- 持续跟进unity的版本更新，支持更多的unity版本。查看[现在支持的Unity版本](docs/support_versions.md)
- 支持 hotfix AOT部分的bug (2022.6)
- 除去大多数普通AOT泛型类限制 (2022.6)
- 支持增量式gc。 (2022.6)
- 指令优化，编译后指令数减少到原来1/4-1/2，基础指令和大多数对象模型指令有100%-300%的性能提升。 (2022.6 出预览版本)
- 支持 extern函数 (2022.7)
- 其他内存和效率优化，以及代码重构
- **===支持godot引擎===**(2022.12)

## license

Huatuo is licensed under the [MIT](https://github.com/focus-creative-games/luban/blob/main/LICENSE.TXT) license