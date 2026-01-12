# GASDocumentation
我对 Unreal Engine 5 的 GameplayAbilitySystem 插件（GAS）的理解，附带一个简单的多人示例项目。这不是官方文档，本项目及本人均与 Epic Games 无关联。我不保证信息的准确性。

本文档的目标是解释 GAS 中的主要概念和类，并根据我的经验提供一些额外的评论。社区用户之间有很多 GAS 的"隐性知识"，我旨在在这里分享我所有的经验。

示例项目和文档与 **Unreal Engine 5.3**（UE5）保持同步。本文档有针对旧版本 Unreal Engine 的分支，但不再维护，可能存在 bug 或过时信息。请使用与您的引擎版本匹配的分支。

[GASShooter](https://github.com/tranek/GASShooter) 是一个姊妹示例项目，演示了在多人 FPS/TPS 中使用 GAS 的高级技巧。

最好的文档始终是插件源代码。

<a name="table-of-contents"></a>
## 目录

> 1. [GameplayAbilitySystem 插件简介](#intro)
> 1. [示例项目](#sp)
> 1. [设置使用 GAS 的项目](#setup)
> 1. [概念](#concepts)  
>    4.1 [Ability System Component](#concepts-asc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.1.1 [网络同步模式](#concepts-asc-rm)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.1.2 [设置和初始化](#concepts-asc-setup)  
>    4.2 [Gameplay Tags](#concepts-gt)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.2.1 [响应 Gameplay Tags 的变化](#concepts-gt-change)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.2.2 [从插件 .ini 文件加载 Gameplay Tags](#concepts-gt-loadfromplugin)  
>    4.3 [Attributes](#concepts-a)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.1 [属性定义](#concepts-a-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.2 [BaseValue vs CurrentValue](#concepts-a-value)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.3 [元属性](#concepts-a-meta)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.4 [响应属性变化](#concepts-a-changes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.3.5 [派生属性](#concepts-a-derived)  
>    4.4 [Attribute Set](#concepts-as)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.1 [Attribute Set 定义](#concepts-as-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2 [Attribute Set 设计](#concepts-as-design)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.1 [具有单个属性的子组件](#concepts-as-design-subcomponents)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.2 [在运行时添加和移除 AttributeSets](#concepts-as-design-addremoveruntime)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3 [物品属性（武器弹药）](#concepts-as-design-itemattributes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.1 [物品上的普通 Float](#concepts-as-design-itemattributes-plainfloats)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.2 [物品上的 `AttributeSet`](#concepts-as-design-itemattributes-attributeset)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.2.3.3 [物品上的 `ASC`](#concepts-as-design-itemattributes-asc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.3 [定义属性](#concepts-as-attributes)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.4 [初始化属性](#concepts-as-init)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.5 [PreAttributeChange()](#concepts-as-preattributechange)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.6 [PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.4.7 [OnAttributeAggregatorCreated()](#concepts-as-onattributeaggregatorcreated)  
>    4.5 [Gameplay Effects](#concepts-ge)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.1 [Gameplay Effect 定义](#concepts-ge-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.2 [应用 Gameplay Effects](#concepts-ge-applying)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.3 [移除 Gameplay Effects](#concepts-ga-removing)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4 [Gameplay Effect 修饰符](#concepts-ge-mods)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4.1 [乘法和除法修饰符](#concepts-ge-mods-multiplydivide)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.4.2 [修饰符上的 Gameplay Tags](#concepts-ge-mods-gameplaytags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.5 [堆叠 Gameplay Effects](#concepts-ge-stacking)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.6 [授予的能力](#concepts-ge-ga)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.7 [Gameplay Effect Tags](#concepts-ge-tags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.8 [免疫](#concepts-ge-immunity)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.9 [Gameplay Effect Spec](#concepts-ge-spec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.9.1 [SetByCallers](#concepts-ge-spec-setbycaller)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.10 [Gameplay Effect Context](#concepts-ge-context)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.11 [修饰符幅度计算](#concepts-ge-mmc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12 [Gameplay Effect 执行计算](#concepts-ge-ec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1 [向执行计算发送数据](#concepts-ge-ec-senddata)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.1 [SetByCaller](#concepts-ge-ec-senddata-setbycaller)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.2 [备份数据属性计算修饰符](#concepts-ge-ec-senddata-backingdataattribute)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.3 [备份数据临时变量计算修饰符](#concepts-ge-ec-senddata-backingdatatempvariable)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.12.1.4 [Gameplay Effect Context](#concepts-ge-ec-senddata-effectcontext)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.13 [自定义应用要求](#concepts-ge-car)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.14 [消耗 Gameplay Effect](#concepts-ge-cost)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15 [冷却 Gameplay Effect](#concepts-ge-cooldown)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.1 [获取冷却 Gameplay Effect 的剩余时间](#concepts-ge-cooldown-tr)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.2 [监听冷却的开始和结束](#concepts-ge-cooldown-listen)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.15.3 [预测冷却](#concepts-ge-cooldown-prediction)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.16 [更改活动 Gameplay Effect 的持续时间](#concepts-ge-duration)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.17 [在运行时创建动态 Gameplay Effects](#concepts-ge-dynamic)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.5.18 [Gameplay Effect 容器](#concepts-ge-containers)  
>    4.6 [Gameplay Abilities](#concepts-ga)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1 [Gameplay Ability 定义](#concepts-ga-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.1 [网络同步策略](#concepts-ga-definition-reppolicy)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.2 [服务器尊重远程能力取消](#concepts-ga-definition-remotecancel)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.1.3 [直接复制输入](#concepts-ga-definition-repinputdirectly)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.2 [将输入绑定到 ASC](#concepts-ga-input)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.2.1 [绑定到输入而不激活能力](#concepts-ga-input-noactivate)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.3 [授予能力](#concepts-ga-granting)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4 [激活能力](#concepts-ga-activating)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4.1 [被动能力](#concepts-ga-activating-passive)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.4.2 [激活失败标签](#concepts-ga-activating-failedtags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.5 [取消能力](#concepts-ga-cancelabilities)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.6 [获取活动能力](#concepts-ga-definition-activeability)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.7 [实例化策略](#concepts-ga-instancing)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.8 [网络执行策略](#concepts-ga-net)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.9 [能力标签](#concepts-ga-tags)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.10 [Gameplay Ability Spec](#concepts-ga-spec)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.11 [向能力传递数据](#concepts-ga-data)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.12 [能力消耗和冷却](#concepts-ga-commit)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.13 [升级能力](#concepts-ga-leveling)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.14 [能力集](#concepts-ga-sets)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.15 [能力批处理](#concepts-ga-batching)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.6.16 [网络安全策略](#concepts-ga-netsecuritypolicy)  
>    4.7 [Ability Tasks](#concepts-at)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.1 [Ability Task 定义](#concepts-at-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.2 [自定义 Ability Tasks](#concepts-at-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.3 [使用 Ability Tasks](#concepts-at-using)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.7.4 [Root Motion Source Ability Tasks](#concepts-at-rms)  
>    4.8 [Gameplay Cues](#concepts-gc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.1 [Gameplay Cue 定义](#concepts-gc-definition)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.2 [触发 Gameplay Cues](#concepts-gc-trigger)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.3 [本地 Gameplay Cues](#concepts-gc-local)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.4 [Gameplay Cue 参数](#concepts-gc-parameters)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.5 [Gameplay Cue Manager](#concepts-gc-manager)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.6 [防止 Gameplay Cues 触发](#concepts-gc-prevention)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7 [Gameplay Cue 批处理](#concepts-gc-batching)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7.1 [手动 RPC](#concepts-gc-batching-manualrpc)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.7.2 [一个 GE 上的多个 GC](#concepts-gc-batching-gcsonge)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.8 [Gameplay Cue 事件](#concepts-gc-events)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.8.9 [Gameplay Cue 可靠性](#concepts-gc-reliability)  
>    4.9 [Ability System Globals](#concepts-asg)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.9.1 [InitGlobalData()](#concepts-asg-initglobaldata)  
>    4.10 [预测](#concepts-p)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.1 [预测键](#concepts-p-key)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.2 [在能力中创建新的预测窗口](#concepts-p-windows)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.3 [预测性生成 Actor](#concepts-p-spawn)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.4 [GAS 中预测的未来](#concepts-p-future)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.10.5 [网络预测插件](#concepts-p-npp)  
>    4.11 [瞄准](#concepts-targeting)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.1 [目标数据](#concepts-targeting-data)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.2 [目标 Actor](#concepts-targeting-actors)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.3 [目标数据过滤器](#concepts-target-data-filters)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.4 [Gameplay Ability World Reticles](#concepts-targeting-reticles)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4.11.5 [Gameplay Effect 容器瞄准](#concepts-targeting-containers)  
> 1. [常用的能力和效果实现](#cae)  
>    5.1 [眩晕](#cae-stun)  
>    5.2 [冲刺](#cae-sprint)  
>    5.3 [瞄准下望](#cae-ads)  
>    5.4 [生命偷取](#cae-ls)  
>    5.5 [在客户端和服务器上生成随机数](#cae-random)  
>    5.6 [暴击](#cae-crit)  
>    5.7 [非堆叠 Gameplay Effects 但只有最大幅度实际影响目标](#cae-nonstackingge)  
>    5.8 [游戏暂停时生成目标数据](#cae-paused)  
>    5.9 [单按钮交互系统](#cae-onebuttoninteractionsystem)  
> 1. [调试 GAS](#debugging)  
>    6.1 [showdebug abilitysystem](#debugging-sd)  
>    6.2 [Gameplay Debugger](#debugging-gd)  
>    6.3 [GAS 日志记录](#debugging-log)  
> 1. [优化](#optimizations)  
>    7.1 [能力批处理](#optimizations-abilitybatching)  
>    7.2 [Gameplay Cue 批处理](#optimizations-gameplaycuebatching)  
>    7.3 [AbilitySystemComponent 网络同步模式](#optimizations-ascreplicationmode)  
>    7.4 [属性代理网络同步](#optimizations-attributeproxyreplication)  
>    7.5 [ASC 延迟加载](#optimizations-asclazyloading)  
> 1. [生活质量建议](#qol)  
>    8.1 [Gameplay Effect 容器](#qol-gameplayeffectcontainers)  
>    8.2 [用于绑定到 ASC 委托的蓝图 AsyncTasks](#qol-asynctasksascdelegates)  
> 1. [故障排除](#troubleshooting)  
>    9.1 [`LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`](#troubleshooting-notlocal)  
>    9.2 [`ScriptStructCache` 错误](#troubleshooting-scriptstructcache)  
>    9.3 [动画蒙太奇未复制到客户端](#troubleshooting-replicatinganimmontages)  
>    9.4 [复制蓝图 Actor 将 AttributeSets 设置为 nullptr](#troubleshooting-duplicatingblueprintactors)  
>    9.5 [未解析的外部符号 UEPushModelPrivate::MarkPropertyDirty(int,int)](#troubleshooting-unresolvedexternalsymbolmarkpropertydirty)  
>    9.6 [枚举名称现在由路径名表示](#troubleshooting-enumnamesarenowpathnames)  
> 1. [常用 GAS 缩写](#acronyms)  
> 1. [其他资源](#resources)  
>    11.1 [与 Epic Games 的 Dave Ratti 的问答](#resources-daveratti)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11.1.1 [社区问题 1](#resources-daveratti-community1)  
>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;11.1.2 [社区问题 2](#resources-daveratti-community2)  
> 1. [GAS 更新日志](#changelog)  
>    * [5.3](#changelog-5.3)  
>    * [5.2](#changelog-5.2)  
>    * [5.1](#changelog-5.1)  
>    * [5.0](#changelog-5.0)  
>    * [4.27](#changelog-4.27)  
>    * [4.26](#changelog-4.26)  
>    * [4.25.1](#changelog-4.25.1)  
>    * [4.25](#changelog-4.25)  
>    * [4.24](#changelog-4.24)  

<a name="intro"></a>
## 1. GameplayAbilitySystem 插件简介
根据[官方文档](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)：
>Gameplay Ability System 是一个高度灵活的框架，用于构建类似于 RPG 或 MOBA 游戏中的能力和属性。您可以为游戏角色构建主动技能或被动能力、由这些操作产生的可以累积或消耗各种属性的状态效果、实现用于调节这些操作使用的"冷却"计时器或资源消耗、改变能力级别及其在每个级别下的效果、激活粒子或声音效果等。简而言之，该系统可以帮助您设计、实现并高效地在网络中实现游戏内技能，从简单的跳跃到任何现代 RPG 或 MOBA 游戏中您最喜欢的角色技能集一样复杂的技能。

GameplayAbilitySystem 插件由 Epic Games 开发并随 Unreal Engine 一起提供。该系统已在 Paragon 和 Fortnite 等AAA级商业游戏中经过了实战测试。

该插件为单人和多人游戏提供了开箱即用的解决方案，用于：
* 实现基于等级的角色技能或能力，支持可选的消耗和冷却（[GameplayAbilities](#concepts-ga)）
* 操作属于 Actor 的数值属性（[Attributes](#concepts-a)）
* 向 Actor 应用状态效果（[GameplayEffects](#concepts-ge)）
* 向 Actor 应用`GameplayTags`（[GameplayTags](#concepts-gt)）
* 生成视觉或声音效果（[GameplayCues](#concepts-gc)）
* 上述所有内容的网络同步

在多人游戏中，GAS 支持以下内容的[客户端预测](#concepts-p)：
* 技能激活
* 播放动画蒙太奇
* `Attributes` 的更改
* 应用 `GameplayTags`
* 生成 `GameplayCues`
* 通过连接到 `CharacterMovementComponent` 的 `RootMotionSource` 函数进行移动。

**GAS 必须在 C++ 中设置**，但设计师可以在蓝图中创建 `GameplayAbilities` 和 `GameplayEffects`。

GAS 目前的局限性：
* `GameplayEffect` 延迟协调（无法预测技能冷却，导致与低延迟玩家相比，高延迟玩家对低冷却技能的发射率较低）。
* 无法预测 `GameplayEffects` 的移除。但是，我们可以预测添加具有相反效果的 `GameplayEffects`，从而有效地移除它们。这并不总是合适或可行的，仍然是一个问题。
* 缺乏样板模板、多人游戏示例和文档。希望本文档能对此有所帮助！

**[⬆ 返回顶部](#table-of-contents)**

<a name="sp"></a>
## 2. 示例项目
本文档包含一个多人第三人称射击示例项目，面向不熟悉 GameplayAbilitySystem 插件但熟悉 Unreal Engine 的用户。期望用户已了解 C++、蓝图、UMG、网络同步以及其他 UE 中级主题。该项目提供了一个示例，展示如何设置一个基本的第三人称射击多人游戏项目，其中 `AbilitySystemComponent`（`ASC`）位于 `PlayerState` 类上用于玩家/AI 控制的英雄，而 `ASC` 位于 `Character` 类上用于 AI 控制的小兵。

该项目旨在保持简单，同时展示 GAS 基础知识并演示一些常见请求的能力，代码注释详细。由于侧重初学者，该项目不展示高级主题，如[预测投弹物](#concepts-p-spawn)。

演示的概念：
* `PlayerState` vs `Character` 上的 `ASC`
* 网络同步的 `Attributes`
* 网络同步的动画蒙太奇
* `GameplayTags`
* 在 `GameplayAbilities` 内部和外部应用和移除 `GameplayEffects`
* 应用由护甲减免的伤害以改变角色的生命值
* `GameplayEffectExecutionCalculations`
* 眩晕效果
* 死亡和重生
* 从技能在服务器上生成 Actor（投弹物）
* 通过瞄准下望和冲刺预测性地改变本地玩家的速度
* 持续消耗体力进行冲刺
* 使用法力施放技能
* 被动技能
* 堆叠 `GameplayEffects`
* 瞄准 Actor
* 在蓝图中创建的 `GameplayAbilities`
* 在 C++ 中创建的 `GameplayAbilities`
* 每个 `Actor` 实例化的 `GameplayAbilities`
* 非实例化的 `GameplayAbilities`（跳跃）
* 静态 `GameplayCues`（开枪投弹物撞击粒子效果）
* Actor `GameplayCues`（冲刺和眩晕粒子效果）

英雄类具有以下技能：

| 技能                    | 输入绑定          | 预测  | C++ / 蓝图 | 说明                                                                                                                                                                  |
| -------------------------- | ------------------- | ---------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 跳跃                       | 空格键           | 是        | C++             | 使英雄跳跃。                                                                                                                                                         |
| 开枪                        | 鼠标左键   | 否         | C++             | 从英雄的枪发射投弹物。动画已预测，但投弹物未预测。                                                                                |
| 瞄准下望            | 鼠标右键  | 是        | 蓝图       | 按住按钮时，英雄会走得更慢，相机会放大以便用枪进行更精确的射击。                                                    |
| 冲刺                     | 左 Shift          | 是        | 蓝图       | 按住按钮时，英雄会跑得更快，消耗体力。                                                                                                         |
| 前冲               | Q                   | 是        | 蓝图       | 英雄向前冲刺，消耗体力。                                                                                                                              |
| 被动护甲堆叠       | 被动             | 否         | 蓝图       | 每 4 秒英雄获得一层护甲，最多 4 层。受到伤害会移除一层护甲。                                                    |
| 陨石                     | R                   | 否         | 蓝图       | 玩家瞄准一个位置向敌人投下陨石，造成伤害并使其眩晕。瞄准已预测，但生成陨石未预测。                     |

`GameplayAbilities` 是在 C++ 还是蓝图中创建并不重要。这里混合使用两者，以展示如何在每种语言中创建它们。

小兵没有任何预定义的 `GameplayAbilities`。红色小兵有更高的生命值回复，而蓝色小兵有更高的初始生命值。

关于 `GameplayAbility` 命名，我使用后缀 `_BP` 表示 `GameplayAbility` 的逻辑是在蓝图中创建的。缺少后缀意味着逻辑是在 C++ 中创建的。

**蓝图资源命名前缀**

| 前缀      | 资源类型          |
| ----------- | ------------------- |
| GA_         | GameplayAbility     |
| GC_         | GameplayCue         |
| GE_         | GameplayEffect      |

**[⬆ 返回顶部](#table-of-contents)**

<a name="setup"></a>
## 3. 设置使用 GAS 的项目
设置使用 GAS 的项目的基本步骤：
1. 在编辑器中启用 GameplayAbilitySystem 插件
1. 编辑 `YourProjectName.Build.cs`，将 `"GameplayAbilities", "GameplayTags", "GameplayTasks"` 添加到您的 `PrivateDependencyModuleNames`
1. 刷新/重新生成您的 Visual Studio 项目文件
1. 从 4.24 到 5.2 版本，必须调用 `UAbilitySystemGlobals::Get().InitGlobalData()` 才能使用 [`TargetData`](#concepts-targeting-data)。示例项目在 `UAssetManager::StartInitialLoading()` 中执行此操作。从 5.3 版本开始，此操作会自动调用。有关更多信息，请参阅 [`InitGlobalData()`](#concepts-asg-initglobaldata)。

这就是启用 GAS 所需的全部操作。接下来，向您的 `Character` 或 `PlayerState` 添加 [`ASC`](#concepts-asc) 和 [`AttributeSet`](#concepts-as)，并开始创建 [`GameplayAbilities`](#concepts-ga) 和 [`GameplayEffects`](#concepts-ge)！

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts"></a>
## 4. GAS 概念

#### 章节

> 4.1 [Ability System Component](#concepts-asc)  
> 4.2 [Gameplay Tags](#concepts-gt)  
> 4.3 [Attributes](#concepts-a)  
> 4.4 [Attribute Set](#concepts-as)  
> 4.5 [Gameplay Effects](#concepts-ge)  
> 4.6 [Gameplay Abilities](#concepts-ga)  
> 4.7 [Ability Tasks](#concepts-at)  
> 4.8 [Gameplay Cues](#concepts-gc)  
> 4.9 [Ability System Globals](#concepts-asg)  
> 4.10 [Prediction](#concepts-p)  

<a name="concepts-asc"></a>
### 4.1 Ability System Component
`AbilitySystemComponent`（`ASC`）是 GAS 的核心。它是一个 `UActorComponent`（[`UAbilitySystemComponent`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemComponent/index.html)），处理与系统的所有交互。任何希望使用 [`GameplayAbilities`](#concepts-ga)、拥有 [`Attributes`](#concepts-a) 或接收 [`GameplayEffects`](#concepts-ge) 的 `Actor` 都必须附加一个 `ASC`。这些对象都存在于 `ASC` 内部，并由其管理和复制（`Attributes` 除外，它们由其 [`AttributeSet`](#concepts-as) 复制）。建议开发者对此类进行子类化，但不是强制的。

附加了 `ASC` 的 `Actor` 被称为 `ASC` 的 `OwnerActor`。`ASC` 的物理表现 `Actor` 被称为 `AvatarActor`。`OwnerActor` 和 `AvatarActor` 可以是同一个 `Actor`，例如 MOBA 游戏中的简单 AI 小兵。它们也可以是不同的 `Actors`，例如 MOBA 游戏中玩家控制的英雄，其中 `OwnerActor` 是 `PlayerState`，而 `AvatarActor` 是英雄的 `Character` 类。大多数 `Actors` 会将 `ASC` 放在自己身上。如果你的 `Actor` 会重生，并且需要在重生之间保持 `Attributes` 或 `GameplayEffects` 的持久性（如 MOBA 中的英雄），那么 `ASC` 的理想位置是在 `PlayerState` 上。

**注意：** 如果你的 `ASC` 在 `PlayerState` 上，则需要增加 `PlayerState` 的 `NetUpdateFrequency`。它在 `PlayerState` 上默认为一个非常低的值，可能会导致客户端在 `Attributes` 和 `GameplayTags` 等发生变化之前出现延迟或感知到的卡顿。确保启用 [`Adaptive Network Update Frequency`](https://docs.unrealengine.com/en-US/Gameplay/Networking/Actors/Properties/index.html#adaptivenetworkupdatefrequency)，Fortnite 使用了它。

如果 `OwnerActor` 和 `AvatarActor` 是不同的 `Actors`，则两者都应实现 `IAbilitySystemInterface`。此接口有一个必须重写的函数 `UAbilitySystemComponent* GetAbilitySystemComponent() const`，它返回指向其 `ASC` 的指针。`ASCs` 通过查找此接口函数在系统内部相互交互。

`ASC` 将其当前活动的 `GameplayEffects` 保存在 `FActiveGameplayEffectsContainer ActiveGameplayEffects` 中。

`ASC` 将其授予的 `Gameplay Abilities` 保存在 `FGameplayAbilitySpecContainer ActivatableAbilities` 中。任何时候你计划遍历 `ActivatableAbilities.Items`，请确保在循环上方添加 `ABILITYLIST_SCOPE_LOCK();` 以锁定列表，防止其发生变化（由于移除能力）。作用域内的每个 `ABILITYLIST_SCOPE_LOCK();` 都会增加 `AbilityScopeLockCount`，然后在超出作用域时递减。不要尝试在 `ABILITYLIST_SCOPE_LOCK();` 的作用域内移除能力（清除能力函数会在内部检查 `AbilityScopeLockCount` 以防止在列表被锁定时移除能力）。

<a name="concepts-asc-rm"></a>
### 4.1.1 Replication Mode
`ASC` 定义了三种不同的复制模式用于复制 `GameplayEffects`、`GameplayTags` 和 `GameplayCues` - `Full`、`Mixed` 和 `Minimal`。`Attributes` 由其 `AttributeSet` 复制。

| Replication Mode   | 使用场景                             | 描述                                                                                                                    |
| ------------------ | --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `Full`             | 单人游戏                           | 每个 `GameplayEffect` 都会复制到每个客户端。                                                                          |
| `Mixed`            | 多人游戏，玩家控制的 `Actors` | `GameplayEffects` 仅复制到拥有者客户端。只有 `GameplayTags` 和 `GameplayCues` 会复制给所有人。 |
| `Minimal`          | 多人游戏，AI 控制的 `Actors`     | `GameplayEffects` 从不复制给任何人。只有 `GameplayTags` 和 `GameplayCues` 会复制给所有人。           |

**注意：** `Mixed` 复制模式期望 `OwnerActor's` 的 `Owner` 是 `Controller`。`PlayerState's` 的 `Owner` 默认是 `Controller`，但 `Character's` 的不是。如果在 `OwnerActor` 不是 `PlayerState` 的情况下使用 `Mixed` 复制模式，则需要在 `OwnerActor` 上使用有效的 `Controller` 调用 `SetOwner()`。

从 4.24 开始，`PossessedBy()` 现在会将 `Pawn` 的拥有者设置为新的 `Controller`。

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-asc-setup"></a>
### 4.1.2 Setup and Initialization
`ASCs` 通常在 `OwnerActor's` 构造函数中构造，并显式标记为复制。**这必须在 C++ 中完成**。

```c++
AGDPlayerState::AGDPlayerState()
{
	// Create ability system component, and set it to be explicitly replicated
	AbilitySystemComponent = CreateDefaultSubobject<UGDAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
	AbilitySystemComponent->SetIsReplicated(true);
	//...
}
```

`ASC` 需要在服务器和客户端上使用其 `OwnerActor` 和 `AvatarActor` 进行初始化。你希望在 `Pawn's` 的 `Controller` 被设置后（在控制后）进行初始化。单人游戏只需要担心服务器路径。

对于 `ASC` 位于 `Pawn` 上的玩家控制角色，我通常在 `Pawn's` 的 `PossessedBy()` 函数中在服务器上初始化，并在 `PlayerController's` 的 `AcknowledgePossession()` 函数中在客户端上初始化。

```c++
void APACharacterBase::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->InitAbilityActorInfo(this, this);
	}

	// ASC MixedMode replication requires that the ASC Owner's Owner be the Controller.
	SetOwner(NewController);
}
```

```c++
void APAPlayerControllerBase::AcknowledgePossession(APawn* P)
{
	Super::AcknowledgePossession(P);

	APACharacterBase* CharacterBase = Cast<APACharacterBase>(P);
	if (CharacterBase)
	{
		CharacterBase->GetAbilitySystemComponent()->InitAbilityActorInfo(CharacterBase, CharacterBase);
	}

	//...
}
```

对于 `ASC` 位于 `PlayerState` 上的玩家控制角色，我通常在 `Pawn's` 的 `PossessedBy()` 函数中在服务器上初始化，并在 `Pawn's` 的 `OnRep_PlayerState()` 函数中在客户端上初始化。这确保 `PlayerState` 在客户端上存在。

```c++
// Server only
void AGDHeroCharacter::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// Set the ASC on the Server. Clients do this in OnRep_PlayerState()
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// AI won't have PlayerControllers so we can init again here just to be sure. No harm in initing twice for heroes that have PlayerControllers.
		PS->GetAbilitySystemComponent()->InitAbilityActorInfo(PS, this);
	}

	//...
}
```

```c++
// Client only
void AGDHeroCharacter::OnRep_PlayerState()
{
	Super::OnRep_PlayerState();

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// Set the ASC for clients. Server does this in PossessedBy.
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// Init ASC Actor Info for clients. Server will init its ASC when it possesses a new Actor.
		AbilitySystemComponent->InitAbilityActorInfo(PS, this);
	}

	// ...
}
```

如果你收到错误消息 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`，说明你没有在客户端上初始化 `ASC`。

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gt"></a>
### 4.2 Gameplay Tags
[`FGameplayTags`](https://docs.unrealengine.com/en-US/API/Runtime/GameplayTags/FGameplayTag/index.html) 是以 `Parent.Child.Grandchild...` 形式表示的分层名称，它们注册到 `GameplayTagManager` 中。这些标签对于分类和描述对象状态非常有用。例如，如果一个角色被眩晕，我们可以在眩晕持续期间给它一个 `State.Debuff.Stun` `GameplayTag`。

你会发现自己会使用 `GameplayTags` 来替代以前用布尔值或枚举处理的事物，并根据对象是否具有某些 `GameplayTags` 来执行布尔逻辑。

当给对象添加标签时，如果它有 `ASC`，我们通常会将标签添加到 `ASC` 中，以便 GAS 可以与它们交互。`UAbilitySystemComponent` 实现了 `IGameplayTagAssetInterface`，提供了访问其拥有的 `GameplayTags` 的函数。

多个 `GameplayTags` 可以存储在 `FGameplayTagContainer` 中。与 `TArray<FGameplayTag>` 相比，优先使用 `GameplayTagContainer`，因为 `GameplayTagContainers` 添加了一些效率优化。虽然标签是标准的 `FNames`，但如果在项目设置中启用了 `Fast Replication`，它们可以高效地打包在 `FGameplayTagContainers` 中进行复制。`Fast Replication` 要求服务器和客户端具有相同的 `GameplayTags` 列表。这通常不应该成为问题，因此你应该启用此选项。`GameplayTagContainers` 也可以返回用于迭代的 `TArray<FGameplayTag>`。

存储在 `FGameplayTagCountContainer` 中的 `GameplayTags` 有一个 `TagMap`，用于存储该 `GameplayTag` 的实例数量。`FGameplayTagCountContainer` 可能仍然包含 `GameplayTag`，但其 `TagMapCount` 为零。如果在调试时 `ASC` 仍然具有 `GameplayTag`，你可能会遇到这种情况。任何 `HasTag()` 或 `HasMatchingTag()` 或类似函数都会检查 `TagMapCount`，如果 `GameplayTag` 不存在或其 `TagMapCount` 为零，则返回 false。

`GameplayTags` 必须在 `DefaultGameplayTags.ini` 中预先定义。虚幻引擎编辑器在项目设置中提供了一个界面，让开发人员可以管理 `GameplayTags` 而无需手动编辑 `DefaultGameplayTags.ini`。`GameplayTag` 编辑器可以创建、重命名、搜索引用和删除 `GameplayTags`。

![GameplayTag Editor in Project Settings](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaytageditor.png)

搜索 `GameplayTag` 引用将在编辑器中显示熟悉的 `Reference Viewer` 图表，显示所有引用该 `GameplayTag` 的资源。但是，这不会显示任何引用该 `GameplayTag` 的 C++ 类。

重命名 `GameplayTags` 会创建一个重定向，以便仍然引用原始 `GameplayTag` 的资源可以重定向到新的 `GameplayTag`。如果可能，我更倾向于创建一个新的 `GameplayTag`，手动将所有引用更新到新的 `GameplayTag`，然后删除旧的 `GameplayTag`，以避免创建重定向。

除了 `Fast Replication` 之外，`GameplayTag` 编辑器还有一个选项可以填充常用复制的 `GameplayTags` 以进一步优化它们。

`GameplayTags` 如果从 `GameplayEffect` 添加则会被复制。`ASC` 允许你添加不会被复制的 `LooseGameplayTags`，必须手动管理它们。示例项目使用 `State.Dead` 的 `LooseGameplayTag`，以便拥有客户端可以立即响应其生命值降为零的情况。重生会手动将 `TagMapCount` 设置回零。在使用 `LooseGameplayTags` 时，仅手动调整 `TagMapCount`。最好使用 `UAbilitySystemComponent::AddLooseGameplayTag()` 和 `UAbilitySystemComponent::RemoveLooseGameplayTag()` 函数，而不是手动调整 `TagMapCount`。

在 C++ 中获取对 `GameplayTag` 的引用：
```c++
FGameplayTag::RequestGameplayTag(FName("Your.GameplayTag.Name"))
```

对于高级 `GameplayTag` 操作（如获取父级或子级 `GameplayTags`），请查看 `GameplayTagManager` 提供的函数。要访问 `GameplayTagManager`，请包含 `GameplayTagManager.h` 并使用 `UGameplayTagManager::Get().FunctionName` 调用它。`GameplayTagManager` 实际上将 `GameplayTags` 存储为关系节点（父节点、子节点等），以实现比常量字符串操作和比较更快的处理。

`GameplayTags` 和 `GameplayTagContainers` 可以具有可选的 `UPROPERTY` 说明符 `Meta = (Categories = "GameplayCue")`，该说明符过滤蓝图中显示的标签，仅显示具有 `GameplayCue` 父标签的 `GameplayTags`。当你知道 `GameplayTag` 或 `GameplayTagContainer` 变量应仅用于 `GameplayCues` 时，这很有用。

或者，有一个名为 `FGameplayCueTag` 的单独结构，它封装了 `FGameplayTag`，并且还会自动过滤蓝图中的 `GameplayTags`，仅显示具有 `GameplayCue` 父标签的标签。

如果要过滤函数中的 `GameplayTag` 参数，请使用 `UFUNCTION` 说明符 `Meta = (GameplayTagFilter = "GameplayCue")`。函数中的 `GameplayTagContainer` 参数无法被过滤。如果要编辑引擎以允许此操作，请查看 `Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagGraphPin.cpp` 中的 `SGameplayTagGraphPin::ParseDefaultValueData()` 如何调用 `FilterString = UGameplayTagsManager::Get().GetCategoriesMetaFromField(PinStructType);` 并在 `SGameplayTagGraphPin::GetListContent()` 中将 `FilterString` 传递给 `SGameplayTagWidget`。`Engine\Plugins\Editor\GameplayTagsEditor\Source\GameplayTagsEditor\Private\SGameplayTagContainerGraphPin.cpp` 中这些函数的 `GameplayTagContainer` 版本不检查元字段属性，也不传递过滤器。

示例项目广泛使用了 `GameplayTags`。

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gt-change"></a>
### 4.2.1 响应 Gameplay Tags 的变化
`ASC` 为添加或移除 `GameplayTags` 时提供了一个委托。它接收一个 `EGameplayTagEventType`，可以指定仅在添加/移除 `GameplayTag` 时触发，或者在 `GameplayTag's` 的 `TagMapCount` 发生任何变化时触发。

```c++
AbilitySystemComponent->RegisterGameplayTagEvent(FGameplayTag::RequestGameplayTag(FName("State.Debuff.Stun")), EGameplayTagEventType::NewOrRemoved).AddUObject(this, &AGDPlayerState::StunTagChanged);
```

回调函数具有一个 `GameplayTag` 和新 `TagCount` 的参数。
```c++
virtual void StunTagChanged(const FGameplayTag CallbackTag, int32 NewCount);
```

**[⬆ Back to Top](#table-of-contents)**

<a name="concepts-gt-loadfromplugin"></a>
### 4.2.2 从插件 .ini 文件加载 Gameplay Tags
如果你创建了一个带有自己的 .ini 文件和 `GameplayTags` 的插件，可以在插件的 `StartupModule()` 函数中加载该插件的 `GameplayTag` .ini 目录。

例如，这是虚幻引擎附带的 CommonConversation 插件的实现方式：

```c++
void FCommonConversationRuntimeModule::StartupModule()
{
	TSharedPtr<IPlugin> ThisPlugin = IPluginManager::Get().FindPlugin(TEXT("CommonConversation"));
	check(ThisPlugin.IsValid());

	UGameplayTagsManager::Get().AddTagIniSearchPath(ThisPlugin->GetBaseDir() / TEXT("Config") / TEXT("Tags"));

	//...
}
```

这将在目录 `Plugins\CommonConversation\Config\Tags` 中查找，并在引擎启动时（如果启用了插件）将其中包含 `GameplayTags` 的任何 .ini 文件加载到项目中。

**[⬆ Back to Top](#table-of-contents)**


<a name="concepts-a"></a>
### 4.3 Attributes

<a name="concepts-a-definition"></a>
#### 4.3.1 Attribute 定义
`Attributes` 是由结构体 [`FGameplayAttributeData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayAttributeData/index.html) 定义的浮点数值。它们可以表示从角色生命值到角色等级，再到药水充能次数等任何内容。如果它是属于某个 `Actor` 的与游戏相关的数值，你应该考虑使用 `Attribute`。`Attributes` 通常应该只由 [`GameplayEffects`](#concepts-ge) 修改，以便 ASC 可以[预测](#concepts-p)这些更改。

`Attributes` 由 [`AttributeSet`](#concepts-as) 定义并存在于其中。`AttributeSet` 负责复制标记为复制的 `Attributes`。有关如何定义 `Attributes` 的信息，请参阅 [`AttributeSets`](#concepts-as) 章节。

**提示：**如果你不希望某个 `Attribute` 显示在编辑器的 `Attributes` 列表中，可以使用 `Meta = (HideInDetailsView)` `属性说明符`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-a-value"></a>
#### 4.3.2 BaseValue 与 CurrentValue
一个 `Attribute` 由两个值组成 - `BaseValue` 和 `CurrentValue`。`BaseValue` 是 `Attribute` 的永久值，而 `CurrentValue` 是 `BaseValue` 加上来自 `GameplayEffects` 的临时修改。例如，你的 `Character` 可能有一个移动速度 `Attribute`，其 `BaseValue` 为 600 单位/秒。由于还没有 `GameplayEffects` 修改移动速度，因此 `CurrentValue` 也是 600 u/s。如果她获得一个临时 50 u/s 的移动速度增益，`BaseValue` 保持不变，仍为 600 u/s，而 `CurrentValue` 现在是 600 + 50，总计 650 u/s。当移动速度增益过期时，`CurrentValue` 会恢复到 `BaseValue` 的 600 u/s。

GAS 的初学者经常会将 `BaseValue` 与 `Attribute` 的最大值混淆，并试图将其作为最大值处理。这是一种不正确的方法。可以更改或在能力或 UI 中引用的 `Attributes` 的最大值应该被视为单独的 `Attributes`。对于硬编码的最大值和最小值，有一种方法可以定义一个包含 `FAttributeMetaData` 的 `DataTable`，可以设置最大值和最小值，但 Epic 在结构体上方的注释称其为"正在进行的工作"。有关更多信息，请参阅 `AttributeSet.h`。为避免混淆，我建议可以在能力或 UI 中引用的最大值应作为单独的 `Attributes` 创建，而仅用于限制 `Attributes` 的硬编码最大值和最小值应在 `AttributeSet` 中定义为硬编码浮点数。`Attributes` 的限制在 [PreAttributeChange()](#concepts-as-preattributechange) 中讨论，用于更改 `CurrentValue`，在 [PostGameplayEffectExecute()](#concepts-as-postgameplayeffectexecute) 中讨论，用于通过 `GameplayEffects` 更改 `BaseValue`。

`BaseValue` 的永久更改来自 `Instant`（即时）`GameplayEffects`，而 `Duration`（持续）和 `Infinite`（无限）`GameplayEffects` 更改 `CurrentValue`。周期性 `GameplayEffects` 被视为即时 `GameplayEffects` 并更改 `BaseValue`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-a-meta"></a>
#### 4.3.3 Meta Attributes
某些 `Attributes` 被用作旨在与 `Attributes` 交互的临时值的占位符。这些被称为 `Meta Attributes`。例如，我们通常将伤害定义为一个 `Meta Attribute`。我们不使用 `GameplayEffect` 直接更改生命值 `Attribute`，而是使用一个名为 damage 的 `Meta Attribute` 作为占位符。这样，伤害值可以在 [`GameplayEffectExecutionCalculation`](#concepts-ge-ec) 中被增益和减益修改，并可以在 `AttributeSet` 中进一步操作，例如先从当前护盾 `Attribute` 中减去伤害，最后再从生命值 `Attribute` 中减去剩余部分。伤害 `Meta Attribute` 在 `GameplayEffects` 之间没有持久性，并且会被每个 `GameplayEffect` 覆盖。`Meta Attributes` 通常不会被复制。

`Meta Attributes` 为伤害和治疗之类的事物提供了很好的逻辑分离，区分"我们造成了多少伤害？"和"我们要如何处理这个伤害？"。这种逻辑分离意味着我们的 `Gameplay Effects` 和 `Execution Calculations` 不需要知道目标如何处理伤害。继续我们的伤害示例，`Gameplay Effect` 决定造成多少伤害，然后 `AttributeSet` 决定如何处理该伤害。并非所有角色都具有相同的 `Attributes`，特别是如果你使用子类化的 `AttributeSets`。基础 `AttributeSet` 类可能只有生命值 `Attribute`，但子类化的 `AttributeSet` 可能会添加护盾 `Attribute`。具有护盾 `Attribute` 的子类化 `AttributeSet` 与基础 `AttributeSet` 类分发接收到的伤害的方式不同。

虽然 `Meta Attributes` 是一个很好的设计模式，但它们不是强制性的。如果你只使用一个 `Execution Calculation` 来处理所有伤害实例，并且所有角色共享一个 `Attribute Set` 类，那么你可以直接在 `Execution Calculation` 中进行伤害分发到生命值、护盾等，并直接修改这些 `Attributes`。你只会牺牲灵活性，但这对你来说可能没问题。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-a-changes"></a>
#### 4.3.4 响应 Attribute 更改
要监听 `Attribute` 更改以更新 UI 或其他游戏逻辑，请使用 `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`。此函数返回一个可以绑定的委托，每当 `Attribute` 更改时都会自动调用。该委托提供一个包含 `NewValue`、`OldValue` 和 `FGameplayEffectModCallbackData` 的 `FOnAttributeChangeData` 参数。**注意：** `FGameplayEffectModCallbackData` 只会在服务器上设置。

```c++
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(AttributeSetBase->GetHealthAttribute()).AddUObject(this, &AGDPlayerState::HealthChanged);
```

```c++
virtual void HealthChanged(const FOnAttributeChangeData& Data);
```

示例项目在 `GDPlayerState` 上绑定到 `Attribute` 值更改委托，以更新 HUD 并在生命值归零时响应玩家死亡。

示例项目中包含一个自定义的 Blueprint 节点，将其包装在 `ASyncTask` 中。它在 `UI_HUD` UMG Widget 中用于更新生命值、法力值和体力值。这个 `AsyncTask` 将一直存在，直到手动调用 `EndTask()`，我们在 UMG Widget 的 `Destruct` 事件中执行此操作。请参阅 `AsyncTaskAttributeChanged.h/cpp`。

![Listen for Attribute Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/attributechange.png)

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-a-derived"></a>
#### 4.3.5 派生 Attributes
要创建一个其部分或全部值从一个或多个其他 `Attributes` 派生的 `Attribute`，请使用一个或多个 `Attribute Based` 或 [`MMC`](#concepts-ge-mmc) [`Modifiers`](#concepts-ge-mods) 的 `Infinite` `GameplayEffect`。当派生 `Attribute` 所依赖的 `Attribute` 更新时，`Derived Attribute` 将自动更新。

`Derived Attribute` 上所有 `Modifiers` 的最终公式与 `Modifier Aggregators` 的公式相同。如果需要按特定顺序进行计算，请在一个 `MMC` 中完成所有操作。

```
((CurrentValue + Additive) * Multiplicitive) / Division
```

**注意：** 如果在 PIE 中使用多个客户端进行游戏，你需要在编辑器首选项中禁用 `Run Under One Process`，否则当独立 `Attributes` 在第一个客户端之外的其他客户端上更新时，`Derived Attributes` 将不会更新。

在此示例中，我们有一个 `Infinite` `GameplayEffect`，它从 `Attributes` `TestAttrB` 和 `TestAttrC` 派生 `TestAttrA` 的值，公式为 `TestAttrA = (TestAttrA + TestAttrB) * ( 2 * TestAttrC)`。每当任何 `Attributes` 更新其值时，`TestAttrA` 都会自动重新计算其值。

![Derived Attribute Example](https://github.com/tranek/GASDocumentation/raw/master/Images/derivedattribute.png)

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-as"></a>
### 4.4 Attribute Set

<a name="concepts-as-definition"></a>
#### 4.4.1 Attribute Set 定义
`AttributeSet` 定义、保存并管理对 `Attributes` 的更改。开发者应该从 [`UAttributeSet`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAttributeSet/index.html) 子类化。在 `OwnerActor's` 构造函数中创建 `AttributeSet` 会自动将其注册到其 `ASC`。**这必须在 C++ 中完成**。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-as-design"></a>
#### 4.4.2 Attribute Set 设计
一个 `ASC` 可能有一个或多个 `AttributeSets`。AttributeSets 的内存开销可以忽略不计，因此使用多少 `AttributeSets` 是由开发者自行决定的组织决策。

可以接受在游戏中的每个 `Actor` 之间共享一个大型单体 `AttributeSet`，并且仅在需要时使用属性，而忽略未使用的属性。

或者，你可以选择有多个 `AttributeSets`，代表按需选择性添加到 `Actors` 的 `Attributes` 分组。例如，你可以有一个与生命值相关的 `Attributes` 的 `AttributeSet`，一个与法力值相关的 `Attributes` 的 `AttributeSet`，等等。在 MOBA 游戏中，英雄可能需要法力值，但小兵可能不需要。因此，英雄将获得法力值 `AttributeSet`，而小兵则不会。

此外，`AttributeSets` 可以被子类化，作为选择性选择 `Actor` 具有哪些 `Attributes` 的另一种方式。`Attributes` 在内部被称为 `AttributeSetClassName.AttributeName`。当你子类化 `AttributeSet` 时，来自父类的所有 `Attributes` 仍将使用父类名称作为前缀。

虽然你可以有多个 `AttributeSet`，但不应该在 `ASC` 上有多个相同类的 `AttributeSet`。如果你有多个来自同一类的 `AttributeSet`，它将不知道使用哪个 `AttributeSet`，只会选择一个。

<a name="concepts-as-design-subcomponents"></a>
##### 4.4.2.1 具有独立 Attributes 的子组件
在 `Pawn` 上有多个可损坏组件（例如可单独损坏的装甲部件）的场景中，我建议如果你知道 `Pawn` 可能拥有的可损坏组件的最大数量，则在一个 `AttributeSet` 上创建那么多的生命值 `Attributes` - DamageableCompHealth0、DamageableCompHealth1 等，以表示这些可损坏组件的逻辑"插槽"。在你的可损坏组件类实例中，分配插槽编号 `Attribute`，可以被 `GameplayAbilities` 或 [`Executions`](#concepts-ge-ec) 读取，以知道要将伤害应用于哪个 `Attribute`。拥有少于最大数量或零个可损坏组件的 `Pawns` 是可以的。仅仅因为 `AttributeSet` 有一个 `Attribute`，并不意味着你必须使用它。未使用的 `Attributes` 占用的内存量很少。

如果你的子组件各自需要许多 `Attributes`，可能有无限数量的子组件，子组件可以分离并被其他玩家使用（例如武器），或者出于任何其他原因这种方法不适合你，我建议不再使用 `Attributes`，而是在组件上存储普通的浮点数。请参阅 [Item Attributes](#concepts-as-design-itemattributes)。

<a name="concepts-as-design-addremoveruntime"></a>
##### 4.4.2.2 在运行时添加和删除 AttributeSets
`AttributeSets` 可以在运行时从 `ASC` 添加和删除；但是，删除 `AttributeSets` 可能是危险的。例如，如果 `AttributeSet` 在客户端上先于服务器被删除，并且 `Attribute` 值更改被复制到客户端，`Attribute` 将找不到其 `AttributeSet` 并导致游戏崩溃。

在添加武器到物品栏时：
```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().AddUnique(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```

在从物品栏移除武器时：
```c++
AbilitySystemComponent->GetSpawnedAttributes_Mutable().Remove(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```

<a name="concepts-as-design-itemattributes"></a>
##### 4.4.2.3 物品 Attributes（武器弹药）
有几种方法可以实现带有 `Attributes` 的可装备物品（武器弹药、护甲耐久度等）。所有这些方法都将值直接存储在物品上。这对于可以在其生命周期内被多个玩家装备的物品是必要的。

> 1. 在物品上使用普通浮点数（**推荐**）
> 1. 物品上的单独 `AttributeSet`
> 1. 物品上的单独 `ASC`

<a name="concepts-as-design-itemattributes-plainfloats"></a>
###### 4.4.2.3.1 物品上的普通浮点数
不使用 `Attributes`，而是在物品类实例上存储普通的浮点值。Fortnite 和 [GASShooter](https://github.com/tranek/GASShooter) 以这种方式处理枪支弹药。对于枪支，将最大弹夹大小、弹夹中当前弹药、备用弹药等直接作为复制浮点数（`COND_OwnerOnly`）存储在枪支实例上。如果武器共享备用弹药，你会将备用弹药移至角色上作为共享弹药 `AttributeSet` 中的 `Attribute`（装弹能力可以使用 `Cost GE` 从备用弹药中提取到枪支的浮点弹夹弹药中）。由于你没有为当前弹夹弹药使用 `Attributes`，你需要在 `UGameplayAbility` 中重写某些函数，以检查并根据枪支上的浮点数应用成本。在授予权限时将枪支作为 [`GameplayAbilitySpec`](https://github.com/tranek/GASDocumentation#concepts-ga-spec) 中的 `SourceObject` 意味着你可以在能力内部访问授予权限的枪支。

为了防止枪支在自动开火期间将弹药量复制回来并覆盖本地弹药量，在 `PreReplication()` 中当玩家具有 `IsFiring` `GameplayTag` 时禁用复制。你本质上是在这里进行自己的本地预测。

```c++
void AGSWeapon::PreReplication(IRepChangedPropertyTracker& ChangedPropertyTracker)
{
	Super::PreReplication(ChangedPropertyTracker);

	DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, PrimaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
	DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, SecondaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
}
```

优点：
1. 避免了使用 `AttributeSets` 的限制（见下文）

限制：
1. 不能使用现有的 `GameplayEffect` 工作流程（用于弹药使用的 `Cost GEs` 等）
1. 需要工作来重写 `UGameplayAbility` 上的关键函数，以检查并根据枪支上的浮点数应用弹药成本

<a name="concepts-as-design-itemattributes-attributeset"></a>
###### 4.4.2.3.2 物品上的 `AttributeSet`
在物品上使用单独的 `AttributeSet`，在[将其添加到玩家物品栏时添加到玩家的 `ASC`](#concepts-as-design-addremoveruntime)，可以工作，但它有一些主要限制。我在 [GASShooter](https://github.com/tranek/GASShooter) 的早期版本中为此做了武器弹药。武器将其 `Attributes`（例如最大弹夹大小、弹夹中当前弹药、备用弹药等）存储在位于武器类上的 `AttributeSet` 中。如果武器共享备用弹药，你会将备用弹药移至共享弹药 `AttributeSet` 中的角色上。当武器在服务器上添加到玩家物品栏时，武器会将其 `AttributeSet` 添加到玩家的 `ASC::SpawnedAttributes`。然后服务器会将其复制到客户端。如果武器从物品栏中移除，它会从 `ASC::SpawnedAttributes` 中删除其 `AttributeSet`。

当 `AttributeSet` 位于 `OwnerActor` 以外的其他东西（例如武器）上时，你最初会在 `AttributeSet` 中遇到一些编译错误。解决方法是在 `BeginPlay()` 而不是构造函数中构造 `AttributeSet`，并在武器上实现 `IAbilitySystemInterface`（在将武器添加到玩家物品栏时将指针设置为 `ASC`）。

```c++
void AGSWeapon::BeginPlay()
{
	if (!AttributeSet)
	{
		AttributeSet = NewObject<UGSWeaponAttributeSet>(this);
	}
	//...
}
```

你可以通过查看 [GASShooter 的旧版本](https://github.com/tranek/GASShooter/tree/df5949d0dd992bd3d76d4a728f370f2e2c827735) 来实际了解它。

优点：
1. 可以使用现有的 `GameplayAbility` 和 `GameplayEffect` 工作流程（用于弹药使用的 `Cost GEs` 等）
1. 对于非常少量的物品来说设置简单

限制：
1. 你必须为每种武器类型创建一个新的 `AttributeSet` 类。`ASCs` 在功能上只能拥有一个类的一个 `AttributeSet` 实例，因为对 `Attribute` 的更改会在 `ASCs` 的 `SpawnedAttributes` 数组中查找其 `AttributeSet` 类的第一个实例。同一 `AttributeSet` 类的其他实例将被忽略。
1. 由于每个 `AttributeSet` 类只有一个 `AttributeSet` 实例，你只能在玩家的物品栏中拥有每种类型的武器之一。
1. 删除 `AttributeSet` 是危险的。在 GASShooter 中，如果玩家用火箭炸死了自己，玩家会立即从物品栏中移除火箭发射器（包括其来自 `ASC` 的 `AttributeSet`）。当服务器复制火箭发射器的弹药 `Attribute` 更改时，`AttributeSet` 在客户端的 `ASC` 上不再存在，游戏崩溃。

<a name="concepts-as-design-itemattributes-asc"></a>
###### 4.4.2.3.3 物品上的 `ASC`
在每个物品上放置一个完整的 `AbilitySystemComponent` 是一种极端的方法。我个人没有这样做，也没有在野外见过它。这需要大量的工程工作才能使其工作。

> 拥有多个具有相同所有者但不同化身的 AbilitySystemComponents（例如在 pawn 和武器/物品/投射物上，Owner 设置为 PlayerState）是否可行？
>
> 我看到的第一个问题是在拥有者 actor 上实现 IGameplayTagAssetInterface 和 IAbilitySystemInterface。前者可能可行：只是聚合来自所有 ASC 的标签（但要注意 - HasAllMatchingGameplayTags 可能只能通过跨 ASC 聚合来满足。仅仅将调用转发到每个 ASC 并将结果 OR 在一起是不够的）。但后者更棘手：哪个 ASC 是权威的？如果有人想应用 GE - 哪个应该接收它？也许你可以解决这些问题，但问题的这一方面将是最困难的：拥有者下面将有多个 ASC。
>
> pawn 和武器上的单独 ASC 本身是有意义的。例如，区分描述武器的标签和描述拥有 pawn 的标签。也许授予武器的标签也"应用"于所有者而不应用其他东西（例如，属性和 GE 是独立的，但所有者将像我上面描述的那样聚合拥有的标签）。这可能是可行的，我确信。但是拥有多个具有相同所有者的 ASC 可能会变得棘手。

*Epic 的 Dave Ratti 对[社区问题 #6](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)的回答*

优点：
1. 可以使用现有的 `GameplayAbility` 和 `GameplayEffect` 工作流程（用于弹药使用的 `Cost GEs` 等）
1. 可以重用 `AttributeSet` 类（每个武器 ASC 上一个）

限制：
1. 未知的工程成本
1. 这可能吗？

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-as-attributes"></a>
#### 4.4.3 定义 Attributes
**`Attributes` 只能在 C++ 中**在 `AttributeSet's` 头文件中定义。建议将此宏块添加到每个 `AttributeSet` 头文件的顶部。它将自动为你的 `Attributes` 生成 getter 和 setter 函数。

```c++
// 使用来自 AttributeSet.h 的宏
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```

复制的生命值属性将定义如下：

```c++
UPROPERTY(BlueprintReadOnly, Category = "Health", ReplicatedUsing = OnRep_Health)
FGameplayAttributeData Health;
ATTRIBUTE_ACCESSORS(UGDAttributeSetBase, Health)
```

还要在头文件中定义 `OnRep` 函数：
```c++
UFUNCTION()
virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);
```

`AttributeSet` 的 .cpp 文件应该使用预测系统使用的 `GAMEPLAYATTRIBUTE_REPNOTIFY` 宏填充 `OnRep` 函数：
```c++
void UGDAttributeSetBase::OnRep_Health(const FGameplayAttributeData& OldHealth)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGDAttributeSetBase, Health, OldHealth);
}
```

最后，需要将 `Attribute` 添加到 `GetLifetimeReplicatedProps`：
```c++
void UGDAttributeSetBase::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME_CONDITION_NOTIFY(UGDAttributeSetBase, Health, COND_None, REPNOTIFY_Always);
}
```

`REPNOTIFY_Always` 告诉 `OnRep` 函数，如果本地值已经等于从服务器复制下来的值（由于预测），则触发。默认情况下，如果本地值与从服务器复制下来的值相同，它不会触发 `OnRep` 函数。

如果 `Attribute` 不像 `Meta Attribute` 那样被复制，则可以跳过 `OnRep` 和 `GetLifetimeReplicatedProps` 步骤。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-as-init"></a>
#### 4.4.4 初始化 Attributes
有多种方法可以初始化 `Attributes`（将其 `BaseValue` 以及随之而来的 `CurrentValue` 设置为某个初始值）。Epic 建议使用即时 `GameplayEffect`。这也是示例项目中使用的方法。

请参阅示例项目中的 `GE_HeroAttributes` Blueprint，了解如何创建即时 `GameplayEffect` 来初始化 `Attributes`。此 `GameplayEffect` 的应用在 C++ 中进行。

如果你在定义 `Attributes` 时使用了 `ATTRIBUTE_ACCESSORS` 宏，则会自动在 `AttributeSet` 上为每个 `Attribute` 生成初始化函数，你可以随时在 C++ 中调用。

```c++
// InitHealth(float InitialValue) 是使用 `ATTRIBUTE_ACCESSORS` 宏定义的 'Health' Attribute 的自动生成函数
AttributeSet->InitHealth(100.0f);
```

有关初始化 `Attributes` 的更多方法，请参阅 `AttributeSet.h`。

**注意：** 在 4.24 之前，`FAttributeSetInitterDiscreteLevels` 不适用于 `FGameplayAttributeData`。它是在 `Attributes` 是原始浮点数时创建的，并且会抱怨 `FGameplayAttributeData` 不是 `Plain Old Data` (`POD`)。这在 4.24 中已修复 https://issues.unrealengine.com/issue/UE-76557。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-as-preattributechange"></a>
#### 4.4.5 PreAttributeChange()
`PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)` 是 `AttributeSet` 中在更改发生之前响应对 `Attribute's` `CurrentValue` 更改的主要函数之一。这是通过引用参数 `NewValue` 限制对 `CurrentValue` 的传入更改的理想位置。

例如，为了限制移动速度修改器，示例项目这样做：
```c++
if (Attribute == GetMoveSpeedAttribute())
{
	// 不能慢于 150 单位/秒，不能快于 1000 单位/秒
	NewValue = FMath::Clamp<float>(NewValue, 150, 1000);
}
```
`GetMoveSpeedAttribute()` 函数由我们添加到 `AttributeSet.h` 的宏块创建（[定义 Attributes](#concepts-as-attributes)）。

这可以从对 `Attributes` 的任何更改触发，无论是使用 `Attribute` setter（由 `AttributeSet.h` 中的宏块定义（[定义 Attributes](#concepts-as-attributes)））还是使用 [`GameplayEffects`](#concepts-ge)。

**注意：** 这里发生的任何限制都不会永久更改 `ASC` 上的修改器。它只更改从查询修改器返回的值。这意味着任何从所有修改器重新计算 `CurrentValue` 的东西，如 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 和 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc)，都需要再次实现限制。

**注意：** Epic 对 `PreAttributeChange()` 的注释说不要将其用于游戏事件，而主要用于限制。在 `Attribute` 更改上进行游戏事件的推荐位置是 `UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`（[响应 Attribute 更改](#concepts-a-changes)）。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-as-postgameplayeffectexecute"></a>
#### 4.4.6 PostGameplayEffectExecute()
`PostGameplayEffectExecute(const FGameplayEffectModCallbackData & Data)` 仅在即时 [`GameplayEffect`](#concepts-ge) 更改 `Attribute` 的 `BaseValue` 后触发。这是在它们从 `GameplayEffect` 更改时进行更多 `Attribute` 操作的有效位置。

例如，在示例项目中，我们在此处从生命值 `Attribute` 中减去最终伤害 `Meta Attribute`。如果有护盾 `Attribute`，我们会先从护盾中减去伤害，然后再从生命值中减去剩余部分。示例项目还使用此位置来应用受击反应动画、显示浮动伤害数字，并为杀手分配经验和黄金奖励。根据设计，伤害 `Meta Attribute` 将始终通过即时 `GameplayEffect` 传递，而绝不会通过 `Attribute` setter。

只有在即时 `GameplayEffects` 中更改 `BaseValue` 的其他 `Attributes`，例如法力值和体力，也可以在此处限制为其最大值对应的 `Attributes`。

**注意：** 当调用 `PostGameplayEffectExecute()` 时，对 `Attribute` 的更改已经发生，但它们还没有复制回客户端，因此在此处限制值不会导致对客户端进行两次网络更新。客户端将只收到限制后的更新。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-as-onattributeaggregatorcreated"></a>
#### 4.4.7 OnAttributeAggregatorCreated()
`OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator)` 在为此集合中的 `Attribute` 创建 `Aggregator` 时触发。它允许自定义设置 [`FAggregatorEvaluateMetaData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FAggregatorEvaluateMetaData/index.html)。`AggregatorEvaluateMetaData` 被 `Aggregator` 用于基于应用于 `Attribute` 的所有 [`Modifiers`](#concepts-ge-mods) 评估 `CurrentValue`。默认情况下，`AggregatorEvaluateMetaData` 仅被 `Aggregator` 用于确定哪些 `Modifiers` 符合条件，例如 `MostNegativeMod_AllPositiveMods`，它允许所有正 `Modifiers`，但将负 `Modifiers` 限制为仅最负的一个。Paragon 使用它来只允许最负的移动速度减速效果应用于玩家，而不管他们同时受到多少减速效果的影响，同时应用所有正的移动速度增益。不符合条件的 `Modifiers` 仍然存在于 `ASC` 上，它们只是不会被聚合到最终的 `CurrentValue` 中。一旦条件发生变化，它们可能会在以后符合条件，例如如果最负的 `Modifier` 过期，那么下一个最负的 `Modifier`（如果存在）就符合条件。

要在仅允许最负的 `Modifier` 和所有正的 `Modifiers` 的示例中使用 AggregatorEvaluateMetaData：

```c++
virtual void OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const override;
```

```c++
void UGSAttributeSetBase::OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const
{
	Super::OnAttributeAggregatorCreated(Attribute, NewAggregator);

	if (!NewAggregator)
	{
		return;
	}

	if (Attribute == GetMoveSpeedAttribute())
	{
		NewAggregator->EvaluationMetaData = &FAggregatorEvaluateMetaDataLibrary::MostNegativeMod_AllPositiveMods;
	}
}
```

你应该将用于限定符的自定义 `AggregatorEvaluateMetaData` 作为静态变量添加到 `FAggregatorEvaluateMetaDataLibrary`。

**[⬆ 返回顶部](#table-of-contents)**


<a name="concepts-ge"></a>
### 4.5 Gameplay Effects

<a name="concepts-ge-definition"></a>
#### 4.5.1 Gameplay Effect Definition
[`GameplayEffects`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffect/index.html) (`GE`) 是 ability 改变自身和他人的 [`Attributes`](#concepts-a) 和 [`GameplayTags`](#concepts-gt) 的载体。它们可以造成立即的 `Attribute` 变化,如伤害或治疗,或者应用长期的状态增益/减益,如移动速度提升或眩晕。`UGameplayEffect` 类是一个**仅数据**类,用于定义单个 gameplay effect。不应向 `GameplayEffects` 添加任何额外的逻辑。通常,设计人员会创建许多 `UGameplayEffect` 的 Blueprint 子类。

`GameplayEffects` 通过 [`Modifiers`](#concepts-ge-mods) 和 [`Executions` (`GameplayEffectExecutionCalculation`)](#concepts-ge-ec) 来改变 `Attributes`。

`GameplayEffects` 有三种持续时间类型:`Instant`、`Duration` 和 `Infinite`。

此外,`GameplayEffects` 可以添加/执行 [`GameplayCues`](#concepts-gc)。`Instant` `GameplayEffect` 将在 `GameplayCue` `GameplayTags` 上调用 `Execute`,而 `Duration` 或 `Infinite` `GameplayEffect` 将在 `GameplayCue` `GameplayTags` 上调用 `Add` 和 `Remove`。

| Duration Type | GameplayCue Event | When to use                                                                                                                                                                                                                                |
| ------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Instant`     | Execute           | 用于对 `Attribute` 的 `BaseValue` 进行立即的永久性更改。`GameplayTags` 不会被应用,甚至不会应用一帧。                                                                                                                    |
| `Duration`    | Add & Remove      | 用于对 `Attribute` 的 `CurrentValue` 进行临时更改,并应用 `GameplayTags`,当 `GameplayEffect` 过期或被手动移除时,这些 `GameplayTags` 将被移除。持续时间在 `UGameplayEffect` 类/Blueprint 中指定。       |
| `Infinite`    | Add & Remove      | 用于对 `Attribute` 的 `CurrentValue` 进行临时更改,并应用 `GameplayTags`,当 `GameplayEffect` 被移除时,这些 `GameplayTags` 将被移除。这些永远不会自动过期,必须由 ability 或 `ASC` 手动移除。 |

`Duration` 和 `Infinite` `GameplayEffects` 可以选择应用 `Periodic Effects`,即按照其 `Period` 定义的每 `X` 秒应用其 `Modifiers` 和 `Executions`。在更改 `Attribute` 的 `BaseValue` 和 `Executing` `GameplayCues` 方面,`Periodic Effects` 被视为 `Instant` `GameplayEffects`。这些对于持续伤害(DOT)类型的效果非常有用。**注意:**`Periodic Effects` 无法被[预测](#concepts-p)。

如果 `Duration` 和 `Infinite` `GameplayEffects` 的 `Ongoing Tag Requirements` 未满足/满足([Gameplay Effect Tags](#concepts-ge-tags)),则可以在应用后临时关闭和打开它们。关闭 `GameplayEffect` 会移除其 `Modifiers` 和应用的 `GameplayTags` 的效果,但不会移除 `GameplayEffect`。重新打开 `GameplayEffect` 会重新应用其 `Modifiers` 和 `GameplayTags`。

如果您需要手动重新计算 `Duration` 或 `Infinite` `GameplayEffect` 的 `Modifiers`(比如您有一个使用不来自 `Attributes` 的数据的 `MMC`),您可以使用 `UAbilitySystemComponent::ActiveGameplayEffects.GetActiveGameplayEffect(ActiveHandle).Spec.GetLevel()` 获取其当前等级,然后使用相同的等级调用 `UAbilitySystemComponent::ActiveGameplayEffects.SetActiveGameplayEffectLevel(FActiveGameplayEffectHandle ActiveHandle, int32 NewLevel)`。基于 backing `Attributes` 的 `Modifiers` 会在这些 backing `Attributes` 更新时自动更新。`SetActiveGameplayEffectLevel()` 用于更新 `Modifiers` 的关键函数是:

```C++
MarkItemDirty(Effect);
Effect.Spec.CalculateModifierMagnitudes();
// 如果不是私有函数,我们可以直接调用这三个函数,而不需要将等级设置为它已经有的值
UpdateAllAggregatorModMagnitudes(Effect);
```

`GameplayEffects` 通常不会被实例化。当 ability 或 `ASC` 想要应用 `GameplayEffect` 时,它会从 `GameplayEffect` 的 `ClassDefaultObject` 创建 [`GameplayEffectSpec`](#concepts-ge-spec)。成功应用的 `GameplayEffectSpecs` 会被添加到一个名为 `FActiveGameplayEffect` 的新结构体中,这是 `ASC` 在一个名为 `ActiveGameplayEffects` 的特殊容器结构体中跟踪的内容。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-applying"></a>
#### 4.5.2 应用 Gameplay Effects
`GameplayEffects` 可以通过多种方式从 [`GameplayAbilities`](#concepts-ga) 的函数和 `ASC` 的函数应用,通常采用 `ApplyGameplayEffectTo` 的形式。不同的函数本质上是便捷函数,最终将在 `Target` 上调用 `UAbilitySystemComponent::ApplyGameplayEffectSpecToSelf()`。

要在 `GameplayAbility` 之外应用 `GameplayEffects`,例如从 projectile,您需要获取 `Target` 的 `ASC` 并使用其函数之一来 `ApplyGameplayEffectToSelf`。

您可以通过绑定到 `ASC` 的委托来监听任何 `Duration` 或 `Infinite` `GameplayEffects` 何时被应用到 `ASC`:
```c++
AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(this, &APACharacterBase::OnActiveGameplayEffectAddedCallback);
```
回调函数:
```c++
virtual void OnActiveGameplayEffectAddedCallback(UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);
```

无论复制模式如何,服务器始终会调用此函数。自主代理只会在 `Full` 和 `Mixed` 复制模式下为复制的 `GameplayEffects` 调用此函数。模拟代理只会在 `Full` [复制模式](#concepts-asc-rm)下调用此函数。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-removing"></a>
#### 4.5.3 移除 Gameplay Effects
`GameplayEffects` 可以通过多种方式从 [`GameplayAbilities`](#concepts-ga) 的函数和 `ASC` 的函数移除,通常采用 `RemoveActiveGameplayEffect` 的形式。不同的函数本质上是便捷函数,最终将在 `Target` 上调用 `FActiveGameplayEffectsContainer::RemoveActiveEffects()`。

要在 `GameplayAbility` 之外移除 `GameplayEffects`,您需要获取 `Target` 的 `ASC` 并使用其函数之一来 `RemoveActiveGameplayEffect`。

您可以通过绑定到 `ASC` 的委托来监听任何 `Duration` 或 `Infinite` `GameplayEffects` 何时从 `ASC` 移除:
```c++
AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate().AddUObject(this, &APACharacterBase::OnRemoveGameplayEffectCallback);
```
回调函数:
```c++
virtual void OnRemoveGameplayEffectCallback(const FActiveGameplayEffect& EffectRemoved);
```

无论复制模式如何,服务器始终会调用此函数。自主代理只会在 `Full` 和 `Mixed` 复制模式下为复制的 `GameplayEffects` 调用此函数。模拟代理只会在 `Full` [复制模式](#concepts-asc-rm)下调用此函数。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-mods"></a>
#### 4.5.4 Gameplay Effect Modifiers
`Modifiers` 改变 `Attribute` 并且是[预测性地](#concepts-p)更改 `Attribute` 的唯一方法。`GameplayEffect` 可以有零个或多个 `Modifiers`。每个 `Modifier` 负责通过指定的操作仅改变一个 `Attribute`。

| Operation  | Description                                                                                                         |
| ---------- | ------------------------------------------------------------------------------------------------------------------- |
| `Add`      | 将结果加到 `Modifier` 指定的 `Attribute`。使用负值进行减法。                    |
| `Multiply` | 将结果乘到 `Modifier` 指定的 `Attribute`。                                                    |
| `Divide`   | 将结果除到 `Modifier` 指定的 `Attribute`。                                                  |
| `Override` | 用结果覆盖 `Modifier` 指定的 `Attribute`。                                                   |

`Attribute` 的 `CurrentValue` 是其所有 `Modifiers` 加到其 `BaseValue` 的聚合结果。`Modifiers` 的聚合公式在 `GameplayEffectAggregator.cpp` 中的 `FAggregatorModChannel::EvaluateWithBase` 中定义如下:
```c++
((InlineBaseValue + Additive) * Multiplicitive) / Division
```

任何 `Override` `Modifiers` 都会覆盖最终值,最后应用的 `Modifier` 优先。

**注意:**对于基于百分比的变化,请确保使用 `Multiply` 操作,以便在加法之后进行。

**注意:**[预测](#concepts-p)在百分比变化方面存在问题。

`Modifiers` 有四种类型:Scalable Float、Attribute Based、Custom Calculation Class 和 Set By Caller。它们都生成一些浮点值,然后根据其操作用于更改 `Modifier` 指定的 `Attribute`。

| `Modifier` Type            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Scalable Float`           | `FScalableFloats` 是一种可以指向 Data Table 的结构,该 Data Table 以变量为行,以等级为列。Scalable Floats 将自动在 ability 的当前等级(或在 [`GameplayEffectSpec`](#concepts-ge-spec) 上覆盖时的不同等级)读取指定表行的值。此值可以由系数进一步操作。如果未指定 Data Table/Row,它将值视为 1,以便系数可用于在所有等级硬编码单个值。 ![ScalableFloat](https://github.com/tranek/GASDocumentation/raw/master/Images/scalablefloats.png)                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `Attribute Based`          | `Attribute Based` `Modifiers` 获取 `Source`(创建 `GameplayEffectSpec` 的人)或 `Target`(接收 `GameplayEffectSpec` 的人)上的 backing `Attribute` 的 `CurrentValue` 或 `BaseValue`,并通过系数和系数前后加法进一步修改它。`Snapshotting` 意味着在创建 `GameplayEffectSpec` 时捕获 backing `Attribute`,而不进行快照意味着在应用 `GameplayEffectSpec` 时捕获 `Attribute`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `Custom Calculation Class` | `Custom Calculation Class` 为复杂的 `Modifiers` 提供最大的灵活性。此 `Modifier` 采用 [`ModifierMagnitudeCalculation`](#concepts-ge-mmc) 类,并可以通过系数和系数前后加法进一步操作生成的浮点值。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `Set By Caller`            | `SetByCaller` `Modifiers` 是由 ability 或在 `GameplayEffectSpec` 上创建 `GameplayEffectSpec` 的人在运行时在 `GameplayEffect` 之外设置的值。例如,如果您希望将伤害设置为基于玩家按住按钮以充能 ability 的时间,则可以使用 `SetByCaller`。`SetByCallers` 本质上是 `TMap<FGameplayTag, float>`,位于 `GameplayEffectSpec` 上。`Modifier` 只是告诉 `Aggregator` 查找与提供的 `GameplayTag` 关联的 `SetByCaller` 值。`Modifiers` 使用的 `SetByCallers` 只能使用该概念的 `GameplayTag` 版本。这里禁用了 `FName` 版本。如果 `Modifier` 设置为 `SetByCaller`,但 `GameplayEffectSpec` 上不存在具有正确 `GameplayTag` 的 `SetByCaller`,游戏将抛出运行时错误并返回值 0。在 `Divide` 操作的情况下,这可能会导致问题。有关如何使用 `SetByCallers` 的更多信息,请参阅 [`SetByCallers`](#concepts-ge-spec-setbycaller)。 |

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-mods-multiplydivide"></a>
##### 4.5.4.1 Multiply 和 Divide Modifiers
默认情况下,所有 `Multiply` 和 `Divide` `Modifiers` 在乘以或除以 `Attribute` 的 `BaseValue` 之前会先加在一起。

```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Additive = SumMods(Mods[EGameplayModOp::Additive], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Additive), Parameters);
	float Multiplicitive = SumMods(Mods[EGameplayModOp::Multiplicitive], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Multiplicitive), Parameters);
	float Division = SumMods(Mods[EGameplayModOp::Division], GameplayEffectUtilities::GetModifierBiasByModifierOp(EGameplayModOp::Division), Parameters);
	...
	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
	...
}
```

```c++
float FAggregatorModChannel::SumMods(const TArray<FAggregatorMod>& InMods, float Bias, const FAggregatorEvaluateParameters& Parameters)
{
	float Sum = Bias;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Sum += (Mod.EvaluatedMagnitude - Bias);
		}
	}

	return Sum;
}
```
*来自 `GameplayEffectAggregator.cpp`*

在此公式中,`Multiply` 和 `Divide` `Modifiers` 的 `Bias` 值均为 `1`(`Addition` 的 `Bias` 为 `0`)。因此,它看起来像这样:

```
1 + (Mod1.Magnitude - 1) + (Mod2.Magnitude - 1) + ...
```

此公式会导致一些意想不到的结果。首先,此公式在将 modifiers 乘以或除以 `BaseValue` 之前将所有 modifiers 加在一起。大多数人会期望它们相乘或相除。例如,如果您有两个 `1.5` 的 `Multiply` modifiers,大多数人会期望 `BaseValue` 乘以 `1.5 x 1.5 = 2.25`。相反,这将 `1.5` 加在一起,使 `BaseValue` 乘以 `2`(`增加 50% + 再增加 50% = 增加 100%`)。这是为了 `GameplayPrediction.h` 中的示例,在 `500` 基础速度上 `10%` 的速度增益将是 `550`。再加一个 `10%` 的速度增益,它将是 `600`。

其次,此公式有一些关于可以使用哪些值的未记录规则,因为它是考虑到 Paragon 而设计的。

`Multiply` 和 `Divide` 乘法加法公式的规则:
* `(不超过一个值 < 1) AND (任意数量的值 [1, 2))`
* `OR (一个值 >= 2)`

公式中的 `Bias` 基本上减去了 `[1, 2)` 范围内数字的整数位。第一个 `Modifier` 的 `Bias` 从起始 `Sum` 值中减去(在循环之前设置为 `Bias`),这就是为什么任何值本身都可以工作,以及为什么一个值 `< 1` 可以与 `[1, 2)` 范围内的数字一起工作。

`Multiply` 的一些示例:
乘数: `0.5`
`1 + (0.5 - 1) = 0.5`,正确

乘数: `0.5, 0.5`
`1 + (0.5 - 1) + (0.5 - 1) = 0`,不正确,期望 `1`?小于 `1` 的多个值对于加法乘数没有意义。Paragon 被设计为仅使用 [`Multiply` `Modifiers` 的最大负值](#cae-nonstackingge),因此最多只会有一个小于 `1` 的值乘以 `BaseValue`。

乘数: `1.1, 0.5`
`1 + (0.5 - 1) + (1.1 - 1) = 0.6`,正确

乘数: `5, 5`
`1 + (5 - 1) + (5 - 1) = 9`,不正确,期望 `10`。将始终是 `Modifiers 的总和 - Modifiers 的数量 + 1`。

许多游戏会希望它们的 `Multiply` 和 `Divide` `Modifiers` 在应用到 `BaseValue` 之前先相乘和相除。要实现这一点,您需要**更改引擎代码**中的 `FAggregatorModChannel::EvaluateWithBase()`。

```c++
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Multiplicitive = MultiplyMods(Mods[EGameplayModOp::Multiplicitive], Parameters);
	float Division = MultiplyMods(Mods[EGameplayModOp::Division], Parameters);
	...

	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
}
```

```c++
float FAggregatorModChannel::MultiplyMods(const TArray<FAggregatorMod>& InMods, const FAggregatorEvaluateParameters& Parameters)
{
	float Multiplier = 1.0f;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Multiplier *= Mod.EvaluatedMagnitude;
		}
	}

	return Multiplier;
}
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-mods-gameplaytags"></a>
##### 4.5.4.2 Modifiers 上的 Gameplay Tags

可以为每个 [Modifier](#concepts-ge-mods) 设置 `SourceTags` 和 `TargetTags`。它们的工作方式与 `GameplayEffect` 的 [`Application Tag requirements`](#concepts-ge-tags) 相同。因此,标签仅在应用效果时被考虑。例如,当具有周期性、无限效果时,它们仅在效果的首次应用时被考虑,而*不会*在每次周期执行时被考虑。

`Attribute Based` Modifiers 还可以设置 `SourceTagFilter` 和 `TargetTagFilter`。在确定作为 `Attribute Based` Modifier 源的属性的大小时,这些过滤器用于排除对该属性的某些 Modifiers。源或目标没有过滤器所有标签的 Modifiers 将被排除。

这详细意味着:`GameplayEffects` 捕获源 ASC 和目标 ASC 的标签。在创建 `GameplayEffectSpec` 时捕获源 ASC 标签,在执行效果时捕获目标 ASC 标签。在确定无限或持续效果的 Modifier 是否"有资格"应用(即其 Aggregator 有资格)并设置了这些过滤器时,会将捕获的标签与过滤器进行比较。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-stacking"></a>
#### 4.5.5 堆叠 Gameplay Effects
默认情况下,`GameplayEffects` 将应用 `GameplayEffectSpec` 的新实例,这些实例不知道或关心之前存在的 `GameplayEffectSpec` 实例。`GameplayEffects` 可以设置为堆叠,即不添加 `GameplayEffectSpec` 的新实例,而是更改当前存在的 `GameplayEffectSpec` 的堆叠计数。堆叠仅适用于 `Duration` 和 `Infinite` `GameplayEffects`。

有两种堆叠类型:Aggregate by Source 和 Aggregate by Target。

| Stacking Type       | Description                                                                                                                          |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Aggregate by Source | 在 Target 上每个 Source `ASC` 有一个单独的堆叠实例。每个 Source 可以应用 X 数量的堆叠。                     |
| Aggregate by Target | 在 Target 上只有一个堆叠实例,无论 Source 如何。每个 Source 可以应用一个堆叠,直到达到共享堆叠限制。 |

堆叠还有过期、持续时间刷新和周期重置的策略。它们在 `GameplayEffect` Blueprint 中有有用的悬停工具提示。

示例项目包括一个自定义 Blueprint 节点,用于监听 `GameplayEffect` 堆叠更改。HUD UMG Widget 使用它来更新玩家拥有的被动护甲堆叠数量。此 `AsyncTask` 将一直存在,直到手动调用 `EndTask()`,我们在 UMG Widget 的 `Destruct` 事件中执行此操作。参见 `AsyncTaskEffectStackChanged.h/cpp`。

![Listen for GameplayEffect Stack Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-ga"></a>
#### 4.5.6 授予的 Abilities
`GameplayEffects` 可以向 `ASCs` 授予新的 [`GameplayAbilities`](#concepts-ga)。只有 `Duration` 和 `Infinite` `GameplayEffects` 可以授予 abilities。

一个常见的用例是,当您想要强制另一个玩家执行某些操作时,例如从击退或拉动中移动他们。您可以向他们应用 `GameplayEffect`,向他们授予一个自动激活的 ability(有关在授予时如何自动激活 ability 的信息,请参阅 [Passive Abilities](#concepts-ga-activating-passive)),该 ability 对他们执行所需的操作。

设计人员可以选择 `GameplayEffect` 授予哪些 abilities、授予它们的等级、[绑定到哪个输入](#concepts-ga-input)以及授予的 ability 的移除策略。

| Removal Policy             | Description                                                                                                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cancel Ability Immediately | 当授予它的 `GameplayEffect` 从 Target 移除时,授予的 ability 将立即被取消和移除。                                                   |
| Remove Ability on End      | 允许授予的 ability 完成,然后从 Target 移除。                                                                                                   |
| Do Nothing                 | 授予的 ability 不受从 Target 移除授予 `GameplayEffect` 的影响。Target 永久拥有该 ability,直到以后手动移除。 |

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-tags"></a>
#### 4.5.7 Gameplay Effect Tags
`GameplayEffects` 携带多个 [`GameplayTagContainers`](#concepts-gt)。设计人员将编辑每个类别的 `Added` 和 `Removed` `GameplayTagContainers`,结果将在编译时显示在 `Combined` `GameplayTagContainer` 中。`Added` 标签是此 `GameplayEffect` 添加的、其父级以前没有的新标签。`Removed` 标签是父类拥有但此子类没有的标签。

| Category                          | Description                                                                                                                                                                                                                                                                                                                                                                        |
| --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Gameplay Effect Asset Tags        | `GameplayEffect` 拥有的标签。它们不执行任何功能,仅用于描述 `GameplayEffect`。                                                                                                                                                                                                                                        |
| Granted Tags                      | 位于 `GameplayEffect` 上但也赋予应用了 `GameplayEffect` 的 `ASC` 的标签。当移除 `GameplayEffect` 时,它们将从 `ASC` 中移除。这仅适用于 `Duration` 和 `Infinite` `GameplayEffects`。                                                                                                                             |
| Ongoing Tag Requirements          | 应用后,这些标签确定 `GameplayEffect` 是打开还是关闭。`GameplayEffect` 可以关闭但仍然应用。如果 `GameplayEffect` 由于未满足 Ongoing Tag Requirements 而关闭,但随后满足了这些要求,`GameplayEffect` 将再次打开并重新应用其 modifiers。这仅适用于 `Duration` 和 `Infinite` `GameplayEffects`。 |
| Application Tag Requirements      | Target 上的标签,用于确定是否可以向 Target 应用 `GameplayEffect`。如果不满足这些要求,则不会应用 `GameplayEffect`。                                                                                                                                                                                                                      |
| Remove Gameplay Effects with Tags | Target 上在其 `Asset Tags` 或 `Granted Tags` 中具有任何这些标签的 `GameplayEffects` 将在成功应用此 `GameplayEffect` 时从 Target 移除。                                                                                                                                                                                            |

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-immunity"></a>
#### 4.5.8 免疫
`GameplayEffects` 可以基于 [`GameplayTags`](#concepts-gt) 授予免疫,有效地阻止其他 `GameplayEffects` 的应用。虽然可以通过其他方式(如 `Application Tag Requirements`)有效地实现免疫,但使用此系统提供了一个委托,用于在 `GameplayEffects` 由于免疫而被阻止时使用 `UAbilitySystemComponent::OnImmunityBlockGameplayEffectDelegate`。

`GrantedApplicationImmunityTags` 检查 Source `ASC`(包括来自 Source ability 的 `AbilityTags` 的标签,如果有的话)是否具有任何指定的标签。这是一种基于标签为来自某些角色或源的所有 `GameplayEffects` 提供免疫的方法。

`Granted Application Immunity Query` 检查传入的 `GameplayEffectSpec` 是否匹配任何查询以阻止或允许其应用。

这些查询在 `GameplayEffect` Blueprint 中有有用的悬停工具提示。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-spec"></a>
#### 4.5.9 Gameplay Effect Spec
[`GameplayEffectSpec`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectSpec/index.html) (`GESpec`) 可以被视为 `GameplayEffects` 的实例化。它们持有对它们所代表的 `GameplayEffect` 类的引用、创建它的等级以及创建者。这些可以在应用之前在运行时自由创建和修改,不像 `GameplayEffects` 应该在设计人员运行时之前创建。当应用 `GameplayEffect` 时,会从 `GameplayEffect` 创建 `GameplayEffectSpec`,这实际上是应用到 Target 的内容。

`GameplayEffectSpecs` 使用 `UAbilitySystemComponent::MakeOutgoingSpec()` 从 `GameplayEffects` 创建,该函数是 `BlueprintCallable`。`GameplayEffectSpecs` 不必立即应用。将 `GameplayEffectSpec` 传递给从 ability 创建的 projectile 是很常见的,projectile 可以稍后将其应用到它击中的 target。当成功应用 `GameplayEffectSpecs` 时,它们返回一个名为 `FActiveGameplayEffect` 的新结构体。

值得注意的 `GameplayEffectSpec` 内容:
* 此 `GameplayEffect` 创建自的 `GameplayEffect` 类。
* 此 `GameplayEffectSpec` 的等级。通常与创建 `GameplayEffectSpec` 的 ability 的等级相同,但可以不同。
* `GameplayEffectSpec` 的持续时间。默认为 `GameplayEffect` 的持续时间,但可以不同。
* 周期性效果的 `GameplayEffectSpec` 的周期。默认为 `GameplayEffect` 的周期,但可以不同。
* 此 `GameplayEffectSpec` 的当前堆叠计数。堆叠限制在 `GameplayEffect` 上。
* [`GameplayEffectContextHandle`](#concepts-ge-context) 告诉我们谁创建了这个 `GameplayEffectSpec`。
* 由于快照而在 `GameplayEffectSpec` 创建时捕获的 `Attributes`。
* `GameplayEffectSpec` 除了 `GameplayEffect` 授予的 `GameplayTags` 之外授予 Target 的 `DynamicGrantedTags`。
* `GameplayEffectSpec` 除了 `GameplayEffect` 拥有的 `AssetTags` 之外拥有的 `DynamicAssetTags`。
* `SetByCaller` `TMaps`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-spec-setbycaller"></a>
##### 4.5.9.1 SetByCallers
`SetByCallers` 允许 `GameplayEffectSpec` 携带与 `GameplayTag` 或 `FName` 关联的浮点值。它们存储在各自的 `TMap` 中：`TMap<FGameplayTag, float>` 和 `TMap<FName, float>` 位于 `GameplayEffectSpec` 上。这些可以用作 `GameplayEffect` 上的 `Modifiers`,或作为传递浮点数的通用手段。通常会通过 `SetByCallers` 将 ability 内部生成的数值数据传递给 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 或 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc)。

| `SetByCaller` Use | Notes                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Modifiers`       | 必须在 `GameplayEffect` 类中提前定义。只能使用 `GameplayTag` 版本。如果在 `GameplayEffect` 类上定义了一个,但 `GameplayEffectSpec` 没有相应的标签和浮点值对,游戏将在应用 `GameplayEffectSpec` 时出现运行时错误并返回 0。这对于 `Divide` 操作来说是一个潜在问题。参见 [`Modifiers`](#concepts-ge-mods)。 |
| Elsewhere         | 不需要在任何地方提前定义。读取 `GameplayEffectSpec` 上不存在的 `SetByCaller` 可以返回开发人员定义的默认值,并带有可选警告。                                                                                                                                                                                                                                      |

要在 Blueprint 中分配 `SetByCaller` 值,请使用您需要的版本(`GameplayTag` 或 `FName`)的 Blueprint 节点:

![Assigning SetByCaller](https://github.com/tranek/GASDocumentation/raw/master/Images/setbycaller.png)

要在 Blueprint 中读取 `SetByCaller` 值,您需要在 Blueprint Library 中创建自定义节点。

要在 C++ 中分配 `SetByCaller` 值,请使用您需要的函数版本(`GameplayTag` 或 `FName`):

```c++
void FGameplayEffectSpec::SetSetByCallerMagnitude(FName DataName, float Magnitude);
```
```c++
void FGameplayEffectSpec::SetSetByCallerMagnitude(FGameplayTag DataTag, float Magnitude);
```

要在 C++ 中读取 `SetByCaller` 值,请使用您需要的函数版本(`GameplayTag` 或 `FName`):

```c++
float GetSetByCallerMagnitude(FName DataName, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```
```c++
float GetSetByCallerMagnitude(FGameplayTag DataTag, bool WarnIfNotFound = true, float DefaultIfNotFound = 0.f) const;
```

我建议使用 `GameplayTag` 版本而不是 `FName` 版本。这可以防止 Blueprint 中的拼写错误。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-context"></a>
#### 4.5.10 Gameplay Effect Context
[`GameplayEffectContext`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/FGameplayEffectContext/index.html) 结构体持有关于 `GameplayEffectSpec` 的发起者和 [`TargetData`](#concepts-targeting-data) 的信息。这也是一个很好的结构体,可以子类化以在 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc) / [`GameplayEffectExecutionCalculations`](#concepts-ge-ec)、[`AttributeSets`](#concepts-as) 和 [`GameplayCues`](#concepts-gc) 等地方之间传递任意数据。

要子类化 `GameplayEffectContext`:

1. 子类化 `FGameplayEffectContext`
1. 重写 `FGameplayEffectContext::GetScriptStruct()`
1. 重写 `FGameplayEffectContext::Duplicate()`
1. 如果您的新数据需要复制,则重写 `FGameplayEffectContext::NetSerialize()`
1. 为您的子类实现 `TStructOpsTypeTraits`,就像父结构体 `FGameplayEffectContext` 那样
1. 在您的 [`AbilitySystemGlobals`](#concepts-asg) 类中重写 `AllocGameplayEffectContext()` 以返回您子类的新对象

[GASShooter](https://github.com/tranek/GASShooter) 使用子类化的 `GameplayEffectContext` 来添加 `TargetData`,这可以在 `GameplayCues` 中访问,特别是对于霰弹枪,因为它可以击中多个敌人。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-mmc"></a>
#### 4.5.11 Modifier Magnitude Calculation
[`ModifierMagnitudeCalculations`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayModMagnitudeCalculation/index.html) (`ModMagCalc` 或 `MMC`) 是强大的类,用作 `GameplayEffects` 中的 [`Modifiers`](#concepts-ge-mods)。它们的功能类似于 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec),但功能较弱,最重要的是它们可以被[预测](#concepts-p)。它们的唯一目的是从 `CalculateBaseMagnitude_Implementation()` 返回一个浮点值。您可以在 Blueprint 和 C++ 中子类化并重写此函数。

`MMCs` 可以用于任何持续时间的 `GameplayEffects` - `Instant`、`Duration`、`Infinite` 或 `Periodic`。

`MMCs` 的优势在于它们能够捕获 `GameplayEffect` 的 `Source` 或 `Target` 上任意数量的 `Attributes` 的值,并且完全访问 `GameplayEffectSpec` 以读取 `GameplayTags` 和 `SetByCallers`。`Attributes` 可以快照或不快照。快照的 `Attributes` 在创建 `GameplayEffectSpec` 时捕获,而非快照的 `Attributes` 在应用 `GameplayEffectSpec` 时捕获,并在 `Attribute` 更改时自动更新 `Infinite` 和 `Duration` `GameplayEffects`。捕获 `Attributes` 会从 `ASC` 上的现有 mods 重新计算它们的 `CurrentValue`。这种重新计算将**不会**在 `AbilitySet` 中运行 [`PreAttributeChange()`](#concepts-as-preattributechange),因此必须在此处再次进行任何限制。

| Snapshot | Source or Target | 在 `GameplayEffectSpec` 上捕获 | 当 `Attribute` 为 `Infinite` 或 `Duration` `GE` 更改时自动更新 |
| -------- | ---------------- | -------------------------------- | -------------------------------------------------------------------------------- |
| Yes      | Source           | Creation                         | No                                                                               |
| Yes      | Target           | Application                      | No                                                                               |
| No       | Source           | Application                      | Yes                                                                              |
| No       | Target           | Application                      | Yes                                                                              |

`MMC` 的结果浮点数可以通过系数和系数前后加法在 `GameplayEffect's` `Modifier` 中进一步修改。

下面是一个示例 `MMC`,它捕获 `Target` 的法力 `Attribute`,并从毒药效果中减少它,减少的量取决于 `Target` 有多少法力以及 `Target` 可能拥有的标签:
```c++
UPAMMC_PoisonMana::UPAMMC_PoisonMana()
{

	//ManaDef 定义在头文件中 FGameplayEffectAttributeCaptureDefinition ManaDef;
	ManaDef.AttributeToCapture = UPAAttributeSetBase::GetManaAttribute();
	ManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	ManaDef.bSnapshot = false;

	//MaxManaDef 定义在头文件中 FGameplayEffectAttributeCaptureDefinition MaxManaDef;
	MaxManaDef.AttributeToCapture = UPAAttributeSetBase::GetMaxManaAttribute();
	MaxManaDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Target;
	MaxManaDef.bSnapshot = false;

	RelevantAttributesToCapture.Add(ManaDef);
	RelevantAttributesToCapture.Add(MaxManaDef);
}

float UPAMMC_PoisonMana::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	// 从源和目标收集标签,因为这可以影响应该使用哪些增益
	const FGameplayTagContainer* SourceTags = Spec.CapturedSourceTags.GetAggregatedTags();
	const FGameplayTagContainer* TargetTags = Spec.CapturedTargetTags.GetAggregatedTags();

	FAggregatorEvaluateParameters EvaluationParameters;
	EvaluationParameters.SourceTags = SourceTags;
	EvaluationParameters.TargetTags = TargetTags;

	float Mana = 0.f;
	GetCapturedAttributeMagnitude(ManaDef, Spec, EvaluationParameters, Mana);
	Mana = FMath::Max<float>(Mana, 0.0f);

	float MaxMana = 0.f;
	GetCapturedAttributeMagnitude(MaxManaDef, Spec, EvaluationParameters, MaxMana);
	MaxMana = FMath::Max<float>(MaxMana, 1.0f); // 避免除以零

	float Reduction = -20.0f;
	if (Mana / MaxMana > 0.5f)
	{
		// 如果目标拥有超过一半的法力,则加倍效果
		Reduction *= 2;
	}

	if (TargetTags->HasTagExact(FGameplayTag::RequestGameplayTag(FName("Status.WeakToPoisonMana"))))
	{
		// 如果目标对 PoisonMana 弱,则加倍效果
		Reduction *= 2;
	}

	return Reduction;
}
```

如果您没有将 `FGameplayEffectAttributeCaptureDefinition` 添加到 `MMC's` 构造函数中的 `RelevantAttributesToCapture` 并尝试捕获 `Attributes`,则在捕获时会收到关于缺少 Spec 的错误。如果您不需要捕获 `Attributes`,则不必向 `RelevantAttributesToCapture` 添加任何内容。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-ec"></a>
#### 4.5.12 Gameplay Effect Execution Calculation
[`GameplayEffectExecutionCalculations`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffectExecutionCalculat-/index.html) (`ExecutionCalculation`、`Execution`(您会在插件源代码中经常看到此术语)或 `ExecCalc`) 是 `GameplayEffects` 对 `ASC` 进行更改的最强大方式。像 [`ModifierMagnitudeCalculations`](#concepts-ge-mmc) 一样,这些可以捕获 `Attributes` 并可选择快照它们。与 `MMCs` 不同,这些可以更改多个 `Attribute`,并且基本上可以做程序员想要的其他任何事情。这种强大功能和灵活性的缺点是它们不能被[预测](#concepts-p),并且必须在 C++ 中实现。

`ExecutionCalculations` 只能与 `Instant` 和 `Periodic` `GameplayEffects` 一起使用。任何带有 "Execute" 一词的通常都指这两种类型的 `GameplayEffects`。

快照在创建 `GameplayEffectSpec` 时捕获 `Attribute`,而不快照在应用 `GameplayEffectSpec` 时捕获 `Attribute`。捕获 `Attributes` 会从 `ASC` 上的现有 mods 重新计算它们的 `CurrentValue`。这种重新计算将**不会**在 `AbilitySet` 中运行 [`PreAttributeChange()`](#concepts-as-preattributechange),因此必须在此处再次进行任何限制。

| Snapshot | Source or Target | Captured on `GameplayEffectSpec` |
| -------- | ---------------- | -------------------------------- |
| Yes      | Source           | Creation                         |
| Yes      | Target           | Application                      |
| No       | Source           | Application                      |
| No       | Target           | Application                      |

要设置 `Attribute` 捕获,我们遵循 Epic 的 ActionRPG 示例项目设置的模式,通过定义一个结构体来保存和定义我们如何捕获 `Attributes`,并在结构体的构造函数中创建一个副本。您将为每个 `ExecCalc` 拥有这样一个结构体。**注意:**每个结构体需要一个唯一的名称,因为它们共享相同的命名空间。对结构体使用相同的名称将导致在捕获您的 `Attributes` 时出现不正确的行为(主要是捕获错误的 `Attributes` 的值)。

对于 `Local Predicted`、`Server Only` 和 `Server Initiated` [`GameplayAbilities`](#concepts-ga),`ExecCalc` 仅在服务器上调用。

基于从 `Source` 和 `Target` 上的多个属性读取的复杂公式来计算受到的伤害是 `ExecCalc` 最常见的例子。包含的示例项目有一个简单的 `ExecCalc` 用于计算伤害,它从 `GameplayEffectSpec's` [`SetByCaller`](#concepts-ge-spec-setbycaller) 读取伤害值,然后根据从 `Target` 捕获的护甲 `Attribute` 减轻该值。参见 `GDDamageExecCalculation.cpp/.h`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-ec-senddata"></a>
##### 4.5.12.1 发送数据到 Execution Calculations
除了捕获 `Attributes` 之外,还有几种方法可以将数据发送到 `ExecutionCalculation`。

<a name="concepts-ge-ec-senddata-setbycaller"></a>
###### 4.5.12.1.1 SetByCaller
在 `GameplayEffectSpec` 上设置的任何 [`SetByCallers`](#concepts-ge-spec-setbycaller) 都可以在 `ExecutionCalculation` 中直接读取。

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
float Damage = FMath::Max<float>(Spec.GetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName("Data.Damage")), false, -1.0f), 0.0f);
```

<a name="concepts-ge-ec-senddata-backingdataattribute"></a>
###### 4.5.12.1.2 Backing Data Attribute Calculation Modifier
如果您想要将硬编码的值传递给 `GameplayEffect`,您可以使用一个 `CalculationModifier` 来传递它们,该修饰符使用捕获的 `Attributes` 之一作为支持数据。

在此截图示例中,我们将 50 添加到捕获的 Damage `Attribute`。您也可以将其设置为 `Override` 以仅接收硬编码的值。

![Backing Data Attribute Calculation Modifier](https://github.com/tranek/GASDocumentation/raw/master/Images/calculationmodifierbackingdataattribute.png)

`ExecutionCalculation` 在捕获 `Attribute` 时读取此值。

```c++
float Damage = 0.0f;
// 捕获在伤害 GE 上作为 CalculationModifier 设置的可选伤害值,位于 ExecutionCalculation 下
ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(DamageStatics().DamageDef, EvaluationParameters, Damage);
```

<a name="concepts-ge-ec-senddata-backingdatatempvariable"></a>
###### 4.5.12.1.3 Backing Data Temporary Variable Calculation Modifier
如果您想要将硬编码的值传递给 `GameplayEffect`,您可以使用一个 `CalculationModifier` 来传递它们,该修饰符使用 `Temporary Variable` 或在 C++ 中称为 `Transient Aggregator`。`Temporary Variable` 与一个 `GameplayTag` 相关联。

在此截图示例中,我们使用 `Data.Damage` `GameplayTag` 将 50 添加到 `Temporary Variable`。

![Backing Data Temporary Variable Calculation Modifier](https://github.com/tranek/GASDocumentation/raw/master/Images/calculationmodifierbackingdatatempvariable.png)

将支持的 `Temporary Variables` 添加到您的 `ExecutionCalculation` 的构造函数中:

```c++
ValidTransientAggregatorIdentifiers.AddTag(FGameplayTag::RequestGameplayTag("Data.Damage"));
```

`ExecutionCalculation` 使用类似于 `Attribute` 捕获函数的特殊捕获函数来读取此值。

```c++
float Damage = 0.0f;
ExecutionParams.AttemptCalculateTransientAggregatorMagnitude(FGameplayTag::RequestGameplayTag("Data.Damage"), EvaluationParameters, Damage);
```

<a name="concepts-ge-ec-senddata-effectcontext"></a>
###### 4.5.12.1.4 Gameplay Effect Context
您可以通过 `GameplayEffectSpec` 上的自定义 [`GameplayEffectContext`](#concepts-ge-context) 将数据发送到 `ExecutionCalculation`。

在 `ExecutionCalculation` 中,您可以从 `FGameplayEffectCustomExecutionParameters` 访问 `EffectContext`。

```c++
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(Spec.GetContext().Get());
```

如果您需要更改 `GameplayEffectSpec` 或 `EffectContext` 上的某些内容:

```c++
FGameplayEffectSpec* MutableSpec = ExecutionParams.GetOwningSpecForPreExecuteMod();
FGSGameplayEffectContext* ContextHandle = static_cast<FGSGameplayEffectContext*>(MutableSpec->GetContext().Get());
```

如果在 `ExecutionCalculation` 中修改 `GameplayEffectSpec`,请谨慎操作。请参阅 `GetOwningSpecForPreExecuteMod()` 的注释。

```c++
/** 非 const 访问。请小心使用,特别是在捕获属性后修改 spec 时。 */
FGameplayEffectSpec* GetOwningSpecForPreExecuteMod() const;
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-car"></a>
#### 4.5.13 Custom Application Requirement
[`CustomApplicationRequirement`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayEffectCustomApplication-/index.html) (`CAR`) 类为设计人员提供了高级控制,以决定是否可以应用 `GameplayEffect`,而不是简单地检查 `GameplayEffect` 上的 `GameplayTag`。这些可以通过重写 `CanApplyGameplayEffect()` 在 Blueprint 中实现,也可以通过重写 `CanApplyGameplayEffect_Implementation()` 在 C++ 中实现。

何时使用 `CARs` 的示例:
* `Target` 需要拥有一定数量的 `Attribute`
* `Target` 需要拥有一定数量的 `GameplayEffect` 堆叠

`CARs` 还可以做更高级的事情,比如检查此 `GameplayEffect` 的实例是否已经存在于 `Target` 上,并[更改现有实例的持续时间](#concepts-ge-duration),而不是应用新实例(对 `CanApplyGameplayEffect()` 返回 false)。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-cost"></a>
#### 4.5.14 Cost Gameplay Effect
[`GameplayAbilities`](#concepts-ga) 有一个可选的 `GameplayEffect`,专门设计用作 ability 的消耗。消耗是指 `ASC` 需要拥有多少 `Attribute` 才能够激活 `GameplayAbility`。如果 `GA` 无法支付 `Cost GE`,那么它们将无法激活。此 `Cost GE` 应该是一个 `Instant` `GameplayEffect`,带有一个或多个从 `Attributes` 中减去的 `Modifiers`。默认情况下,`Cost GEs` 旨在可预测,建议保持此能力,即不要使用 `ExecutionCalculations`。`MMCs` 对于复杂的消耗计算是完全可接受且鼓励的。

开始时,您很可能为每个具有消耗的 `GA` 拥有一个唯一的 `Cost GE`。一个更高级的技术是为多个 `GA` 重用一个 `Cost GE`,只需使用 `GA` 特定的数据修改从 `Cost GE` 创建的 `GameplayEffectSpec`(消耗值在 `GA` 上定义)。**这仅适用于 `Instanced` abilities。**

重用 `Cost GE` 的两种技术:

1. **使用 `MMC`。** 这是最简单的方法。创建一个 [`MMC`](#concepts-ge-mmc),从 `GameplayAbility` 实例读取消耗值,您可以从 `GameplayEffectSpec` 获取该实例。

```c++
float UPGMMC_HeroAbilityCost::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->Cost.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

在此示例中,消耗值是我添加到 `GameplayAbility` 子类上的 `FScalableFloat`。
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cost")
FScalableFloat Cost;
```

![Cost GE With MMC](https://github.com/tranek/GASDocumentation/raw/master/Images/costmmc.png)

2. **重写 `UGameplayAbility::GetCostGameplayEffect()`。** 重写此函数并[在运行时创建 `GameplayEffect`](#concepts-ge-dynamic),该效果读取 `GameplayAbility` 上的消耗值。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-cooldown"></a>
#### 4.5.15 Cooldown Gameplay Effect
[`GameplayAbilities`](#concepts-ga) 有一个可选的 `GameplayEffect`,专门设计用作 ability 的冷却时间。冷却时间决定 ability 在激活后多久可以再次激活。如果 `GA` 仍在冷却中,则无法激活。此 `Cooldown GE` 应该是一个 `Duration` `GameplayEffect`,不带 `Modifiers`,并且在 `GameplayEffect's` 的 `GrantedTags` 中每个 `GameplayAbility` 或每个 ability 槽位(如果您的游戏有分配给共享冷却时间的槽位的可互换 abilities)都有一个唯一的 `GameplayTag`("`Cooldown Tag`")。`GA` 实际上检查的是 `Cooldown Tag` 的存在,而不是 `Cooldown GE` 的存在。默认情况下,`Cooldown GEs` 旨在可预测,建议保持此能力,即不要使用 `ExecutionCalculations`。`MMCs` 对于复杂的冷却计算是完全可接受且鼓励的。

开始时,您很可能为每个具有冷却时间的 `GA` 拥有一个唯一的 `Cooldown GE`。一个更高级的技术是为多个 `GA` 重用一个 `Cooldown GE`,只需使用 `GA` 特定的数据修改从 `Cooldown GE` 创建的 `GameplayEffectSpec`(冷却持续时间和 `Cooldown Tag` 在 `GA` 上定义)。**这仅适用于 `Instanced` abilities。**

重用 `Cooldown GE` 的两种技术:

1. **使用 [`SetByCaller`](#concepts-ge-spec-setbycaller)。** 这是最简单的方法。将共享的 `Cooldown GE` 的持续时间设置为使用 `GameplayTag` 的 `SetByCaller`。在您的 `GameplayAbility` 子类上,为持续时间定义一个 float / `FScalableFloat`,为唯一的 `Cooldown Tag` 定义一个 `FGameplayTagContainer`,以及一个临时的 `FGameplayTagContainer`,我们将用它作为 `Cooldown Tag` 和 `Cooldown GE's` 标签的并集的返回指针。
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// 我们将在 GetCooldownTags() 中返回指针的临时容器。
// 这将是我们的 CooldownTags 和 Cooldown GE 的冷却标签的并集。
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

然后重写 `UGameplayAbility::GetCooldownTags()` 以返回我们的 `Cooldown Tags` 和任何现有 `Cooldown GE's` 标签的并集。
```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset(); // MutableTags 写入 CDO 上的 TempCooldownTags,所以请清除它,以防 ability 冷却标签更改(移动到不同的槽位)
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

最后,重写 `UGameplayAbility::ApplyCooldown()` 以注入我们的 `Cooldown Tags` 并将 `SetByCaller` 添加到冷却 `GameplayEffectSpec`。
```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		SpecHandle.Data.Get()->SetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName(  OurSetByCallerTag  )), CooldownDuration.GetValueAtLevel(GetAbilityLevel()));
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

在此图中,冷却的持续时间 `Modifier` 被设置为使用 `Data.Cooldown` 的 `Data Tag` 的 `SetByCaller`。`Data.Cooldown` 将是上面代码中的 `OurSetByCallerTag`。

![Cooldown GE with SetByCaller](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownsbc.png)

2. **使用 [`MMC`](#concepts-ge-mmc)。** 这与上面的设置相同,除了不将 `SetByCaller` 设置为 `Cooldown GE` 和 `ApplyCooldown` 上的持续时间。相反,将持续时间设置为 `Custom Calculation Class` 并指向我们将创建的新 `MMC`。
```c++
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// 我们将在 GetCooldownTags() 中返回指针的临时容器。
// 这将是我们的 CooldownTags 和 Cooldown GE 的冷却标签的并集。
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

然后重写 `UGameplayAbility::GetCooldownTags()` 以返回我们的 `Cooldown Tags` 和任何现有 `Cooldown GE's` 标签的并集。
```c++
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset(); // MutableTags writes to the TempCooldownTags on the CDO so clear it in case the ability cooldown tags change (moved to a different slot)
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

最后,重写 `UGameplayAbility::ApplyCooldown()` 以注入我们的 `Cooldown Tags` 到冷却 `GameplayEffectSpec` 中。
```c++
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

```c++
float UPGMMC_HeroAbilityCooldown::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->CooldownDuration.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

![Cooldown GE with MMC](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownmmc.png)

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-cooldown-tr"></a>
##### 4.5.15.1 获取 Cooldown Gameplay Effect 的剩余时间
```c++
bool APGPlayerState::GetCooldownRemainingForTag(FGameplayTagContainer CooldownTags, float & TimeRemaining, float & CooldownDuration)
{
	if (AbilitySystemComponent && CooldownTags.Num() > 0)
	{
		TimeRemaining = 0.f;
		CooldownDuration = 0.f;

		FGameplayEffectQuery const Query = FGameplayEffectQuery::MakeQuery_MatchAnyOwningTags(CooldownTags);
		TArray< TPair<float, float> > DurationAndTimeRemaining = AbilitySystemComponent->GetActiveEffectsTimeRemainingAndDuration(Query);
		if (DurationAndTimeRemaining.Num() > 0)
		{
			int32 BestIdx = 0;
			float LongestTime = DurationAndTimeRemaining[0].Key;
			for (int32 Idx = 1; Idx < DurationAndTimeRemaining.Num(); ++Idx)
			{
				if (DurationAndTimeRemaining[Idx].Key > LongestTime)
				{
					LongestTime = DurationAndTimeRemaining[Idx].Key;
					BestIdx = Idx;
				}
			}

			TimeRemaining = DurationAndTimeRemaining[BestIdx].Key;
			CooldownDuration = DurationAndTimeRemaining[BestIdx].Value;

			return true;
		}
	}

	return false;
}
```

**注意:** 在客户端查询冷却时间的剩余时间要求它们能够接收复制的 `GameplayEffects`。这将取决于它们的 `ASC's` [复制模式](#concepts-asc-rm)。

<a name="concepts-ge-cooldown-listen"></a>
##### 4.5.15.2 监听冷却的开始和结束
要监听冷却何时开始,您可以通过绑定到 `AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf` 来响应 `Cooldown GE` 的应用,或者通过绑定到 `AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)` 来响应 `Cooldown Tag` 的添加。我建议监听 `Cooldown GE` 的添加,因为您还可以访问应用它的 `GameplayEffectSpec`。从中您可以确定 `Cooldown GE` 是本地预测的还是服务器的更正。

要监听冷却何时结束,您可以通过绑定到 `AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate()` 来响应 `Cooldown GE` 的移除,或者通过绑定到 `AbilitySystemComponent->RegisterGameplayTagEvent(CooldownTag, EGameplayTagEventType::NewOrRemoved)` 来响应 `Cooldown Tag` 的移除。我建议监听 `Cooldown Tag` 的移除,因为当服务器的更正 `Cooldown GE` 到来时,它将移除我们本地预测的,导致 `OnAnyGameplayEffectRemovedDelegate()` 触发,即使我们仍在冷却中。在移除预测的 `Cooldown GE` 和应用服务器的更正 `Cooldown GE` 期间,`Cooldown Tag` 不会更改。

**注意:**在客户端上监听 `GameplayEffect` 的添加或移除要求它们能够接收复制的 `GameplayEffects`。这将取决于它们的 `ASC's` [复制模式](#concepts-asc-rm)。

示例项目包括一个自定义 Blueprint 节点,用于监听冷却的开始和结束。HUD UMG Widget 使用它来更新 Meteor 冷却的剩余时间。此 `AsyncTask` 将一直存在,直到手动调用 `EndTask()`,我们在 UMG Widget 的 `Destruct` 事件中执行此操作。参见 [`AsyncTaskCooldownChanged.h/cpp`](Source/GASDocumentation/Private/Characters/Abilities/AsyncTaskCooldownChanged.cpp)。

![Listen for Cooldown Change BP Node](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

<a name="concepts-ge-cooldown-prediction"></a>
##### 4.5.15.3 预测冷却时间
冷却时间目前实际上无法预测。我们可以在应用本地预测的 `Cooldown GE` 时启动 UI 冷却计时器,但 `GameplayAbility's` 的实际冷却时间与服务器的冷却时间的剩余时间绑定。根据玩家的延迟,本地预测的冷却时间可能到期,但 `GameplayAbility` 仍将在服务器上处于冷却状态,这将阻止 `GameplayAbility's` 立即重新激活,直到服务器的冷却时间到期。

示例项目通过在本地预测的冷却开始时将 Meteor ability 的 UI 图标灰显,然后在服务器的更正 `Cooldown GE` 到来时启动冷却计时器来处理此问题。

这种情况的一个游戏后果是,具有高延迟的玩家在短冷却 abilities 上的射速比具有低延迟的玩家更低,从而使他们处于不利地位。《堡垒之夜》通过武器具有自定义记账来避免这种情况,这些记账不使用冷却 `GameplayEffects`。

允许真正的预测冷却时间(玩家可以在本地冷却时间到期时激活 `GameplayAbility`,但服务器仍处于冷却状态)是 Epic 有朝一日希望在 [GAS 的未来迭代](#concepts-p-future) 中实现的东西。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-duration"></a>
#### 4.5.16 更改活动的 Gameplay Effect 持续时间
要更改 `Cooldown GE` 或任何 `Duration` `GameplayEffect` 的剩余时间,我们需要更改 `GameplayEffectSpec's` 的 `Duration`,更新其 `StartServerWorldTime`,更新其 `CachedStartServerWorldTime`,更新其 `StartWorldTime`,并使用 `CheckDuration()` 重新运行持续时间的检查。在服务器上执行此操作并将 `FActiveGameplayEffect` 标记为脏将把更改复制到客户端。
**注意:**这确实涉及 `const_cast`,可能不是 Epic 更改持续时间的预期方式,但到目前为止它似乎工作良好。

```c++
bool UPAAbilitySystemComponent::SetGameplayEffectDurationHandle(FActiveGameplayEffectHandle Handle, float NewDuration)
{
	if (!Handle.IsValid())
	{
		return false;
	}

	const FActiveGameplayEffect* ActiveGameplayEffect = GetActiveGameplayEffect(Handle);
	if (!ActiveGameplayEffect)
	{
		return false;
	}

	FActiveGameplayEffect* AGE = const_cast<FActiveGameplayEffect*>(ActiveGameplayEffect);
	if (NewDuration > 0)
	{
		AGE->Spec.Duration = NewDuration;
	}
	else
	{
		AGE->Spec.Duration = 0.01f;
	}

	AGE->StartServerWorldTime = ActiveGameplayEffects.GetServerWorldTime();
	AGE->CachedStartServerWorldTime = AGE->StartServerWorldTime;
	AGE->StartWorldTime = ActiveGameplayEffects.GetWorldTime();
	ActiveGameplayEffects.MarkItemDirty(*AGE);
	ActiveGameplayEffects.CheckDuration(Handle);

	AGE->EventSet.OnTimeChanged.Broadcast(AGE->Handle, AGE->StartWorldTime, AGE->GetDuration());
	OnGameplayEffectDurationChange(*AGE);

	return true;
}
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-dynamic"></a>
#### 4.5.17 在运行时创建动态 Gameplay Effects
在运行时创建动态 `GameplayEffects` 是一个高级主题。您不应该经常这样做。

只有 `Instant` `GameplayEffects` 可以在 C++ 中从零开始在运行时创建。`Duration` 和 `Infinite` `GameplayEffects` 无法在运行时动态创建,因为当它们复制时,它们会寻找不存在的 `GameplayEffect` 类定义。要实现此功能,您应该像通常在编辑器中所做的那样创建一个原型 `GameplayEffect` 类。然后在运行时使用您需要的内容自定义 `GameplayEffectSpec` 实例。

在运行时创建的 `Instant` `GameplayEffects` 也可以从[本地预测](#concepts-p)的 `GameplayAbility` 中调用。但是,目前尚不清楚动态创建是否会带来副作用。

##### 示例

示例项目创建了一个动态效果,当角色在其 `AttributeSet` 中受到致命一击时,将黄金和经验点发送给击杀者。

```c++
// 创建一个动态即时 Gameplay Effect 来给予赏金
UGameplayEffect* GEBounty = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Bounty")));
GEBounty->DurationPolicy = EGameplayEffectDurationType::Instant;

int32 Idx = GEBounty->Modifiers.Num();
GEBounty->Modifiers.SetNum(Idx + 2);

FGameplayModifierInfo& InfoXP = GEBounty->Modifiers[Idx];
InfoXP.ModifierMagnitude = FScalableFloat(GetXPBounty());
InfoXP.ModifierOp = EGameplayModOp::Additive;
InfoXP.Attribute = UGDAttributeSetBase::GetXPAttribute();

FGameplayModifierInfo& InfoGold = GEBounty->Modifiers[Idx + 1];
InfoGold.ModifierMagnitude = FScalableFloat(GetGoldBounty());
InfoGold.ModifierOp = EGameplayModOp::Additive;
InfoGold.Attribute = UGDAttributeSetBase::GetGoldAttribute();

Source->ApplyGameplayEffectToSelf(GEBounty, 1.0f, Source->MakeEffectContext());
```

第二个示例显示了在本地预测的 `GameplayAbility` 中创建的运行时 `GameplayEffect`。使用风险自负(请参阅代码中的注释)!

```c++
UGameplayAbilityRuntimeGE::UGameplayAbilityRuntimeGE()
{
	NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::LocalPredicted;
}

void UGameplayAbilityRuntimeGE::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
	if (HasAuthorityOrPredictionKey(ActorInfo, &ActivationInfo))
	{
		if (!CommitAbility(Handle, ActorInfo, ActivationInfo))
		{
			EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
		}

		// 在运行时创建 GE。
		UGameplayEffect* GameplayEffect = NewObject<UGameplayEffect>(GetTransientPackage(), TEXT("RuntimeInstantGE"));
		GameplayEffect->DurationPolicy = EGameplayEffectDurationType::Instant; // 只有 instant 适用于运行时 GE。

		// 添加一个简单的可缩放 float 修饰符,将 MyAttribute 覆盖为 42。
		// 在实际应用中,使用通过 TriggerEventData 传递的信息。
		const int32 Idx = GameplayEffect->Modifiers.Num();
		GameplayEffect->Modifiers.SetNum(Idx + 1);
		FGameplayModifierInfo& ModifierInfo = GameplayEffect->Modifiers[Idx];
		ModifierInfo.Attribute.SetUProperty(UMyAttributeSet::GetMyModifiedAttribute());
		ModifierInfo.ModifierMagnitude = FScalableFloat(42.f);
		ModifierInfo.ModifierOp = EGameplayModOp::Override;

		// 应用 GE。

		// 在此处创建 GESpec 以避免 ASC 从 GE 类默认对象创建 GESpecs 的行为。
		// 由于我们这里有一个动态 GE,这将创建一个带有基础 GameplayEffect 类的 GESpec,所以我们会丢失我们的修饰符。
		// 注意:目前尚不清楚这里执行的这种 "hack" 是否会有缺点!
		// spec 防止 GE 对象被 GarbageCollector 收集,因为 GE 是 spec 上的 UPROPERTY。
		FGameplayEffectSpec* GESpec = new FGameplayEffectSpec(GameplayEffect, {}, 0.f); // "new",因为生命周期由句柄内的共享 ptr 管理
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, FGameplayEffectSpecHandle(GESpec));
	}
	EndAbility(Handle, ActorInfo, ActivationInfo, false, false);
}
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ge-containers"></a>
#### 4.5.18 Gameplay Effect Containers
Epic 的 [Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg) 实现了一个称为 `FGameplayEffectContainer` 的结构。这些不在原版 GAS 中,但对于包含 `GameplayEffects` 和 [`TargetData`](#concepts-targeting-data) 非常有用。它自动化了一些工作,比如从 `GameplayEffects` 创建 `GameplayEffectSpecs` 并在其 `GameplayEffectContext` 中设置默认值。在 `GameplayAbility` 中创建 `GameplayEffectContainer` 并将其传递给生成的 projectile 非常简单直接。我选择不在包含的示例项目中实现 `GameplayEffectContainers`,以展示如何在原版 GAS 中不使用它们,但我强烈建议您研究它们并考虑将它们添加到您的项目中。

要访问 `GameplayEffectContainers` 内部的 `GESpecs` 以执行添加 `SetByCallers` 等操作,请拆解 `FGameplayEffectContainer` 并通过 `GESpecs` 数组中的索引访问 `GESpec` 引用。这要求您提前知道要访问的 `GESpec` 的索引。

![SetByCaller with a GameplayEffectContainer](https://github.com/tranek/GASDocumentation/raw/master/Images/gecontainersetbycaller.png)

`GameplayEffectContainers` 还包含一种可选的高效[目标选择](#concepts-targeting-containers)方式。

**[⬆ 返回顶部](#table-of-contents)**


<a name="concepts-ga"></a>
### 4.6 Gameplay Abilities

<a name="concepts-ga-definition"></a>
#### 4.6.1 Gameplay Ability Definition
[`GameplayAbilities`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/UGameplayAbility/index.html) (`GA`) 是游戏中 `Actor` 可以执行的任何动作或技能。同一时间可以有多个 `GameplayAbility` 处于激活状态，例如同时冲刺和射击。这些可以使用 Blueprint 或 C++ 来制作。

`GameplayAbilities` 的示例：
* 跳跃
* 冲刺
* 射击
* 每隔 X 秒被动格挡一次攻击
* 使用药水
* 打开门
* 收集资源
* 建造建筑

不应该使用 `GameplayAbilities` 实现的内容：
* 基本移动输入
* 某些 UI 交互 - 不要使用 `GameplayAbility` 来从商店购买物品。

这些不是规则，只是我的建议。你的设计和实现可能会有所不同。

`GameplayAbilities` 带有默认功能，可以具有等级来修改属性的变化量或改变 `GameplayAbility` 的功能。

`GameplayAbilities` 根据 [`Net Execution Policy`](#concepts-ga-net) 在拥有客户端和/或服务器上运行，但不在模拟代理上运行。`Net Execution Policy` 决定了 `GameplayAbility` 是否进行本地[预测](#concepts-p)。它们包括[可选的消耗和冷却 `GameplayEffects`](#concepts-ga-commit)的默认行为。`GameplayAbilities` 使用 [`AbilityTasks`](#concepts-at) 来处理随时间发生的动作，例如等待事件、等待属性变化、等待玩家选择目标，或使用 `Root Motion Source` 移动 `Character`。**模拟客户端不会运行 `GameplayAbilities`**。相反，当服务器运行 ability 时，任何需要在模拟代理上视觉播放的内容（如动画 montages）将通过 `AbilityTasks` 或 [`GameplayCues`](#concepts-gc) 进行复制或 RPC，用于声音和粒子等装饰性内容。

所有 `GameplayAbilities` 都将覆盖其 `ActivateAbility()` 函数以添加游戏逻辑。可以在 `EndAbility()` 中添加额外的逻辑，该逻辑在 `GameplayAbility` 完成或被取消时运行。

简单 `GameplayAbility` 的流程图：
![Simple GameplayAbility Flowchart](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartsimple.png)


更复杂 `GameplayAbility` 的流程图：
![Complex GameplayAbility Flowchart](https://github.com/tranek/GASDocumentation/raw/master/Images/abilityflowchartcomplex.png)

复杂的 ability 可以使用多个相互交互（激活、取消等）的 `GameplayAbilities` 来实现。

<a name="concepts-ga-definition-reppolicy"></a>
##### 4.6.1.1 Replication Policy
不要使用此选项。这个名称具有误导性，你不需要它。默认情况下，[`GameplayAbilitySpecs`](#concepts-ga-spec) 会从服务器复制到拥有客户端。如上所述，**`GameplayAbilities` 不会在模拟代理上运行**。它们使用 `AbilityTasks` 和 `GameplayCues` 来复制或 RPC 视觉变化到模拟代理。Epic 的 Dave Ratti 已经表示他希望[在未来删除此选项](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)。

<a name="concepts-ga-definition-remotecancel"></a>
##### 4.6.1.2 Server Respects Remote Ability Cancellation
这个选项往往会带来麻烦。这意味着如果客户端的 `GameplayAbility` 由于取消或自然完成而结束，它将强制服务器版本结束，无论服务器版本是否完成。后一个问题很重要，特别是对于具有高延迟的玩家使用的本地预测 `GameplayAbilities`。通常你会想要禁用此选项。

<a name="concepts-ga-definition-repinputdirectly"></a>
##### 4.6.1.3 Replicate Input Directly
设置此选项将始终复制输入按下和释放事件到服务器。Epic 建议不要使用此选项，而是依赖内置在现有输入相关 [`AbilityTasks`](#concepts-at) 中的 `Generic Replicated Events`，前提是你已经将[输入绑定到你的 `ASC`](#concepts-ga-input)。

Epic 的注释：
```c++
/** Direct Input state replication. These will be called if bReplicateInputDirectly is true on the ability and is generally not a good thing to use. (Instead, prefer to use Generic Replicated Events). */
UAbilitySystemComponent::ServerSetInputPressed()
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-input"></a>
#### 4.6.2 Binding Input to the ASC
`ASC` 允许你直接将输入动作绑定到它，并在授予 ability 时将这些输入分配给 `GameplayAbilities`。分配给 `GameplayAbilities` 的输入动作会在按下时自动激活这些 `GameplayAbilities`，前提是满足 `GameplayTag` 要求。分配的输入动作是使用响应输入的内置 `AbilityTasks` 所必需的。

除了分配用于激活 `GameplayAbilities` 的输入动作外，`ASC` 还接受通用的 `Confirm` 和 `Cancel` 输入。这些特殊输入由 `AbilityTasks` 用于确认[`Target Actors`](#concepts-targeting-actors)之类的内容或取消它们。

要将输入绑定到 `ASC`，你必须首先创建一个 enum，将输入动作名称转换为字节。enum 名称必须与项目设置中用于输入动作的名称完全匹配。`DisplayName` 不重要。

来自示例项目：
```c++
UENUM(BlueprintType)
enum class EGDAbilityInputID : uint8
{
	// 0 None
	None			UMETA(DisplayName = "None"),
	// 1 Confirm
	Confirm			UMETA(DisplayName = "Confirm"),
	// 2 Cancel
	Cancel			UMETA(DisplayName = "Cancel"),
	// 3 LMB
	Ability1		UMETA(DisplayName = "Ability1"),
	// 4 RMB
	Ability2		UMETA(DisplayName = "Ability2"),
	// 5 Q
	Ability3		UMETA(DisplayName = "Ability3"),
	// 6 E
	Ability4		UMETA(DisplayName = "Ability4"),
	// 7 R
	Ability5		UMETA(DisplayName = "Ability5"),
	// 8 Sprint
	Sprint			UMETA(DisplayName = "Sprint"),
	// 9 Jump
	Jump			UMETA(DisplayName = "Jump")
};
```

如果你的 `ASC` 位于 `Character` 上，则在 `SetupPlayerInputComponent()` 中包含绑定到 `ASC` 的函数：
```c++
// Bind to AbilitySystemComponent
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));
AbilitySystemComponent->BindAbilityActivationToInputComponent(PlayerInputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), AbilityEnumAssetPath, static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

如果你的 `ASC` 位于 `PlayerState` 上，则在 `SetupPlayerInputComponent()` 中存在潜在的竞态条件，即 `PlayerState` 可能尚未复制到客户端。因此，我建议尝试在 `SetupPlayerInputComponent()` 和 `OnRep_PlayerState()` 中绑定输入。`OnRep_PlayerState()` 本身是不够的，因为可能存在这样的情况：当 `PlayerState` 在 `PlayerController` 告知客户端调用创建 `InputComponent` 的 `ClientRestart()` 之前复制时，`Actor` 的 `InputComponent` 可能为 null。示例项目演示了在这两个位置尝试绑定，并使用布尔值控制该过程，使其仅实际绑定输入一次。

**注意：** 在示例项目中，enum 中的 `Confirm` 和 `Cancel` 与项目设置中的输入动作名称（`ConfirmTarget` 和 `CancelTarget`）不匹配，但我们在 `BindAbilityActivationToInputComponent()` 中提供了它们之间的映射。这些是特殊的，因为我们提供了映射，它们不必匹配，但它们可以匹配。enum 中的所有其他输入必须与项目设置中的输入动作名称匹配。

对于只会由一个输入激活的 `GameplayAbilities`（它们将始终存在于相同的"槽位"中，如 MOBA），我更喜欢在我的 `UGameplayAbility` 子类中添加一个变量，可以在其中定义它们的输入。然后我可以在授予 ability 时从 `ClassDefaultObject` 读取它。

<a name="concepts-ga-input-noactivate"></a>
##### 4.6.2.1 Binding to Input without Activating Abilities
如果你不希望你的 `GameplayAbilities` 在按下输入时自动激活，但仍希望将它们绑定到输入以与 `AbilityTasks` 一起使用，你可以向你的 `UGameplayAbility` 子类添加一个新的 bool 变量 `bActivateOnInput`，默认值为 `true`，并覆盖 `UAbilitySystemComponent::AbilityLocalInputPressed()`。

```c++
void UGSAbilitySystemComponent::AbilityLocalInputPressed(int32 InputID)
{
	// Consume the input if this InputID is overloaded with GenericConfirm/Cancel and the GenericConfim/Cancel callback is bound
	// 如果此 InputID 被重载为 GenericConfirm/Cancel 且 GenericConfim/Cancel 回调已绑定，则消费该输入
	if (IsGenericConfirmInputBound(InputID))
	{
		LocalInputConfirm();
		return;
	}

	if (IsGenericCancelInputBound(InputID))
	{
		LocalInputCancel();
		return;
	}

	// ---------------------------------------------------------

	ABILITYLIST_SCOPE_LOCK();
	for (FGameplayAbilitySpec& Spec : ActivatableAbilities.Items)
	{
		if (Spec.InputID == InputID)
		{
			if (Spec.Ability)
			{
				Spec.InputPressed = true;
				if (Spec.IsActive())
				{
					if (Spec.Ability->bReplicateInputDirectly && IsOwnerActorAuthoritative() == false)
					{
						ServerSetInputPressed(Spec.Handle);
					}

					AbilitySpecInputPressed(Spec);

					// Invoke the InputPressed event. This is not replicated here. If someone is listening, they may replicate the InputPressed event to the server.
					// 调用 InputPressed 事件。此处不会复制它。如果有人在监听，他们可能会将 InputPressed 事件复制到服务器。
					InvokeReplicatedEvent(EAbilityGenericReplicatedEvent::InputPressed, Spec.Handle, Spec.ActivationInfo.GetActivationPredictionKey());
				}
				else
				{
					UGSGameplayAbility* GA = Cast<UGSGameplayAbility>(Spec.Ability);
					if (GA && GA->bActivateOnInput)
					{
						// Ability is not active, so try to activate it
						// Ability 未激活，因此尝试激活它
						TryActivateAbility(Spec.Handle);
					}
				}
			}
		}
	}
}
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-granting"></a>
#### 4.6.3 Granting Abilities
将 `GameplayAbility` 授予 `ASC` 会将其添加到 `ASC` 的 `ActivatableAbilities` 列表中，允许其在满足 [`GameplayTag` 要求](#concepts-ga-tags)时随意激活 `GameplayAbility`。

我们在服务器上授予 `GameplayAbilities`，然后服务器会自动将 [`GameplayAbilitySpec`](#concepts-ga-spec) 复制到拥有客户端。其他客户端/模拟代理不会收到 `GameplayAbilitySpec`。

示例项目在 `Character` 类上存储了一个 `TArray<TSubclassOf<UGDGameplayAbility>>`，它在游戏开始时读取并授予：
```c++
void AGDCharacterBase::AddCharacterAbilities()
{
	// Grant abilities, but only on the server
	// 授予 abilities，但仅在服务器上
	if (Role != ROLE_Authority || !AbilitySystemComponent.IsValid() || AbilitySystemComponent->bCharacterAbilitiesGiven)
	{
		return;
	}

	for (TSubclassOf<UGDGameplayAbility>& StartupAbility : CharacterAbilities)
	{
		AbilitySystemComponent->GiveAbility(
			FGameplayAbilitySpec(StartupAbility, GetAbilityLevel(StartupAbility.GetDefaultObject()->AbilityID), static_cast<int32>(StartupAbility.GetDefaultObject()->AbilityInputID), this));
	}

	AbilitySystemComponent->bCharacterAbilitiesGiven = true;
}
```

在授予这些 `GameplayAbilities` 时，我们正在使用 `UGameplayAbility` 类、ability 等级、它绑定的输入以及 `SourceObject`（即谁将此 `GameplayAbility` 授予此 `ASC`）创建 `GameplayAbilitySpecs`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-activating"></a>
#### 4.6.4 Activating Abilities
如果为 `GameplayAbility` 分配了输入动作，则在按下输入时且满足其 `GameplayTag` 要求时，它将自动激活。这可能并不总是激活 `GameplayAbility` 的理想方式。`ASC` 提供了四种其他激活 `GameplayAbilities` 的方法：通过 `GameplayTag`、`GameplayAbility` 类、`GameplayAbilitySpec` 句柄以及通过事件。通过事件激活 `GameplayAbility` 允许你[在事件中传递数据 payload](#concepts-ga-data)。

```c++
UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilitiesByTag(const FGameplayTagContainer& GameplayTagContainer, bool bAllowRemoteActivation = true);

UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilityByClass(TSubclassOf<UGameplayAbility> InAbilityToActivate, bool bAllowRemoteActivation = true);

bool TryActivateAbility(FGameplayAbilitySpecHandle AbilityToActivate, bool bAllowRemoteActivation = true);

bool TriggerAbilityFromGameplayEvent(FGameplayAbilitySpecHandle AbilityToTrigger, FGameplayAbilityActorInfo* ActorInfo, FGameplayTag Tag, const FGameplayEventData* Payload, UAbilitySystemComponent& Component);

FGameplayAbilitySpecHandle GiveAbilityAndActivateOnce(const FGameplayAbilitySpec& AbilitySpec, const FGameplayEventData* GameplayEventData);
```
要通过事件激活 `GameplayAbility`，`GameplayAbility` 必须在 `GameplayAbility` 中设置其 `Triggers`。分配一个 `GameplayTag` 并为 `GameplayEvent` 选择一个选项。要发送事件，请使用函数 `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)`。通过事件激活 `GameplayAbility` 允许你传入包含数据的 payload。

`GameplayAbility` `Triggers` 还允许你在添加或删除 `GameplayTag` 时激活 `GameplayAbility`。

**注意：** 在 Blueprint 中从事件激活 `GameplayAbility` 时，你必须使用 `ActivateAbilityFromEvent` 节点。

**注意：** 不要忘记在 `GameplayAbility` 应该终止时调用 `EndAbility()`，除非你有一个将始终运行的 `GameplayAbility`，例如被动 ability。

**本地预测** `GameplayAbilities` 的激活序列：
1. **拥有客户端**调用 `TryActivateAbility()`
1. 调用 `InternalTryActivateAbility()`
1. 调用 `CanActivateAbility()` 并返回是否满足 `GameplayTag` 要求、`ASC` 是否能支付消耗、`GameplayAbility` 是否不在冷却中以及是否没有其他实例当前处于活动状态
1. 调用 `CallServerTryActivateAbility()` 并将其生成的 `Prediction Key` 传递给它
1. 调用 `CallActivateAbility()`
1. 调用 `PreActivate()` Epic 将此称为"样板初始化内容"
1. 调用 `ActivateAbility()` 最终激活 ability

**服务器**接收 `CallServerTryActivateAbility()`
1. 调用 `ServerTryActivateAbility()`
1. 调用 `InternalServerTryActivateAbility()`
1. 调用 `InternalTryActivateAbility()`
1. 调用 `CanActivateAbility()` 并返回是否满足 `GameplayTag` 要求、`ASC` 是否能支付消耗、`GameplayAbility` 是否不在冷却中以及是否没有其他实例当前处于活动状态
1. 如果成功，调用 `ClientActivateAbilitySucceed()`，告知它更新其 `ActivationInfo`，即其激活已由服务器确认，并广播 `OnConfirmDelegate` 委托。这与输入确认不同。
1. 调用 `CallActivateAbility()`
1. 调用 `PreActivate()` Epic 将此称为"样板初始化内容"
1. 调用 `ActivateAbility()` 最终激活 ability

如果服务器在任何时候激活失败，它将调用 `ClientActivateAbilityFailed()`，立即终止客户端的 `GameplayAbility` 并撤消任何预测的更改。

<a name="concepts-ga-activating-passive"></a>
##### 4.6.4.1 Passive Abilities
要实现自动激活并持续运行的被动 `GameplayAbilities`，请覆盖 `UGameplayAbility::OnAvatarSet()`，该函数在授予 `GameplayAbility` 并设置 `AvatarActor` 时自动调用，然后调用 `TryActivateAbility()`。

我建议向你的自定义 `UGameplayAbility` 类添加一个 `bool`，指定是否应在授予时激活 `GameplayAbility`。示例项目为其被动护甲叠加 ability 执行此操作。

被动 `GameplayAbilities` 通常具有 `Server Only` 的 [`Net Execution Policy`](#concepts-ga-net)。

```c++
void UGDGameplayAbility::OnAvatarSet(const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilitySpec & Spec)
{
	Super::OnAvatarSet(ActorInfo, Spec);

	if (bActivateAbilityOnGranted)
	{
		ActorInfo->AbilitySystemComponent->TryActivateAbility(Spec.Handle, false);
	}
}
```

Epic 将此函数描述为启动被动 abilities 和执行 `BeginPlay` 类事项的正确位置。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-activating-failedtags"></a>
##### 4.6.4.2 Activation Failed Tags

Abilities 具有默认逻辑来告诉你 ability 激活失败的原因。要启用此功能，你必须设置对应于默认失败情况的 GameplayTags。

将这些 tags（或你自己的命名约定）添加到你的项目：
```
+GameplayTagList=(Tag="Activation.Fail.BlockedByTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.CantAffordCost",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.IsDead",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.MissingTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.Networking",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.OnCooldown",DevComment="")
```

然后将它们添加到 [`GASDocumentation\Config\DefaultGame.ini`](https://github.com/tranek/GASDocumentation/blob/master/Config/DefaultGame.ini#L8-L13)：
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
ActivateFailIsDeadName=Activation.Fail.IsDead
ActivateFailCooldownName=Activation.Fail.OnCooldown
ActivateFailCostName=Activation.Fail.CantAffordCost
ActivateFailTagsBlockedName=Activation.Fail.BlockedByTags
ActivateFailTagsMissingName=Activation.Fail.MissingTags
ActivateFailNetworkingName=Activation.Fail.Networking
```

现在，每当 ability 激活失败时，此对应的 GameplayTag 将包含在输出日志消息中或在 `showdebug AbilitySystem` hud 上可见。
```
LogAbilitySystem: Display: InternalServerTryActivateAbility. Rejecting ClientActivation of Default__GA_FireGun_C. InternalTryActivateAbility failed: Activation.Fail.BlockedByTags
LogAbilitySystem: Display: ClientActivateAbilityFailed_Implementation. PredictionKey :109 Ability: Default__GA_FireGun_C
```

![Activation Failed Tags Displayed in showdebug AbilitySystem](https://github.com/tranek/GASDocumentation/raw/master/Images/activationfailedtags.png)

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-cancelabilities"></a>
#### 4.6.5 Canceling Abilities
要从内部取消 `GameplayAbility`，你可以调用 `CancelAbility()`。这将调用 `EndAbility()` 并将其 `WasCancelled` 参数设置为 true。

要从外部取消 `GameplayAbility`，`ASC` 提供了几个函数：

```c++
/** Cancels the specified ability CDO. */
/** 取消指定的 ability CDO。 */
void CancelAbility(UGameplayAbility* Ability);

/** Cancels the ability indicated by passed in spec handle. If handle is not found among reactivated abilities nothing happens. */
/** 取消由传入的 spec 句柄指示的 ability。如果在重新激活的 abilities 中找不到句柄，则不会发生任何事情。 */
void CancelAbilityHandle(const FGameplayAbilitySpecHandle& AbilityHandle);

/** Cancel all abilities with the specified tags. Will not cancel the Ignore instance */
/** 取消具有指定 tags 的所有 abilities。不会取消 Ignore 实例 */
void CancelAbilities(const FGameplayTagContainer* WithTags=nullptr, const FGameplayTagContainer* WithoutTags=nullptr, UGameplayAbility* Ignore=nullptr);

/** Cancels all abilities regardless of tags. Will not cancel the ignore instance */
/** 取消所有 abilities，无论 tags 如何。不会取消 ignore 实例 */
void CancelAllAbilities(UGameplayAbility* Ignore=nullptr);

/** Cancels all abilities and kills any remaining instanced abilities */
/** 取消所有 abilities 并终止任何剩余的实例化 abilities */
virtual void DestroyActiveState();
```

**注意：** 我发现如果你有 `Non-Instanced` `GameplayAbilities`，`CancelAllAbilities` 似乎无法正常工作。它似乎会遇到 `Non-Instanced` `GameplayAbility` 然后放弃。`CancelAbilities` 可以更好地处理 `Non-Instanced` `GameplayAbilities`，这也是示例项目所使用的（Jump 是一个非实例化的 `GameplayAbility`）。你的结果可能会有所不同。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-definition-activeability"></a>
#### 4.6.6 Getting Active Abilities
初学者经常问"我如何获取当前的 ability？"，可能是为了在其上设置变量或取消它。同一时间可以有多个 `GameplayAbility` 处于活动状态，因此没有一个单一的"当前 ability"。相反，你必须搜索 `ASC` 的 `ActivatableAbilities` 列表（`ASC` 拥有的已授予 `GameplayAbilities`），并找到与你正在寻找的 [`Asset` 或 `Granted` `GameplayTag`](#concepts-ga-tags) 匹配的那个。

`UAbilitySystemComponent::GetActivatableAbilities()` 返回一个 `TArray<FGameplayAbilitySpec>` 供你迭代。

`ASC` 还有另一个辅助函数，它接受 `GameplayTagContainer` 作为参数来帮助搜索，而不是手动迭代 `GameplayAbilitySpecs` 列表。`bOnlyAbilitiesThatSatisfyTagRequirements` 参数将仅返回满足其 `GameplayTag` 要求且现在可以激活的 `GameplayAbilitySpecs`。例如，你可以有两个基本攻击 `GameplayAbilities`，一个使用武器，另一个使用空手，正确的那个根据是否装备武器来激活设置 `GameplayTag` 要求。有关更多信息，请参阅 Epic 对该函数的注释。
```c++
UAbilitySystemComponent::GetActivatableGameplayAbilitySpecsByAllMatchingTags(const FGameplayTagContainer& GameplayTagContainer, TArray < struct FGameplayAbilitySpec* >& MatchingGameplayAbilities, bool bOnlyAbilitiesThatSatisfyTagRequirements = true)
```

一旦你获得你正在寻找的 `FGameplayAbilitySpec`，你就可以对其调用 `IsActive()`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-instancing"></a>
#### 4.6.7 Instancing Policy
`GameplayAbility` 的 `Instancing Policy` 决定了在激活时是否以及如何实例化 `GameplayAbility`。

| `Instancing Policy`     | Description                                                                                      | Example of when to use                                                                                                                                                                                                                                                                                                                                                                                             |
| ----------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Instanced Per Actor     | 每个 `ASC` 只有一个 `GameplayAbility` 实例，该实例在激活之间重用。                                 | 这可能是你最常使用的 `Instancing Policy`。你可以将其用于任何 ability，并在激活之间提供持久性。设计者负责在需要时手动重置激活之间的任何变量。                                                                                                                                                                                                                               |
| Instanced Per Execution | 每次激活 `GameplayAbility` 时，都会创建一个新的 `GameplayAbility` 实例。                         | 这些 `GameplayAbilities` 的好处是变量每次激活时都会重置。它们的性能比 `Instanced Per Actor` 差，因为它们每次激活都会生成新的 `GameplayAbilities`。示例项目不使用任何此类 ability。                                                                                                                                                                                                 |
| Non-Instanced           | `GameplayAbility` 在其 `ClassDefaultObject` 上运行。不会创建实例。                                | 这是三者中性能最好的，但对于可以用它做什么最有限制。`Non-Instanced` `GameplayAbilities` 无法存储状态，意味着没有动态变量，也没有绑定到 `AbilityTask` 委托。使用它们的最佳位置是用于频繁使用的简单 abilities，例如 MOBA 或 RTS 中的小兵基本攻击。示例项目的 Jump `GameplayAbility` 是 `Non-Instanced`。 |

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-net"></a>
#### 4.6.8 Net Execution Policy
`GameplayAbility` 的 `Net Execution Policy` 决定了谁运行 `GameplayAbility` 以及以什么顺序运行。

| `Net Execution Policy` | Description                                                                                                                                                                                                         |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Local Only`           | `GameplayAbility` 仅在拥有客户端上运行。这对于仅进行本地装饰性更改的 abilities 可能很有用。单人游戏应使用 `Server Only`。                                     |
| `Local Predicted`      | `Local Predicted` `GameplayAbilities` 首先在拥有客户端上激活，然后在服务器上激活。服务器版本将更正客户端预测不正确的任何内容。请参阅 [Prediction](#concepts-p)。 |
| `Server Only`          | `GameplayAbility` 仅在服务器上运行。被动 `GameplayAbilities` 通常将是 `Server Only`。单人游戏应使用此选项。                                                                  |
| `Server Initiated`     | `Server Initiated` `GameplayAbilities` 首先在服务器上激活，然后在拥有客户端上激活。我个人并没有使用过太多这些（如果有的话）。                                                                     | 

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-tags"></a>
#### 4.6.9 Ability Tags
`GameplayAbilities` 带有具有内置逻辑的 `GameplayTagContainers`。这些 `GameplayTags` 都不会被复制。

| `GameplayTag Container`     | Description                                                                                                                                                                                   |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Ability Tags`              | `GameplayAbility` 拥有的 `GameplayTags`。这些只是用于描述 `GameplayAbility` 的 `GameplayTags`。                                                                              |
| `Cancel Abilities with Tag` | 在 `Ability Tags` 中具有这些 `GameplayTags` 的其他 `GameplayAbilities` 将在激活此 `GameplayAbility` 时被取消。                                                   |
| `Block Abilities with Tag`  | 在 `Ability Tags` 中具有这些 `GameplayTags` 的其他 `GameplayAbilities` 在此 `GameplayAbility` 处于活动状态时将被阻止激活。                                          |
| `Activation Owned Tags`     | 当此 `GameplayAbility` 处于活动状态时，这些 `GameplayTags` 将被赋予 `GameplayAbility` 的拥有者。请记住这些不会被复制。                                                    |
| `Activation Required Tags`  | 只有在拥有者拥有**所有**这些 `GameplayTags` 时，才能激活此 `GameplayAbility`。                                                                                                |
| `Activation Blocked Tags`   | 如果拥有者拥有**任何**这些 `GameplayTags`，则无法激活此 `GameplayAbility`。                                                                                                  |
| `Source Required Tags`      | 只有在 `Source` 拥有**所有**这些 `GameplayTags` 时，才能激活此 `GameplayAbility`。`Source` `GameplayTags` 仅在 `GameplayAbility` 由事件触发时设置。 |
| `Source Blocked Tags`       | 如果 `Source` 拥有**任何**这些 `GameplayTags`，则无法激活此 `GameplayAbility`。`Source` `GameplayTags` 仅在 `GameplayAbility` 由事件触发时设置。   |
| `Target Required Tags`      | 只有在 `Target` 拥有**所有**这些 `GameplayTags` 时，才能激活此 `GameplayAbility`。`Target` `GameplayTags` 仅在 `GameplayAbility` 由事件触发时设置。 |
| `Target Blocked Tags`       | 如果 `Target` 拥有**任何**这些 `GameplayTags`，则无法激活此 `GameplayAbility`。`Target` `GameplayTags` 仅在 `GameplayAbility` 由事件触发时设置。   |

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-spec"></a>
#### 4.6.10 Gameplay Ability Spec
在授予 `GameplayAbility` 后，`GameplayAbilitySpec` 存在于 `ASC` 上，并定义了可激活的 `GameplayAbility` - `GameplayAbility` 类、等级、输入绑定和必须与 `GameplayAbility` 类分开的运行时状态。

当在服务器上授予 `GameplayAbility` 时，服务器会将 `GameplayAbilitySpec` 复制到拥有客户端，以便她可以激活它。

激活 `GameplayAbilitySpec` 将根据其实例化策略创建 `GameplayAbility` 的实例（对于 `Non-Instanced` `GameplayAbilities` 则不创建）。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-data"></a>
#### 4.6.11 Passing Data to Abilities
`GameplayAbilities` 的一般范式是 `Activate->Generate Data->Apply->End`。有时你需要对现有数据进行操作。GAS 提供了几种将外部数据获取到你的 `GameplayAbilities` 中的选项：

| Method                                          | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Activate `GameplayAbility` by Event             | 使用包含数据 payload 的事件激活 `GameplayAbility`。事件的 payload 从客户端复制到服务器，用于本地预测的 `GameplayAbilities`。使用两个 `Optional Object` 或 [`TargetData`](#concepts-targeting-data) 变量来存储不适合任何现有变量的任意数据。这样做的缺点是它阻止你使用输入绑定激活 ability。要通过事件激活 `GameplayAbility`，`GameplayAbility` 必须在 `GameplayAbility` 中设置其 `Triggers`。分配一个 `GameplayTag` 并为 `GameplayEvent` 选择一个选项。要发送事件，请使用函数 `UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)`。 |
| Use `WaitGameplayEvent` `AbilityTask`           | 使用 `WaitGameplayEvent` `AbilityTask` 告诉 `GameplayAbility` 在激活后监听带有 payload 数据的事件。事件 payload 和发送它的过程与通过事件激活 `GameplayAbilities` 相同。这样做的缺点是事件不会由 `AbilityTask` 复制，应仅用于 `Local Only` 和 `Server Only` `GameplayAbilities`。你可能会编写自己的 `AbilityTask` 来复制事件 payload。                                                                                                                                                                                                                                                                                               |
| Use `TargetData`                                | 自定义 `TargetData` 结构是在客户端和服务器之间传递任意数据的好方法。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Store Data on the `OwnerActor` or `AvatarActor` | 使用存储在 `OwnerActor`、`AvatarActor` 或你可以获得引用的任何其他对象上的复制变量。此方法最灵活，并且将适用于由输入绑定激活的 `GameplayAbilities`。但是，它不能保证数据在使用时从复制同步。你必须提前确保这一点 - 意味着如果你设置了一个复制变量然后立即激活 `GameplayAbility`，由于可能的数据包丢失，无法保证接收器上发生的顺序。                                                                                                                                                                                                                                   |

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-commit"></a>
#### 4.6.12 Ability Cost and Cooldown
`GameplayAbilities` 带有可选消耗和冷却的功能。消耗是 `ASC` 为了激活 `GameplayAbility` 必须拥有的 `Attributes` 的预定义量，通过 `Instant` `GameplayEffect` ([`Cost GE`](#concepts-ge-cost)) 实现。冷却是防止 `GameplayAbility` 在到期之前重新激活的计时器，通过 `Duration` `GameplayEffect` ([`Cooldown GE`](#concepts-ge-cooldown)) 实现。

在 `GameplayAbility` 调用 `UGameplayAbility::Activate()` 之前，它会调用 `UGameplayAbility::CanActivateAbility()`。此函数检查拥有 `ASC` 是否能支付消耗（`UGameplayAbility::CheckCost()`）并确保 `GameplayAbility` 不在冷却中（`UGameplayAbility::CheckCooldown()`）。

在 `GameplayAbility` 调用 `Activate()` 后，它可以使用 `UGameplayAbility::CommitAbility()` 在任何时候选择性地提交消耗和冷却，该函数调用 `UGameplayAbility::CommitCost()` 和 `UGameplayAbility::CommitCooldown()`。设计者可以选择单独调用 `CommitCost()` 或 `CommitCooldown()`，如果它们不应该同时提交。提交消耗和冷却会再调用一次 `CheckCost()` 和 `CheckCooldown()`，这是 `GameplayAbility` 与它们相关的最后一次失败机会。拥有 `ASC` 的 `Attributes` 可能会在 `GameplayAbility` 激活后发生变化，在提交时无法满足消耗。如果[预测键](#concepts-p-key)在提交时有效，则提交消耗和冷却可以[本地预测](#concepts-p)。

有关实现详细信息，请参阅 [`CostGE`](#concepts-ge-cost) 和 [`CooldownGE`](#concepts-ge-cooldown)。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-leveling"></a>
#### 4.6.13 Leveling Up Abilities
有两种常见的升级 ability 方法：

| Level Up Method                            | Description                                                                                                                                                                                                      |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ungrant and Regrant at the New Level       | 从 `ASC` 中取消授予（移除）`GameplayAbility` 并在服务器上以下一级重新授予它。如果 `GameplayAbility` 当时处于活动状态，这将终止它。                                   |
| Increase the `GameplayAbilitySpec's` Level | 在服务器上，找到 `GameplayAbilitySpec`，增加其等级，并将其标记为 dirty 以复制到拥有客户端。如果 `GameplayAbility` 当时处于活动状态，此方法不会终止它。 |

这两种方法的主要区别在于你是否希望当前活动的 `GameplayAbilities` 在升级时被取消。你很可能会根据你的 `GameplayAbilities` 使用这两种方法。我建议向你的 `UGameplayAbility` 子类添加一个 `bool`，指定要使用哪种方法。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-sets"></a>
#### 4.6.14 Ability Sets
`GameplayAbilitySets` 是便利的 `UDataAsset` 类，用于保存输入绑定和角色的启动 `GameplayAbilities` 列表，以及授予 `GameplayAbilities` 的逻辑。子类还可以包括额外的逻辑或属性。Paragon 每个英雄都有一个 `GameplayAbilitySet`，其中包括所有给定的 `GameplayAbilities`。

我发现这个类至少从目前我所看到的情况来看是不必要的。示例项目在 `GDCharacterBase` 及其子类内部处理 `GameplayAbilitySets` 的所有功能。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-batching"></a>
#### 4.6.15 Ability Batching
传统的 `Gameplay Ability` 生命周期涉及从客户端到服务器的最少两到三个 RPC。

1. `CallServerTryActivateAbility()`
1. `ServerSetReplicatedTargetData()`（可选）
1. `ServerEndAbility()`

如果 `GameplayAbility` 在一帧中作为一个原子分组执行所有这些操作，我们可以优化此工作流以将所有两到三个 RPC 批处理（组合）成一个 RPC。`GAS` 将此 RPC 优化称为 `Ability Batching`。何时使用 `Ability Batching` 的常见示例是 hitscan 枪支。Hitscan 枪支会激活、进行线跟踪、将 [`TargetData`](#concepts-targeting-data) 发送到服务器，并在一帧中作为一个原子组结束 ability。[GASShooter](https://github.com/tranek/GASShooter) 示例项目为其 hitscan 枪支演示了此技术。

半自动枪是最佳情况，将 `CallServerTryActivateAbility()`、`ServerSetReplicatedTargetData()`（子弹命中结果）和 `ServerEndAbility()` 批处理为一个 RPC，而不是三个 RPC。

全自动/ Burst 枪将第一颗子弹的 `CallServerTryActivateAbility()` 和 `ServerSetReplicatedTargetData()` 批处理为一个 RPC，而不是两个 RPC。每个后续子弹都是其自己的 `ServerSetReplicatedTargetData()` RPC。最后，当枪停止射击时，`ServerEndAbility()` 作为单独的 RPC 发送。这是最坏的情况，我们只在第一颗子弹上节省了一个 RPC，而不是两个。这种情况也可以通过 [`Gameplay Event`](#concepts-ga-data) 激活 ability 来实现，这将把子弹的 `TargetData` 随 `EventPayload` 一起从客户端发送到服务器。后一种方法的缺点是 `TargetData` 必须在 ability 外部生成，而批处理方法在 ability 内部生成 `TargetData`。

`Ability Batching` 在 [`ASC`](#concepts-asc) 上默认处于禁用状态。要启用 `Ability Batching`，请覆盖 `ShouldDoServerAbilityRPCBatch()` 以返回 true：

```c++
virtual bool ShouldDoServerAbilityRPCBatch() const override { return true; }
```

现在已启用 `Ability Batching`，在激活要批处理的 abilities 之前，你必须事先创建一个 `FScopedServerAbilityRPCBatcher` 结构。这个特殊的结构将尝试批处理其范围内的所有 abilities。一旦 `FScopedServerAbilityRPCBatcher` 超出范围，任何激活的 abilities 都不会尝试批处理。`FScopedServerAbilityRPCBatcher` 通过在每个可以批处理的函数中拥有特殊代码来工作，这些代码拦截调用而不发送 RPC，而是将消息打包到批处理结构中。当 `FScopedServerAbilityRPCBatcher` 超出范围时，它会自动在 `UAbilitySystemComponent::EndServerAbilityRPCBatch()` 中将此批处理结构 RPC 到服务器。服务器在 `UAbilitySystemComponent::ServerAbilityRPCBatch_Internal(FServerAbilityRPCBatch& BatchInfo)` 中接收批处理 RPC。`BatchInfo` 参数将包含标志，指示 ability 是否应结束、在激活时是否按下了输入，以及 `TargetData`（如果包含的话）。这是一个适合放置断点以确认批处理是否正常工作的函数。或者，使用 cvar `AbilitySystem.ServerRPCBatching.Log 1` 来启用特殊的 ability 批处理日志。

此机制只能在 C++ 中完成，并且只能通过其 `FGameplayAbilitySpecHandle` 激活 abilities。

```c++
bool UGSAbilitySystemComponent::BatchRPCTryActivateAbility(FGameplayAbilitySpecHandle InAbilityHandle, bool EndAbilityImmediately)
{
	bool AbilityActivated = false;
	if (InAbilityHandle.IsValid())
	{
		FScopedServerAbilityRPCBatcher GSAbilityRPCBatcher(this, InAbilityHandle);
		AbilityActivated = TryActivateAbility(InAbilityHandle, true);

		if (EndAbilityImmediately)
		{
			FGameplayAbilitySpec* AbilitySpec = FindAbilitySpecFromHandle(InAbilityHandle);
			if (AbilitySpec)
			{
				UGSGameplayAbility* GSAbility = Cast<UGSGameplayAbility>(AbilitySpec->GetPrimaryInstance());
				GSAbility->ExternalEndAbility();
			}
		}

		return AbilityActivated;
	}

	return AbilityActivated;
}
```

GASShooter 为半自动和全自动枪支重用相同的批处理 `GameplayAbility`，它从不直接调用 `EndAbility()`（它由仅限本地的 ability 在 ability 外部处理，该 ability 管理玩家输入并根据当前开火模式调用批处理 ability）。由于所有 RPC 都必须在 `FScopedServerAbilityRPCBatcher` 的范围内发生，我提供了 `EndAbilityImmediately` 参数，以便控制/管理的仅限本地可以指定此 ability 是否应该批处理 `EndAbility()` 调用（半自动），或者不批处理 `EndAbility()` 调用（全自动），并且 `EndAbility()` 调用将在稍后在其自己的 RPC 中发生。

GASShooter 公开了一个 Blueprint 节点以允许批处理 abilities，上述仅限本地的 ability 使用该节点来触发批处理 ability。

![Activate Batched Ability](https://github.com/tranek/GASDocumentation/raw/master/Images/batchabilityactivate.png)

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-ga-netsecuritypolicy"></a>
#### 4.6.16 Net Security Policy
`GameplayAbility` 的 `NetSecurityPolicy` 确定了 ability 应在网络上的何处执行。它提供保护，防止客户端尝试执行受限的 abilities。

| `NetSecurityPolicy`     | 描述                                                                                                                                                |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ClientOrServer`        | 无安全要求。客户端或服务器可以自由触发此 ability 的执行和终止。                                           |
| `ServerOnlyExecution`   | 服务器将忽略客户端请求执行此 ability。客户端仍然可以请求服务器取消或结束此 ability。 |
| `ServerOnlyTermination` | 服务器将忽略客户端请求取消或结束此 ability。客户端仍然可以请求执行此 ability。      |
| `ServerOnly`            | 服务器控制此 ability 的执行和终止。客户端发出的任何请求都将被忽略。                                      |

**[⬆ Back to Top](#table-of-contents)**


<a name="concepts-at"></a>
### 4.7 Ability Tasks

<a name="concepts-at-definition"></a>
### 4.7.1 Ability Task 定义
`GameplayAbilities` 仅在一帧内执行。这本身并不能提供太大的灵活性。要执行随时间发生的操作或需要响应稍后触发的委托，我们使用称为 `AbilityTasks` 的延迟操作。

GAS 开箱即用地提供了许多 `AbilityTasks`：
* 使用 `RootMotionSource` 移动 `Character` 的任务
* 播放动画蒙太奇的任务
* 响应 `Attribute` 更改的任务
* 响应 `GameplayEffect` 更改的任务
* 响应玩家输入的任务
* 以及更多

`UAbilityTask` 构造函数强制执行游戏范围内 1000 个并发 `AbilityTasks` 同时运行的最大硬限制。在设计可以在同一时刻拥有数百个角色的游戏（如 RTS 游戏）的 `GameplayAbilities` 时，请记住这一点。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-at-definition"></a>
### 4.7.2 自定义 Ability Tasks
您通常会创建自己的自定义 `AbilityTasks`（在 C++ 中）。示例项目包含两个自定义 `AbilityTasks`：
1. `PlayMontageAndWaitForEvent` 是默认的 `PlayMontageAndWait` 和 `WaitGameplayEvent` `AbilityTasks` 的组合。这允许动画蒙太奇从 `AnimNotifies` 向启动它们的 `GameplayAbility` 发送游戏事件。使用此功能可以在动画蒙太奇的特定时间触发操作。
1. `WaitReceiveDamage` 监听 `OwnerActor` 接收伤害。被动护甲堆叠 `GameplayAbility` 会在英雄受到伤害实例时移除一层护甲。

`AbilityTasks` 由以下部分组成：
* 创建 `AbilityTask` 新实例的静态函数
* 当 `AbilityTask` 完成其目的时广播的委托
* 用于启动其主要工作、绑定到外部委托等的 `Activate()` 函数
* 用于清理的 `OnDestroy()` 函数，包括它绑定的外部委托
* 其绑定的任何外部委托的回调函数
* 成员变量和任何内部辅助函数

**注意**：`AbilityTasks` 只能声明一种类型的输出委托。所有输出委托必须属于此类型，无论它们是否使用这些参数。为未使用的委托参数传递默认值。

`AbilityTasks` 仅在运行拥有 `GameplayAbility` 的客户端或服务器上运行；但是，可以通过在 `AbilityTask` 构造函数中设置 `bSimulatedTask = true;`、重写 `virtual void InitSimulatedTask(UGameplayTasksComponent& InGameplayTasksComponent);` 并设置任何成员变量为复制来使 `AbilityTasks` 在模拟客户端上运行。这仅适用于罕见情况，例如移动 `AbilityTasks`，您不希望复制每次移动更改而是模拟整个移动 `AbilityTask`。所有 `RootMotionSource` `AbilityTasks` 都这样做。请参阅 `AbilityTask_MoveToLocation.h/.cpp` 作为示例。

如果需要在帧之间平滑地插值数值，`AbilityTasks` 可以 `Tick`，如果在 `AbilityTask` 构造函数中设置 `bTickingTask = true;` 并重写 `virtual void TickTask(float DeltaTime);`。请参阅 `AbilityTask_MoveToLocation.h/.cpp` 作为示例。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-at-using"></a>
### 4.7.3 使用 Ability Tasks
要在 C++ 中创建和激活 `AbilityTask`（来自 `GDGA_FireGun.cpp`）：
```c++
UGDAT_PlayMontageAndWaitForEvent* Task = UGDAT_PlayMontageAndWaitForEvent::PlayMontageAndWaitForEvent(this, NAME_None, MontageToPlay, FGameplayTagContainer(), 1.0f, NAME_None, false, 1.0f);
Task->OnBlendOut.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnCompleted.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnInterrupted.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->OnCancelled.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->EventReceived.AddDynamic(this, &UGDGA_FireGun::EventReceived);
Task->ReadyForActivation();
```

在蓝图中，我们只需使用为 `AbilityTask` 创建的蓝图节点。我们不必调用 `ReadyForActivation()`。这由 `Engine/Source/Editor/GameplayTasksEditor/Private/K2Node_LatentGameplayTaskCall.cpp` 自动调用。如果 `K2Node_LatentGameplayTaskCall` 在您的 `AbilityTask` 类中存在，它也会自动调用 `BeginSpawningActor()` 和 `FinishSpawningActor()`（请参阅 `AbilityTask_WaitTargetData`）。重申一下，`K2Node_LatentGameplayTaskCall` 仅对蓝图进行自动魔法处理。在 C++ 中，我们必须手动调用 `ReadyForActivation()`、`BeginSpawningActor()` 和 `FinishSpawningActor()`。

![蓝图 WaitTargetData AbilityTask](https://github.com/tranek/GASDocumentation/raw/master/Images/abilitytask.png)

要手动取消 `AbilityTask`，只需在蓝图（称为 `Async Task Proxy`）或 C++ 中的 `AbilityTask` 对象上调用 `EndTask()`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-at-rms"></a>
### 4.7.4 Root Motion Source Ability Tasks
GAS 附带了使用连接到 `CharacterMovementComponent` 的 `Root Motion Sources` 随时间移动 `Character` 的 `AbilityTasks`，用于击退、复杂跳跃、拉动和冲刺等操作。

**注意**：预测 `RootMotionSource` `AbilityTasks` 在引擎版本 4.19 和 4.25+ 中有效。预测在引擎版本 4.20-4.24 中存在 bug；但是，`AbilityTasks` 仍然在多人游戏中执行其功能，只有轻微的网络修正，并且在单人游戏中完美运行。可以从 4.25 中挑选 [预测修复](https://github.com/EpicGames/UnrealEngine/commit/94107438dd9f490e7b743f8e13da46927051bf33#diff-65f6196f9f28f560f95bd578e07e290c) 到自定义 4.20-4.24 引擎。

**[⬆ 返回顶部](#table-of-contents)**


<a name="concepts-gc"></a>
### 4.8 Gameplay Cues

<a name="concepts-gc-definition"></a>
#### 4.8.1 Gameplay Cue 定义
`GameplayCues`（`GC`）执行与游戏无关的内容，例如声音效果、粒子效果、相机震动等。`GameplayCues` 通常会被复制（除非明确在本地 `Execute`、`Add` 或 `Remove`）和预测。

我们通过向 `GameplayCueManager` 发送相应的 `GameplayTag` 以及事件类型（`Execute`、`Add` 或 `Remove`）来触发 `GameplayCues`，该标签必须以 `GameplayCue.` 作为父名称。`GameplayCueNotify` 对象和其他实现 `IGameplayCueInterface` 的 `Actor` 可以根据 `GameplayCue's` 的 `GameplayTag`（`GameplayCueTag`）订阅这些事件。

**注意**：再次强调，`GameplayCue` 的 `GameplayTags` 需要以父 `GameplayTag` `GameplayCue` 开头。因此，一个有效的 `GameplayCue` 的 `GameplayTag` 可能是 `GameplayCue.A.B.C`。

`GameplayCueNotifies` 有两类：`Static` 和 `Actor`。它们响应不同的事件，不同类型的 `GameplayEffects` 可以触发它们。使用您的逻辑重写相应的事件。

| `GameplayCue` 类                                                                                                                   | 事件             | `GameplayEffect` 类型   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| --------------------------------------------------------------------------------------------------------------------------------- | ---------------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`GameplayCueNotify_Static`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UGameplayCueNotify_Static/index.html) | `Execute`        | `Instant` 或 `Periodic` | 静态 `GameplayCueNotifies` 在 `ClassDefaultObject` 上运行（意味着没有实例），非常适合一次性效果，如命中冲击。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| [`GameplayCueNotify_Actor`](https://docs.unrealengine.com/en-US/BlueprintAPI/GameplayCueNotify/index.html)                          | `Add` 或 `Remove` | `Duration` 或 `Infinite` | Actor `GameplayCueNotifies` 在 `Add` 时生成一个新实例。由于这些是实例化的，它们可以随时间执行操作，直到被 `Remove`。这些适用于循环声音和粒子效果，当支持它们的 `Duration` 或 `Infinite` `GameplayEffect` 被移除或通过手动调用移除时，这些效果将被移除。这些还附带用于管理同时允许 `Add` 多少个的选项，以便同一效果的多次应用只启动一次声音或粒子。                                                                                                                                                                                                                                                                                                                                                                |

`GameplayCueNotifies` 技术上可以响应任何事件，但这通常是我们使用它们的方式。

**注意**：使用 `GameplayCueNotify_Actor` 时，检查 `Auto Destroy on Remove`，否则对该 `GameplayCueTag` 的后续 `Add` 调用将不起作用。

当使用非 `Full` 的 `ASC` [复制模式](#concepts-asc-rm)时，`Add` 和 `Remove` 的 `GC` 事件将在服务器玩家（监听服务器）上触发两次 - 一次用于应用 `GE`，另一次从"最小"的 `NetMultiCast` 到客户端。但是，`WhileActive` 事件仍将只触发一次。所有事件在客户端上只触发一次。

示例项目包含一个用于眩晕和冲刺效果的 `GameplayCueNotify_Actor`。它还有一个用于 FireGun 的投弹物撞击的 `GameplayCueNotify_Static`。这些 `GC` 可以通过[在本地触发它们](#concepts-gc-local)而不是通过 `GE` 复制它们来进一步优化。在示例项目中，我选择了展示使用它们的初学者方式。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-gc-trigger"></a>
#### 4.8.2 触发 Gameplay Cues

从 `GameplayEffect` 内部成功应用时（未被标签或免疫阻止），填写所有应触发的 `GameplayCues` 的 `GameplayTags`。

![从 GameplayEffect 触发的 GameplayCue](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromge.png)

`UGameplayAbility` 提供了蓝图节点来 `Execute`、`Add` 或 `Remove` `GameplayCues`。

![从 GameplayAbility 触发的 GameplayCue](https://github.com/tranek/GASDocumentation/raw/master/Images/gcfromga.png)

在 C++ 中，您可以直接在 `ASC` 上调用函数（或在 `ASC` 子类中将它们暴露给蓝图）：

```c++
/** GameplayCues 也可以单独出现。这些接受一个可选的效果上下文来传递命中结果等 */
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** 添加持久的 gameplay cue */
void AddGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void AddGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** 移除持久的 gameplay cue */
void RemoveGameplayCue(const FGameplayTag GameplayCueTag);

/** 移除任何单独添加的 GameplayCue，即不作为 GameplayEffect 的一部分。 */
void RemoveAllGameplayCues();
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-gc-local"></a>
#### 4.8.3 本地 Gameplay Cues
从 `GameplayAbilities` 和 `ASC` 触发 `GameplayCues` 的公开函数默认是复制的。每个 `GameplayCue` 事件都是一个多播 RPC。这可能会导致大量的 RPC。GAS 还强制每个网络更新最多只能有两个相同的 `GameplayCue` RPC。我们通过在可以使用的地方使用本地 `GameplayCues` 来避免这种情况。本地 `GameplayCues` 仅在单个客户端上 `Execute`、`Add` 或 `Remove`。

我们可以使用本地 `GameplayCues` 的场景：
* 投弹物撞击
* 近战碰撞撞击
* 从动画蒙太奇触发的 `GameplayCues`

您应该添加到 `ASC` 子类的本地 `GameplayCue` 函数：

```c++
UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

UFUNCTION(BlueprintCallable, Category = "GameplayCue", Meta = (AutoCreateRefTerm = "GameplayCueParameters", GameplayTagFilter = "GameplayCue"))
void RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);
```

```c++
void UPAAbilitySystemComponent::ExecuteGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Executed, GameplayCueParameters);
}

void UPAAbilitySystemComponent::AddGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::OnActive, GameplayCueParameters);
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::WhileActive, GameplayCueParameters);
}

void UPAAbilitySystemComponent::RemoveGameplayCueLocal(const FGameplayTag GameplayCueTag, const FGameplayCueParameters & GameplayCueParameters)
{
	UAbilitySystemGlobals::Get().GetGameplayCueManager()->HandleGameplayCue(GetOwner(), GameplayCueTag, EGameplayCueEvent::Type::Removed, GameplayCueParameters);
}
```

如果 `GameplayCue` 是本地 `Add` 的，则应该本地 `Remove`。如果是通过复制 `Add` 的，则应该通过复制 `Remove`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-gc-parameters"></a>
#### 4.8.4 Gameplay Cue 参数
`GameplayCues` 接收一个包含 `GameplayCue` 额外信息的 `FGameplayCueParameters` 结构作为参数。如果您从 `GameplayAbility` 或 `ASC` 的函数手动触发 `GameplayCue`，则必须手动填写传递给 `GameplayCue` 的 `GameplayCueParameters` 结构。如果 `GameplayCue` 由 `GameplayEffect` 触发，则以下变量会自动填充到 `GameplayCueParameters` 结构中：

* AggregatedSourceTags
* AggregatedTargetTags
* GameplayEffectLevel
* AbilityLevel
* [EffectContext](#concepts-ge-context)
* Magnitude（如果 `GameplayEffect` 在 `GameplayCue` 标签容器上方的下拉列表中选择了一个用于幅度的 `Attribute`，并且有一个影响该 `Attribute` 的相应 `Modifier`）

`GameplayCueParameters` 结构中的 `SourceObject` 变量可能是一个很好的地方，可以在手动触发 `GameplayCue` 时将任意数据传递给 `GameplayCue`。

**注意**：参数结构中的某些变量（如 `Instigator`）可能已经存在于 `EffectContext` 中。`EffectContext` 还可以包含 `FHitResult`，用于在世界中生成 `GameplayCue` 的位置。子类化 `EffectContext` 可能是一种向 `GameplayCues` 传递更多数据的好方法，尤其是那些由 `GameplayEffect` 触发的。

请参阅 [`UAbilitySystemGlobals`](#concepts-asg) 中的 3 个函数，它们填充 `GameplayCueParameters` 结构以获取更多信息。它们是虚拟的，因此您可以重写它们以自动填充更多信息。

```c++
/** 初始化 GameplayCue 参数 */
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectSpecForRPC &Spec);
virtual void InitGameplayCueParameters_GESpec(FGameplayCueParameters& CueParameters, const FGameplayEffectSpec &Spec);
virtual void InitGameplayCueParameters(FGameplayCueParameters& CueParameters, const FGameplayEffectContextHandle& EffectContext);
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-gc-manager"></a>
#### 4.8.5 Gameplay Cue 管理器
默认情况下，`GameplayCueManager` 将扫描整个游戏目录以查找 `GameplayCueNotifies` 并在播放时将它们加载到内存中。我们可以通过在 `DefaultGame.ini` 中设置来更改 `GameplayCueManager` 扫描的路径。

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
GameplayCueNotifyPaths="/Game/GASDocumentation/Characters"
```

我们确实希望 `GameplayCueManager` 扫描并找到所有 `GameplayCueNotifies`；但是，我们不希望它在播放时异步加载每一个。这将把每个 `GameplayCueNotify` 及其所有引用的声音和粒子放入内存中，无论它们是否在关卡中使用。在像 Paragon 这样的大型游戏中，这可能是数百兆字节的内存中不需要的资源，并会导致启动时的卡顿和游戏冻结。

在启动时异步加载每个 `GameplayCue` 的另一种方法是仅在游戏中触发 `GameplayCues` 时才异步加载它们。这减少了不必要的内存使用和潜在的严重游戏冻结，同时异步加载每个 `GameplayCue`，代价是在游戏过程中首次触发特定 `GameplayCue` 时可能会延迟效果。对于 SSD 来说，这种潜在的延迟是不存在的。我没有在 HDD 上测试过。如果在 UE 编辑器中使用此选项，如果编辑器需要编译粒子系统，则在第一次加载 GameplayCues 期间可能会有轻微的卡顿或冻结。这在构建中不是问题，因为粒子系统已经编译好了。

首先，我们必须子类化 `UGameplayCueManager` 并告诉 `AbilitySystemGlobals` 类在 `DefaultGame.ini` 中使用我们的 `UGameplayCueManager` 子类。

```
[/Script/GameplayAbilities.AbilitySystemGlobals]
GlobalGameplayCueManagerClass="/Script/ParagonAssets.PBGameplayCueManager"
```

在我们的 `UGameplayCueManager` 子类中，重写 `ShouldAsyncLoadRuntimeObjectLibraries()`。

```c++
virtual bool ShouldAsyncLoadRuntimeObjectLibraries() const override
{
	return false;
}
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-gc-prevention"></a>
#### 4.8.6 阻止 Gameplay Cues 触发
有时我们不希望 `GameplayCues` 触发。例如，如果我们格挡攻击，我们可能不想播放附加到伤害 `GameplayEffect` 的命中冲击或播放自定义冲击。我们可以在 [`GameplayEffectExecutionCalculations`](#concepts-ge-ec) 中通过调用 `OutExecutionOutput.MarkGameplayCuesHandledManually()` 然后手动将我们的 `GameplayCue` 事件发送到 `Target` 或 `Source's` 的 `ASC` 来实现。

如果您永远不希望在任何特定的 `ASC` 上触发任何 `GameplayCues`，可以设置 `AbilitySystemComponent->bSuppressGameplayCues = true;`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-gc-batching"></a>
#### 4.8.7 Gameplay Cue 批处理
每个触发的 `GameplayCue` 都是一个不可靠的 NetMulticast RPC。在我们同时触发多个 `GC` 的情况下，有一些优化方法可以将它们压缩为一个 RPC 或通过发送更少的数据来节省带宽。

<a name="concepts-gc-batching-manualrpc"></a>
##### 4.8.7.1 手动 RPC
假设您有一把发射八发子弹的霰弹枪。那就是八个追踪和冲击 `GameplayCues`。[GASShooter](https://github.com/tranek/GASShooter) 采用懒惰的方法，通过将所有追踪信息存储为 [`EffectContext`](#concepts-ge-ec) 中的 [`TargetData`](#concepts-targeting-data) 将它们组合到一个 RPC 中。虽然这将 RPC 从八个减少到一个，但它仍然在一个 RPC 中通过网络发送大量数据（约 500 字节）。一种更优化的方法是发送一个带有自定义结构的 RPC，在其中有效地编码命中位置，或者您给它一个随机种子数来在接收端重新创建/近似冲击位置。然后，客户端将解包这个自定义结构并转回[本地执行的 `GameplayCues`](#concepts-gc-local)。

工作原理：
1. 声明一个 `FScopedGameplayCueSendContext`。这将抑制 `UGameplayCueManager::FlushPendingCues()`，直到它超出作用域，这意味着所有 `GameplayCues` 都将排队，直到 `FScopedGameplayCueSendContext` 超出作用域。
1. 重写 `UGameplayCueManager::FlushPendingCues()` 以将可以基于某些自定义 `GameplayTag` 批处理在一起的 `GameplayCues` 合并到您的自定义结构中并将其 RPC 到客户端。
1. 客户端接收自定义结构并将其解包为本地执行的 `GameplayCues`。

当您的 `GameplayCues` 需要特定参数而这些参数不符合 `GameplayCueParameters` 提供的内容，并且您不想将它们添加到 `EffectContext` 中（例如伤害数字、暴击指示器、破碎护盾指示器、致命命中指示器等）时，也可以使用此方法。

https://forums.unrealengine.com/development-discussion/c-gameplay-programming/1711546-fscopedgameplaycuesendcontext-gameplaycuemanager

<a name="concepts-gc-batching-gcsonge"></a>
##### 4.8.7.2 一个 GE 上的多个 GC
`GameplayEffect` 上的所有 `GameplayCues` 已经在一个 RPC 中发送。默认情况下，`UGameplayCueManager::InvokeGameplayCueAddedAndWhileActive_FromSpec()` 将在整个不可靠的 NetMulticast 中发送整个 `GameplayEffectSpec`（但转换为 `FGameplayEffectSpecForRPC`），无论 `ASC` 的 `复制模式` 如何。这可能会占用大量带宽，具体取决于 `GameplayEffectSpec` 中的内容。我们可以通过设置 cvar `AbilitySystem.AlwaysConvertGESpecToGCParams 1` 来潜在地优化这一点。这将把 `GameplayEffectSpecs` 转换为 `FGameplayCueParameter` 结构并 RPC 它们而不是整个 `FGameplayEffectSpecForRPC`。这可能节省带宽，但信息也较少，具体取决于 `GESpec` 如何转换为 `GameplayCueParameters` 以及您的 `GC` 需要知道什么。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-gc-events"></a>
#### 4.8.8 Gameplay Cue 事件
`GameplayCues` 响应特定的 `EGameplayCueEvents`：

| `EGameplayCueEvent` | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OnActive`          | 在 `GameplayCue` 激活（添加）时调用。                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `WhileActive`       | 当 `GameplayCue` 处于活动状态时调用，即使它实际上并没有刚刚应用（加入进行中等）。这不是 `Tick`！当添加 `GameplayCueNotify_Actor` 或变得相关时，它就像 `OnActive` 一样调用一次。如果您需要 `Tick()`，只需使用 `GameplayCueNotify_Actor` 的 `Tick()`。毕竟它是一个 `AActor`。                                                                                                                                                    |
| `Removed`           | 在 `GameplayCue` 被移除时调用。响应此事件的蓝图 `GameplayCue` 函数是 `OnRemove`。                                                                                                                                                                                                                                                                                                                                                                                                         |
| `Executed`          | 在 `GameplayCue` 执行时调用：即时效果或周期性 `Tick()`。响应此事件的蓝图 `GameplayCue` 函数是 `OnExecute`。                                                                                                                                                                                                                                                                                                                                                                               |

对于 `GameplayCue` 中在开始时发生但迟到加入者错过的内容，使用 `OnActive`。对于您希望迟到加入者看到的 `GameplayCue` 中的持续效果，使用 `WhileActive`。例如，如果您在 MOBA 中有一个用于塔结构爆炸的 `GameplayCue`，您会将初始爆炸粒子系统和爆炸声音放在 `OnActive` 中，并将任何残留的持续火焰粒子或声音放在 `WhileActive` 中。在这种情况下，迟到加入者重放 `OnActive` 的初始爆炸是没有意义的，但您希望它们在爆炸发生后从 `WhileActive` 看到地面上持续的、循环的火焰效果。`OnRemove` 应该清理在 `OnActive` 和 `WhileActive` 中添加的任何内容。每当 Actor 进入 `GameplayCueNotify_Actor` 的相关范围时，就会调用 `WhileActive`。每当 Actor 离开 `GameplayCueNotify_Actor` 的相关范围时，就会调用 `OnRemove`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-gc-reliability"></a>
#### 4.8.9 Gameplay Cue 可靠性

通常应将 `GameplayCues` 视为不可靠的，因此不适合任何直接影响游戏的内容。

**已执行的 `GameplayCues`**：这些 `GameplayCues` 通过不可靠的多播应用，并且始终是不可靠的。

**从 `GameplayEffects` 应用的 `GameplayCues`**：
* 自主代理可靠地接收 `OnActive`、`WhileActive` 和 `OnRemove`
`FActiveGameplayEffectsContainer::NetDeltaSerialize()` 调用 `UAbilitySystemComponent::HandleDeferredGameplayCues()` 来调用 `OnActive` 和 `WhileActive`。`FActiveGameplayEffectsContainer::RemoveActiveGameplayEffectGrantedTagsAndModifiers()` 进行 `OnRemoved` 的调用。
* 模拟代理可靠地接收 `WhileActive` 和 `OnRemove`
`UAbilitySystemComponent::MinimalReplicationGameplayCues` 的复制调用 `WhileActive` 和 `OnRemove`。`OnActive` 事件由不可靠的多播调用。

**没有 `GameplayEffect` 应用的 `GameplayCues`**：
* 自主代理可靠地接收 `OnRemove`
`OnActive` 和 `WhileActive` 事件由不可靠的多播调用。
* 模拟代理可靠地接收 `WhileActive` 和 `OnRemove`
`UAbilitySystemComponent::MinimalReplicationGameplayCues` 的复制调用 `WhileActive` 和 `OnRemove`。`OnActive` 事件由不可靠的多播调用。

如果您需要 `GameplayCue` 中的某些内容是"可靠的"，那么从 `GameplayEffect` 应用它并使用 `WhileActive` 添加 FX，使用 `OnRemove` 移除 FX。

**[⬆ 返回顶部](#table-of-contents)**


<a name="concepts-asg"></a>
### 4.9 Ability System Globals
[`AbilitySystemGlobals`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/UAbilitySystemGlobals/index.html) 类保存 GAS 的全局信息。大多数变量可以在 `DefaultGame.ini` 中设置。通常你不需要与这个类交互，但应该知道它的存在。如果需要子类化 [`GameplayCueManager`](#concepts-gc-manager) 或 [`GameplayEffectContext`](#concepts-ge-context) 等内容，必须通过 `AbilitySystemGlobals` 来实现。

要子类化 `AbilitySystemGlobals`，在 `DefaultGame.ini` 中设置类名：
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
AbilitySystemGlobalsClassName="/Script/ParagonAssets.PAAbilitySystemGlobals"
```

<a name="concepts-asg-initglobaldata"></a>
#### 4.9.1 InitGlobalData()
在 UE 4.24 到 5.2 版本之间，必须调用 `UAbilitySystemGlobals::Get().InitGlobalData()` 才能使用 [`TargetData`](#concepts-targeting-data)，否则会出现与 `ScriptStructCache` 相关的错误，客户端将从服务器断开连接。这个函数在项目中只需要调用一次。Fortnite 在 `UAssetManager::StartInitialLoading()` 中调用它，Paragon 在 `UEngine::Init()` 中调用。我发现将它放在 `UAssetManager::StartInitialLoading()` 中是一个好位置，如示例项目所示。我认为这应该是样板代码，应该复制到项目中以避免 `TargetData` 的问题。从 5.3 版本开始，它会自动调用。

如果在使用 `AbilitySystemGlobals` 的 `GlobalAttributeSetDefaultsTableNames` 时遇到崩溃，可能需要像 Fortnite 那样在 `AssetManager` 或 `GameInstance` 中稍后调用 `UAbilitySystemGlobals::Get().InitGlobalData()`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-p"></a>
### 4.10 Prediction
GAS 开箱即用地支持客户端预测；但是，它不会预测所有内容。GAS 中的客户端预测意味着客户端不必等待服务器的许可来激活 `GameplayAbility` 和应用 `GameplayEffects`。它可以"预测"服务器给予许可并预测将应用 `GameplayEffects` 的目标。服务器在客户端激活后网络延迟时间运行 `GameplayAbility`，并告知客户端其预测是否正确。如果客户端在任何预测中出错，它将从其"错误预测"中"回滚"更改以匹配服务器。

GAS 相关预测的权威来源是插件源代码中的 `GameplayPrediction.h`。

Epic 的思路是只预测你"可以侥幸逃脱"的内容。例如，Paragon 和 Fortnite 不预测伤害。大多数情况下，他们使用 [`ExecutionCalculations`](#concepts-ge-ec) 来处理伤害，而这是无法预测的。这并不是说不能尝试预测某些事情（如伤害）。如果你这样做并且效果很好，那就太好了。

> ... 我们也没有全力以赴实现"预测一切：无缝且自动"的解决方案。我们仍然认为玩家预测应保持在最低限度（意味着：预测可以侥幸逃脱的最小内容量）。

*Epic 的 Dave Ratti 对新的 [Network Prediction Plugin](#concepts-p-npp) 的评论*

**可预测的内容：**
> * Ability 激活
> * 触发的事件
> * GameplayEffect 应用：
>    * 属性修改（例外：Execution 目前不预测，仅预测属性修饰符）
>    * GameplayTag 修改
> * Gameplay Cue 事件（包括来自预测性 gameplay effect 和独立的）
> * Montages
> * 移动（内置在 UE 的 UCharacterMovement 中）

**不可预测的内容：**
> * GameplayEffect 移除
> * GameplayEffect 周期性效果（持续伤害 tick）

*来自 `GameplayPrediction.h`*

虽然我们可以预测 `GameplayEffect` 的应用，但无法预测 `GameplayEffect` 的移除。解决此限制的一种方法是，在想要移除 `GameplayEffect` 时预测反向效果。假设我们预测了 40% 的移动速度减速。我们可以通过应用 40% 的移动速度增益来预测性地移除它。然后同时移除这两个 `GameplayEffects`。这并非适用于所有场景，仍然需要支持预测 `GameplayEffect` 的移除。Epic 的 Dave Ratti 已表示有兴趣将其添加到 [未来版本的 GAS](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89) 中。

由于我们无法预测 `GameplayEffects` 的移除，因此无法完全预测 `GameplayAbility` 的冷却时间，也没有反向 `GameplayEffect` 的解决方法。服务器的复制 `Cooldown GE` 将存在于客户端上，任何绕过此操作的行为（例如使用 `Minimal` 复制模式）都将被服务器拒绝。这意味着延迟较高的客户端需要更长的时间才能告诉服务器进入冷却时间并接收服务器 `Cooldown GE` 的移除。这意味着延迟较高的玩家的射击速度将低于延迟较低的玩家，使他们在对抗低延迟玩家时处于劣势。Fortnite 通过使用自定义记录而不是 `Cooldown GEs` 来避免这个问题。

关于预测伤害，我个人不建议这样做，尽管这是大多数人开始使用 GAS 时首先尝试的事情之一。我特别不建议尝试预测死亡。虽然可以预测伤害，但这样做很棘手。如果错误地预测了伤害的应用，玩家将看到敌人的生命值跳回。如果尝试预测死亡，这可能会特别尴尬和令人沮丧。假设错误地预测了 `Character` 的死亡，它开始布娃娃模拟，但在服务器更正时停止布娃娃模拟并继续向你射击。

**注意**：更改 `Attributes` 的 `Instant` `GameplayEffects`（如 `Cost GEs`）可以在自己身上无缝预测，预测对其他角色的 `Instant` `Attribute` 更改将在其 `Attributes` 中显示短暂的异常或"闪烁"。预测的 `Instant` `GameplayEffects` 实际上被视为 `Infinite` `GameplayEffects`，以便在预测错误时可以回滚它们。当服务器的 `GameplayEffect` 应用时，可能存在两个相同的 `GameplayEffect's`，导致 `Modifier` 被应用两次或短暂地根本不应用。它最终会自我更正，但有时闪烁对玩家来说是明显的。

GAS 的预测实现试图解决的问题：
> 1. "我可以做这个吗？" 预测的基本协议。
> 2. "撤销" 当预测失败时如何撤销副作用。
> 3. "重做" 如何避免重放我们在本地预测但也从服务器复制的副作用。
> 4. "完整性" 如何确定我们 /真的/ 预测了所有副作用。
> 5. "依赖关系" 如何管理依赖性预测和预测事件链。
> 6. "覆盖" 如何预测性地覆盖否则由服务器复制/拥有的状态。

*来自 `GameplayPrediction.h`*

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-p-key"></a>
#### 4.10.1 Prediction Key
GAS 的预测基于 `Prediction Key` 的概念工作，这是一个整数标识符，客户端在激活 `GameplayAbility` 时生成它。

* 客户端在激活 `GameplayAbility` 时生成预测密钥。这是 `Activation Prediction Key`。
* 客户端通过 `CallServerTryActivateAbility()` 将此预测密钥发送到服务器。
* 客户端在预测密钥有效时将此预测密钥添加到其应用的所有 `GameplayEffects` 中。
* 客户端的预测密钥超出范围。同一 `GameplayAbility` 中的进一步预测效果需要新的 [Scoped Prediction Window](#concepts-p-windows)。


* 服务器从客户端接收预测密钥。
* 服务器将此预测密钥添加到其应用的所有 `GameplayEffects` 中。
* 服务器将预测密钥复制回客户端。


* 客户端从服务器接收带有用于应用它们的预测密钥的复制 `GameplayEffects`。如果任何复制的 `GameplayEffects` 与客户端使用相同预测密钥应用的 `GameplayEffects` 匹配，则它们已被正确预测。在目标上将暂时存在两个 `GameplayEffect` 副本，直到客户端移除其预测的那个。
* 客户端从服务器接收回预测密钥。这是 `Replicated Prediction Key`。此预测密钥现在被标记为陈旧。
* 客户端移除它使用现在陈旧的复制预测密钥创建的 **所有** `GameplayEffects`。服务器复制的 `GameplayEffects` 将持久保存。客户端添加的任何未收到服务器匹配复制版本的 `GameplayEffects` 都被错误预测。

预测密钥保证在从 `Activation` 激活预测密钥开始的 `GameplayAbilities` 中的指令"窗口"的原子分组期间有效。你可以将其视为仅在一帧内有效。来自潜在操作 `AbilityTasks` 的任何回调都不再具有有效的预测密钥，除非 `AbilityTask` 具有内置的同步点，该同步点会生成新的 [Scoped Prediction Window](#concepts-p-windows)。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-p-windows"></a>
#### 4.10.2 在 Abilities 中创建新的预测窗口
要在来自 `AbilityTasks` 的回调中预测更多操作，我们需要使用新的 Scoped Prediction Key 创建新的 Scoped Prediction Window。这有时被称为客户端和服务器之间的同步点。一些 `AbilityTasks`（如所有与输入相关的）具有内置功能来创建新的作用域预测窗口，这意味着 `AbilityTasks'` 回调中的原子代码具有有效的作用域预测密钥可供使用。其他任务（如 `WaitDelay` 任务）没有内置代码为其回调创建新的作用域预测窗口。如果需要在没有内置代码来创建作用域预测窗口的 `AbilityTask`（如 `WaitDelay`）之后预测操作，我们必须使用带有 `OnlyServerWait` 选项的 `WaitNetSync` `AbilityTask` 手动执行此操作。当客户端遇到带有 `OnlyServerWait` 的 `WaitNetSync` 时，它会基于 `GameplayAbility's` 激活预测密钥生成新的作用域预测密钥，通过 RPC 将其发送到服务器，并将其添加到它应用的任何新 `GameplayEffects` 中。当服务器遇到带有 `OnlyServerWait` 的 `WaitNetSync` 时，它会等待直到从客户端接收到新的作用域预测密钥，然后继续。此作用域预测密钥与激活预测密钥执行相同的操作 - 应用于 `GameplayEffects` 并复制回客户端以标记为陈旧。作用域预测密钥有效，直到它超出范围，这意味着作用域预测窗口已关闭。同样，只有原子操作，没有潜在操作，可以使用新的作用域预测密钥。

可以根据需要创建任意数量的作用域预测窗口。

如果想将同步点功能添加到自己的自定义 `AbilityTasks`，请查看输入任务如何基本地将 `WaitNetSync` `AbilityTask` 代码注入到它们中。

**注意**：使用 `WaitNetSync` 时，这确实会阻止服务器的 `GameplayAbility` 继续执行，直到它听到客户端的消息。这可能会被恶意用户滥用，他们入侵游戏并故意延迟发送新的作用域预测密钥。虽然 Epic 节制地使用 `WaitNetSync`，但如果这是一个问题，建议可能构建一个带有延迟的 `AbilityTask` 新版本，如果没有客户端，它会自动继续。

示例项目在 Sprint `GameplayAbility` 中使用 `WaitNetSync`，在每次应用体力消耗时创建新的作用域预测窗口，以便我们可以预测它。理想情况下，我们希望在应用消耗和冷却时拥有有效的预测密钥。

如果拥有一个在拥有客户端上播放两次的预测 `GameplayEffect`，则预测密钥已陈旧，并且你遇到了"重做"问题。通常可以通过在应用 `GameplayEffect` 之前放置带有 `OnlyServerWait` 的 `WaitNetSync` `AbilityTask` 来解决此问题，以创建新的作用域预测密钥。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-p-spawn"></a>
#### 4.10.3 预测性生成 Actors
在客户端上预测性地生成 `Actors` 是一个高级主题。GAS 不提供开箱即用地处理此功能的功能（`SpawnActor` `AbilityTask` 仅在服务器上生成 `Actor`）。关键概念是在客户端和服务器上都生成一个复制的 `Actor`。

如果 `Actor` 只是装饰性的或没有任何游戏用途，简单的解决方案是覆盖 `Actor's` `IsNetRelevantFor()` 函数，以限制服务器向拥有客户端复制。拥有客户端将拥有其本地生成的版本，服务器和其他客户端将拥有服务器的复制版本。
```c++
bool APAReplicatedActorExceptOwner::IsNetRelevantFor(const AActor * RealViewer, const AActor * ViewTarget, const FVector & SrcLocation) const
{
	return !IsOwnedBy(ViewTarget);
}
```

如果生成的 `Actor` 影响游戏玩法（如需要预测伤害的抛射物），则需要超出本文档范围的高级逻辑。查看 UnrealTournament 如何在 Epic Games 的 GitHub 上预测性地生成抛射物。它们有一个仅在拥有客户端上生成的虚拟抛射物，与服务器的复制抛射物同步。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-p-future"></a>
#### 4.10.4 GAS 中预测的未来
`GameplayPrediction.h` 指出，将来他们可能会添加预测 `GameplayEffect` 移除和周期性 `GameplayEffects` 的功能。

Epic 的 Dave Ratti 已[表示有兴趣](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)修复预测冷却的 `延迟协调` 问题，这使延迟较高的玩家相对于延迟较低的玩家处于不利地位。

Epic 的新 [`Network Prediction` 插件](#concepts-p-npp)预计将与 GAS 完全互操作，就像之前的 `CharacterMovementComponent` 一样。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-p-npp"></a>
#### 4.10.5 Network Prediction 插件
Epic 最近开始了一项举措，用新的 `Network Prediction` 插件替换 `CharacterMovementComponent`。这个插件仍处于非常早期的阶段，但在 Unreal Engine GitHub 上提供非常早期的访问。现在说它将在引擎的哪个未来版本中首次亮相实验性 beta 版本还为时过早。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-targeting"></a>
### 4.11 Targeting

<a name="concepts-targeting-data"></a>
#### 4.11.1 Target Data
[`FGameplayAbilityTargetData`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetData/index.html) 是用于通过网络传递的目标数据的通用结构。`TargetData` 通常保存 `AActor`/`UObject` 引用、`FHitResults` 和其他通用位置/方向/源信息。但是，你可以将其子类化，以便在其中放置基本上任何想要的内容，作为[在 `GameplayAbilities` 中在客户端和服务器之间传递数据](#concepts-ga-data)的简单方法。基础结构 `FGameplayAbilityTargetData` 不打算直接使用，而是子类化。`GAS` 开箱即用地提供了几个子类化的 `FGameplayAbilityTargetData` 结构，位于 `GameplayAbilityTargetTypes.h` 中。

`TargetData` 通常由 [`Target Actors`](#concepts-targeting-actors) 或**手动创建**生成，并由 [`AbilityTasks`](#concepts-at) 和 [`GameplayEffects`](#concepts-ge) 通过 [`EffectContext`](#concepts-ge-context) 使用。由于位于 `EffectContext` 中，[`Executions`](#concepts-ge-ec)、[`MMCs`](#concepts-ge-mmc)、[`GameplayCues`](#concepts-gc) 和 [`AttributeSet`](#concepts-as) 后端上的函数可以访问 `TargetData`。

我们通常不直接传递 `FGameplayAbilityTargetData`，而是使用 [`FGameplayAbilityTargetDataHandle`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FGameplayAbilityTargetDataHandle/index.html)，它具有指向 `FGameplayAbilityTargetData` 的内部 TArray 指针。这个中间结构为 `TargetData` 的多态性提供了支持。

从 `FGameplayAbilityTargetData` 继承的示例：
```c++
USTRUCT(BlueprintType)
struct MYGAME_API FGameplayAbilityTargetData_CustomData : public FGameplayAbilityTargetData
{
    GENERATED_BODY()
public:

    FGameplayAbilityTargetData_CustomData()
    { }

    UPROPERTY()
    FName CoolName = NAME_None;

    UPROPERTY()
    FPredictionKey MyCoolPredictionKey;

    // 所有 FGameplayAbilityTargetData 的子结构都需要此功能
    virtual UScriptStruct* GetScriptStruct() const override
    {
    	return FGameplayAbilityTargetData_CustomData::StaticStruct();
    }

	// 所有 FGameplayAbilityTargetData 的子结构都需要此功能
    bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
    {
	    // 引擎已经为 FName 和 FPredictionKey 定义了 NetSerialize，感谢 Epic！
        CoolName.NetSerialize(Ar, Map, bOutSuccess);
        MyCoolPredictionKey.NetSerialize(Ar, Map, bOutSuccess);
        bOutSuccess = true;
        return true;
    }
}

template<>
struct TStructOpsTypeTraits<FGameplayAbilityTargetData_CustomData> : public TStructOpsTypeTraitsBase2<FGameplayAbilityTargetData_CustomData>
{
	enum
	{
        WithNetSerializer = true // 这是 FGameplayAbilityTargetDataHandle 网络序列化工作所必需的
	};
};
```
对于将目标数据添加到句柄：
```c++
UFUNCTION(BlueprintPure)
FGameplayAbilityTargetDataHandle MakeTargetDataFromCustomName(const FName CustomName)
{
	// 创建目标数据类型，
	// 句柄在销毁时会自动清理和删除此数据，
	// 如果不将其添加到句柄中，请小心，因为它涉及内存管理和内存泄漏，因此将其添加到句柄中是安全的！
	FGameplayAbilityTargetData_CustomData* MyCustomData = new FGameplayAbilityTargetData_CustomData();
	// 设置结构的信息以使用输入的名称以及可能想要进行的任何其他更改
	MyCustomData->CoolName = CustomName;

	// 为 Blueprint 使用制作句柄包装器
	FGameplayAbilityTargetDataHandle Handle;
	// 将目标数据添加到句柄
	Handle.Add(MyCustomData);
	// 将句柄输出到 Blueprint
	return Handle
}
```

对于获取值，需要进行类型安全检查，因为从句柄的目标数据获取值的唯一方法是使用通用的 C/C++ 转换，这*不是*类型安全的，可能导致对象切片和崩溃。对于类型检查，有多种方法可以执行此操作（实际上可以根据需要），两种常见的方法是：
- Gameplay Tag：可以使用子类层次结构，只要知道何时发生特定代码架构的功能，就可以转换为基本父类型并获取其 gameplay tag，然后与这些进行比较以转换继承类。
- Script Struct 和 Static Structs：可以改为进行直接的类比较（这可能涉及很多 IF 语句或制作一些模板函数），下面是执行此操作的示例，但基本上可以从任何 `FGameplayAbilityTargetData` 获取脚本结构（这是 `USTRUCT` 的一个很好的优势，并且要求任何继承类在 `GetScriptStruct` 中指定结构类型）并比较它是否是正在查找的类型。以下是使用这些函数进行类型检查的示例：
```c++
UFUNCTION(BlueprintPure)
FName GetCoolNameFromTargetData(const FGameplayAbilityTargetDataHandle& Handle, const int Index)
{
    // 注意，此 '::Get(int32 Index)' 函数有两个版本；
    // 1) const 版本，返回 'const FGameplayAbilityTargetData*'，适用于读取目标数据值
    // 2) 非 const 版本，返回 'FGameplayAbilityTargetData*'，适用于修改目标数据值
    FGameplayAbilityTargetData* Data = Handle.Get(Index); // 这将为你有效检查索引

    // 有效检查我们有一些可以使用的东西，空数据意味着没有可转换的内容
    if(Data == nullptr)
    {
    	return NAME_None;
    }
    // 这基本上是类型检查过程，static_cast 没有类型安全，这就是我们要进行此检查的原因。
    // 如果我们不这样做，它将对结构进行对象切片，因此我们无法确定它是该类型。
    if(Data->GetScriptStruct() == FGameplayAbilityTargetData_CustomData::StaticStruct())
    {
        // 这是你要进行转换的时候，因为我们知道它已经是正确的类型
        FGameplayAbilityTargetData_CustomData* CustomData = static_cast<FGameplayAbilityTargetData_CustomData*>(Data);
        return CustomData->CoolName;
    }
    return NAME_None;
}
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-targeting-actors"></a>
#### 4.11.2 Target Actors
`GameplayAbilities` 使用 `WaitTargetData` `AbilityTask` 生成 [`TargetActors`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor/index.html) 来可视化和捕获来自世界的目标信息。`TargetActors` 可以选择使用 [`GameplayAbilityWorldReticles`](#concepts-targeting-reticles) 来显示当前目标。确认后，目标信息作为 [`TargetData`](#concepts-targeting-data) 返回，然后可以传递到 `GameplayEffects` 中。

`TargetActors` 基于 `AActor`，因此它们可以具有任何类型的可见组件来表示它们定位的**位置**和**方式**，例如静态网格或贴花。静态网格可用于可视化角色将构建的对象的放置。贴花可用于显示地面上的效果区域。示例项目使用 [`AGameplayAbilityTargetActor_GroundTrace`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityTargetActor_Grou-/index.html) 和地面上的贴花来表示 Meteor 能力的伤害效果区域。它们也不需要显示任何内容。例如，对于瞬间追踪到其目标的命中扫描枪（如 [GASShooter](https://github.com/tranek/GASShooter) 中使用的），显示任何内容都没有意义。

它们使用基本的追踪或碰撞重叠来捕获目标信息，并根据 `TargetActor` 的实现将结果转换为 `FHitResults` 或 `AActor` 数组到 `TargetData`。`WaitTargetData` `AbilityTask` 通过其 `TEnumAsByte<EGameplayTargetingConfirmation::Type> ConfirmationType` 参数确定何时确认目标。当**不**使用 `TEnumAsByte<EGameplayTargetingConfirmation::Type::Instant` 时，`TargetActor` 通常在 `Tick()` 上执行追踪/重叠，并根据其实现将其位置更新到 `FHitResult`。虽然这在 `Tick()` 上执行追踪/重叠，但通常并不糟糕，因为它不是复制的，并且通常一次不会运行超过一个（尽管可以有多个）`TargetActor`。请注意，它使用 `Tick()`，一些复杂的 `TargetActors` 可能会在它上面做很多事情，例如 GASShooter 中的火箭发射器辅助能力。虽然在 `Tick()` 上追踪对客户端非常响应，但如果性能影响太大，可以考虑降低 `TargetActor` 的 tick 率。在 `TEnumAsByte<EGameplayTargetingConfirmation::Type::Instant` 的情况下，`TargetActor` 立即生成，生成 `TargetData` 并销毁。从不调用 `Tick()`。

| `EGameplayTargetingConfirmation::Type` | 何时确认目标                                                                                                                                                                                                                                                                                                                                     |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Instant`                              | 定位即时发生，没有特殊逻辑或用户输入决定何时"开火"。                                                                                                                                                                                                                                                                   |
| `UserConfirmed`                        | 当用户确认定位时，定位发生，通过[能力绑定到 `Confirm` 输入](#concepts-ga-input)或调用 `UAbilitySystemComponent::TargetConfirm()`。`TargetActor` 还将响应绑定的 `Cancel` 输入或调用 `UAbilitySystemComponent::TargetCancel()` 来取消定位。                              |
| `Custom`                               | GameplayTargeting Ability 负责通过调用 `UGameplayAbility::ConfirmTaskByInstanceName()` 决定定位数据何时准备就绪。`TargetActor` 还将响应 `UGameplayAbility::CancelTaskByInstanceName()` 来取消定位。                                                                                              |
| `CustomMulti`                          | GameplayTargeting Ability 负责通过调用 `UGameplayAbility::ConfirmTaskByInstanceName()` 决定定位数据何时准备就绪。`TargetActor` 还将响应 `UGameplayAbility::CancelTaskByInstanceName()` 来取消定位。不应在数据生成时结束 `AbilityTask`。                                       |

并非每个 EGameplayTargetingConfirmation::Type 都受每个 `TargetActor` 支持。例如，`AGameplayAbilityTargetActor_GroundTrace` 不支持 `Instant` 确认。

`WaitTargetData` `AbilityTask` 接受 `AGameplayAbilityTargetActor` 类作为参数，并在每次激活 `AbilityTask` 时生成一个实例，并在 `AbilityTask` 结束时销毁 `TargetActor`。`WaitTargetDataUsingActor` `AbilityTask` 接受已生成的 `TargetActor`，但在 `AbilityTask` 结束时仍会销毁它。这两个 `AbilityTasks` 效率低下，因为它们要么生成，要么为每次使用新生成的 `TargetActor`。它们非常适合原型设计，但在生产环境中，如果遇到不断生成 `TargetData` 的情况（例如自动步枪的情况），可以探索优化它。GASShooter 具有一个自定义子类 [`AGameplayAbilityTargetActor`](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/GSGATA_Trace.h) 和一个从头开始编写的新 [`WaitTargetDataWithReusableActor`](https://github.com/tranek/GASShooter/blob/master/Source/GASShooter/Public/Characters/Abilities/AbilityTasks/GSAT_WaitTargetDataUsingActor.h) `AbilityTask`，允许你重用 `TargetActor` 而不销毁它。

默认情况下，`TargetActors` 不会被复制；但是，如果在游戏中显示其他玩家本地玩家的目标位置是有意义的，则可以使其复制。它们确实包含默认功能，可以通过 `WaitTargetData` `AbilityTask` 上的 RPC 与服务器通信。如果 `TargetActor` 的 `ShouldProduceTargetDataOnServer` 属性设置为 `false`，则客户端将在确认时通过 `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()` 中的 `CallServerSetReplicatedTargetData()` 将其 `TargetData` 通过 RPC 发送到服务器。如果 `ShouldProduceTargetDataOnServer` 为 `true`，则客户端将在 `UAbilityTask_WaitTargetData::OnTargetDataReadyCallback()` 中向服务器发送通用确认事件 `EAbilityGenericReplicatedEvent::GenericConfirm` RPC，服务器将在收到 RPC 后进行追踪或重叠检查以在服务器上生成数据。如果客户端取消定位，它将在 `UAbilityTask_WaitTargetData::OnTargetDataCancelledCallback` 中向服务器发送通用取消事件 `EAbilityGenericReplicatedEvent::GenericCancel` RPC。如你所见，`TargetActor` 和 `WaitTargetData` `AbilityTask` 上有很多委托。`TargetActor` 响应输入以生成和广播 `TargetData` 准备就绪、确认或取消委托。`WaitTargetData` 监听 `TargetActor` 的 `TargetData` 准备就绪、确认和取消委托，并将其中继回 `GameplayAbility` 和服务器。如果向服务器发送 `TargetData`，可能希望在服务器上进行验证，以确保 `TargetData` 看起来合理，以防止作弊。直接在服务器上生成 `TargetData` 可以完全避免此问题，但可能会导致拥有客户端的错误预测。

根据使用的 `AGameplayAbilityTargetActor` 的特定子类，`WaitTargetData` `AbilityTask` 节点上将暴露不同的 `ExposeOnSpawn` 参数。一些常见参数包括：

| 常见 `TargetActor` 参数 | 定义                                                                                                                                                                                                                                                                                                               |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Debug                           | 如果为 `true`，它将在非发布版本中的 `TargetActor` 执行追踪时绘制调试追踪/重叠信息。请记住，非 `Instant` `TargetActors` 将在 `Tick()` 上执行追踪，因此这些调试绘制调用也将在 `Tick()` 上发生。                                                        |
| Filter                          | [可选] 一个特殊的结构，用于在追踪/重叠发生时从目标中过滤掉（移除）`Actors`。典型用例是过滤掉玩家的 `Pawn`，要求目标是特定类。有关更高级的用例，请参阅 [Target Data Filters](#concepts-target-data-filters)。 |
| Reticle Class                   | [可选] `TargetActor` 将生成的 `AGameplayAbilityWorldReticle` 的子类。                                                                                                                                                                                                                                 |
| Reticle Parameters              | [可选] 配置 Reticles。请参阅 [Reticles](#concepts-targeting-reticles)。                                                                                                                                                                                                                                        |
| Start Location                  | 追踪应从何处开始的特殊结构。通常是玩家的视点、武器枪口或 `Pawn` 的位置。                                                                                                                                                                          |

使用默认的 `TargetActor` 类，`Actors` 仅在直接位于追踪/重叠中时才是有效目标。如果它们离开追踪/重叠（它们移动或你移开视线），它们将不再有效。如果希望 `TargetActor` 记住上一个有效目标，则需要将此功能添加到自定义 `TargetActor` 类。我将这些称为持久目标，因为它们将持续存在，直到 `TargetActor` 收到确认或取消、`TargetActor` 在其追踪/重叠中找到新的有效目标或目标不再有效（被销毁）。GASShooter 为其火箭发射器辅助能力的跟踪火箭定位使用持久目标。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-target-data-filters"></a>
#### 4.11.3 Target Data Filters
使用 `Make GameplayTargetDataFilter` 和 `Make Filter Handle` 节点，可以过滤掉玩家的 `Pawn` 或仅选择特定类。如果需要更高级的过滤，可以子类化 `FGameplayTargetDataFilter` 并覆盖 `FilterPassesForActor` 函数。
```c++
USTRUCT(BlueprintType)
struct GASDOCUMENTATION_API FGDNameTargetDataFilter : public FGameplayTargetDataFilter
{
	GENERATED_BODY()

	/** 如果 actor 通过过滤器并将被定位，则返回 true */
	virtual bool FilterPassesForActor(const AActor* ActorToBeFiltered) const override;
};
```

但是，这不能直接用于 `Wait Target Data` 节点，因为它需要 `FGameplayTargetDataFilterHandle`。必须创建新的自定义 `Make Filter Handle` 来接受子类：
```c++
FGameplayTargetDataFilterHandle UGDTargetDataFilterBlueprintLibrary::MakeGDNameFilterHandle(FGDNameTargetDataFilter Filter, AActor* FilterActor)
{
	FGameplayTargetDataFilter* NewFilter = new FGDNameTargetDataFilter(Filter);
	NewFilter->InitializeFilterContext(FilterActor);

	FGameplayTargetDataFilterHandle FilterHandle;
	FilterHandle.Filter = TSharedPtr<FGameplayTargetDataFilter>(NewFilter);
	return FilterHandle;
}
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-targeting-reticles"></a>
#### 4.11.4 Gameplay Ability World Reticles
[`AGameplayAbilityWorldReticles`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/AGameplayAbilityWorldReticle/index.html)（`Reticles`）在使用非 `Instant` 确认的 [`TargetActors`](#concepts-targeting-actors) 进行定位时可视化**谁**是你定位的目标。`TargetActors` 负责所有 `Reticles` 的生成和销毁生命周期。`Reticles` 是 `AActors`，因此它们可以使用任何类型的视觉组件进行表示。如 [GASShooter](https://github.com/tranek/GASShooter) 中看到的常见实现是使用 `WidgetComponent` 在屏幕空间中显示 UMG Widget（始终面向玩家的相机）。`Reticles` 不知道它们在哪个 `AActor` 上，但你可以在自定义 `TargetActor` 中子类化该功能。`TargetActors` 通常会在每次 `Tick()` 时将 `Reticle` 的位置更新到目标的位置。

GASShooter 使用 `Reticles` 来显示火箭发射器辅助能力的跟踪火箭的锁定目标。敌人身上的红色指示器是 `Reticle`。类似的白色图像是火箭发射器的准星。
![GASShooter 中的 Reticles](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplayabilityworldreticle.png)

`Reticles` 附带了一些 `BlueprintImplementableEvents` 供设计人员使用（它们旨在用 Blueprints 开发）：

```c++
/** 每当 bIsTargetValid 更改值时调用。 */
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnValidTargetChanged(bool bNewValue);

/** 每当 bIsTargetAnActor 更改值时调用。 */
UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnTargetingAnActor(bool bNewValue);

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void OnParametersInitialized();

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void SetReticleMaterialParamFloat(FName ParamName, float value);

UFUNCTION(BlueprintImplementableEvent, Category = Reticle)
void SetReticleMaterialParamVector(FName ParamName, FVector value);
```

`Reticles` 可以选择使用 `TargetActor` 提供的 [`FWorldReticleParameters`](https://docs.unrealengine.com/en-US/API/Plugins/GameplayAbilities/Abilities/FWorldReticleParameters/index.html) 进行配置。默认结构只提供一个变量 `FVector AOEScale`。虽然技术上可以子类化此结构，但 `TargetActor` 只会接受基础结构。不允许使用默认 `TargetActors` 子类化此结构似乎有点短视。但是，如果制作自己的自定义 `TargetActor`，则可以提供自己的自定义准星参数结构，并在生成它们时手动将其传递给 `AGameplayAbilityWorldReticles` 的子类。

默认情况下，`Reticles` 不会被复制，但如果在游戏中显示其他玩家本地玩家的目标是有意义的，则可以使其复制。

使用默认 `TargetActors`，`Reticles` 将仅显示在当前有效目标上。例如，如果使用 `AGameplayAbilityTargetActor_SingleLineTrace` 来追踪目标，则只有当敌人直接位于追踪路径中时，`Reticle` 才会出现。如果移开视线，敌人不再是有效目标，`Reticle` 将消失。如果希望 `Reticle` 保持在最后一个有效目标上，则需要自定义 `TargetActor` 以记住最后一个有效目标并保持 `Reticle` 在其上。我将这些称为持久目标，因为它们将持续存在，直到 `TargetActor` 收到确认或取消、`TargetActor` 在其追踪/重叠中找到新的有效目标或目标不再有效（被销毁）。GASShooter 为其火箭发射器辅助能力的跟踪火箭定位使用持久目标。

**[⬆ 返回顶部](#table-of-contents)**

<a name="concepts-targeting-containers"></a>
#### 4.11.5 Gameplay Effect Containers 定位
[`GameplayEffectContainers`](#concepts-ge-containers) 带有可选的、高效的生成 [`TargetData`](#concepts-targeting-data) 的方法。此定位在客户端和服务器上应用 `EffectContainer` 时即时进行。它比 [`TargetActors`](#concepts-targeting-actors) 更有效，因为它在定位对象的 CDO 上运行（没有生成和销毁 `Actors`），但它缺乏玩家输入，即时发生而无需确认，无法取消，并且无法从客户端向服务器发送数据（在两者上都生成数据）。它非常适用于即时追踪和碰撞重叠。Epic 的 [Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg) 包括两种使用其容器的定位类型示例 - 定位能力所有者并从事件中提取 `TargetData`。它还在 Blueprint 中实现了一种，以在玩家的某个偏移量（由子 Blueprint 类设置）处进行即时球体追踪。可以在 C++ 或 Blueprint 中子类化 `URPGTargetType` 以制作自己的定位类型。

**[⬆ 返回顶部](#table-of-contents)**


<a name="cae"></a>
## 5. 常用的能力和效果实现

<a name="cae-stun"></a>
### 5.1 眩晕
通常对于眩晕，我们希望取消 `Character` 的所有活动 `GameplayAbilities`，防止新的 `GameplayAbility` 激活，并在眩晕期间阻止移动。示例项目的陨石 `GameplayAbility` 会在命中目标时施加眩晕。

要取消目标的活动 `GameplayAbilities`，我们在[添加眩晕 `GameplayTag` 时](#concepts-gt-change)调用 `AbilitySystemComponent->CancelAbilities()`。

为了在眩晕时防止新的 `GameplayAbilities` 激活，`GameplayAbilities` 在其[`激活阻止标签` `GameplayTagContainer`](#concepts-ga-tags)中被赋予眩晕 `GameplayTag`。

为了在眩晕时阻止移动，我们覆盖 `CharacterMovementComponent` 的 `GetMaxSpeed()` 函数，在所有者拥有眩晕 `GameplayTag` 时返回 0。

**[⬆ 返回顶部](#table-of-contents)**

<a name="cae-sprint"></a>
### 5.2 冲刺
示例项目提供了如何冲刺的示例 - 在按住 `Left Shift` 时跑得更快。

更快的移动由 `CharacterMovementComponent` 通过网络向服务器发送标志来预测性地处理。有关详细信息，请参阅 `GDCharacterMovementComponent.h/cpp`。

`GA` 负责响应 `Left Shift` 输入，告诉 `CMC` 开始和停止冲刺，并在按下 `Left Shift` 时预测性地消耗体力。有关详细信息，请参阅 `GA_Sprint_BP`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="cae-ads"></a>
### 5.3 瞄准下望
示例项目处理此问题的方式与冲刺完全相同，但是是降低移动速度而不是提高移动速度。

有关预测性地降低移动速度的详细信息，请参阅 `GDCharacterMovementComponent.h/cpp`。

有关处理输入的详细信息，请参阅 `GA_AimDownSight_BP`。瞄准下望没有体力消耗。

**[⬆ 返回顶部](#table-of-contents)**

<a name="cae-ls"></a>
### 5.4 生命偷取
我在伤害[`ExecutionCalculation`](#concepts-ge-ec)中处理生命偷取。`GameplayEffect` 将在其上具有一个 `GameplayTag`，如 `Effect.CanLifesteal`。`ExecutionCalculation` 检查 `GameplayEffectSpec` 是否具有该 `Effect.CanLifesteal` `GameplayTag`。如果 `GameplayTag` 存在，`ExecutionCalculation` [会创建一个动态的 `Instant` `GameplayEffect`](#concepts-ge-dynamic)，其修饰符的数量是要给予的生命值，并将其应用回 `Source's` `ASC`。

```c++
if (SpecAssetTags.HasTag(FGameplayTag::RequestGameplayTag(FName("Effect.Damage.CanLifesteal"))))
{
	float Lifesteal = Damage * LifestealPercent;

	UGameplayEffect* GELifesteal = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("Lifesteal")));
	GELifesteal->DurationPolicy = EGameplayEffectDurationType::Instant;

	int32 Idx = GELifesteal->Modifiers.Num();
	GELifesteal->Modifiers.SetNum(Idx + 1);
	FGameplayModifierInfo& Info = GELifesteal->Modifiers[Idx];
	Info.ModifierMagnitude = FScalableFloat(Lifesteal);
	Info.ModifierOp = EGameplayModOp::Additive;
	Info.Attribute = UPAAttributeSetBase::GetHealthAttribute();

	SourceAbilitySystemComponent->ApplyGameplayEffectToSelf(GELifesteal, 1.0f, SourceAbilitySystemComponent->MakeEffectContext());
}
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="cae-random"></a>
### 5.5 在客户端和服务器上生成随机数
有时您需要在 `GameplayAbility` 内生成一个"随机"数，用于诸如后坐力或散射之类的事情。客户端和服务器都希望生成相同的随机数。为此，我们必须在 `GameplayAbility` 激活时将 `random seed` 设置为相同。您应该每次激活 `GameplayAbility` 时都设置 `random seed`，以防客户端错误预测激活并且其随机数序列与服务器的不同步。

| 种子设置方法                                                          | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 使用激活预测键                                            | `GameplayAbility` 激活预测键是一个 int16，保证在 `Activation()` 中的客户端和服务器上同步并可用。您可以将其设置客户端和服务器上的 `random seed`。此方法的缺点是预测键在每次游戏启动时始终从零开始，并一致地增加用于生成键之间的值。这意味着每场比赛将有完全相同的随机数序列。这对您的需求来说可能或可能不够随机。 |
| 在激活 `GameplayAbility` 时通过事件负载发送种子 | 通过事件激活您的 `GameplayAbility`，并通过复制的事件负载将客户端生成的随机种子发送到服务器。这允许更多的随机性，但客户端可以轻易地黑入他们的游戏每次只发送相同的种子值。此外，通过事件激活 `GameplayAbilities` 将阻止它们从输入绑定激活。                                                                                                                                                                     |

如果您的随机偏差很小，大多数玩家不会注意到序列在每场比赛中都是相同的，并且使用激活预测键作为 `random seed` 应该适合您。如果您正在做更复杂的事情，需要防止黑客攻击，也许使用 `Server Initiated` `GameplayAbility` 会更好，服务器可以在其中创建预测键或生成要通过事件负载发送的 `random seed`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="cae-crit"></a>
### 5.6 暴击
我在伤害[`ExecutionCalculation`](#concepts-ge-ec)中处理暴击。`GameplayEffect` 将在其上具有一个 `GameplayTag`，如 `Effect.CanCrit`。`ExecutionCalculation` 检查 `GameplayEffectSpec` 是否具有该 `Effect.CanCrit` `GameplayTag`。如果 `GameplayTag` 存在，`ExecutionCalculation` 会生成一个对应于暴击机会的随机数（从 `Source` 捕获的 `Attribute`），如果成功则添加暴击伤害（也是从 `Source` 捕获的 `Attribute`）。由于我不预测伤害，我不必担心同步客户端和服务器上的随机数生成器，因为 `ExecutionCalculation` 只会在服务器上运行。如果您尝试使用 `MMC` 进行伤害计算来预测性地执行此操作，您必须从 `GameplayEffectSpec->GameplayEffectContext->GameplayAbilityInstance` 获取对 `random seed` 的引用。

请参阅 [GASShooter](https://github.com/tranek/GASShooter) 如何处理爆头。这是相同的概念，除了它不依赖随机数来进行机会，而是检查 `FHitResult` 骨骼名称。

**[⬆ 返回顶部](#table-of-contents)**

<a name="cae-nonstackingge"></a>
### 5.7 非堆叠 Gameplay Effects 但只有最大幅度实际影响目标
Paragon 中的减速效果不堆叠。每个减速实例应用并正常跟踪其生命周期，但只有最大幅度的减速效果实际上影响 `Character`。GAS 通过 `AggregatorEvaluateMetaData` 开箱即用地为此场景提供支持。有关详细信息和实现，请参阅 [`AggregatorEvaluateMetaData()`](#concepts-as-onattributeaggregatorcreated)。

**[⬆ 返回顶部](#table-of-contents)**

<a name="cae-paused"></a>
### 5.8 游戏暂停时生成目标数据
如果您需要在等待从玩家的 `WaitTargetData` `AbilityTask` 生成[`TargetData`](#concepts-targeting-data)时暂停游戏，我建议不要暂停，而是使用 `slomo 0`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="cae-onebuttoninteractionsystem"></a>
### 5.9 单按钮交互系统
[GASShooter](https://github.com/tranek/GASShooter) 实现了一个单按钮交互系统，玩家可以按或按住 'E' 与可交互对象交互，例如复活玩家、打开武器箱以及打开或关闭滑动门。

**[⬆ 返回顶部](#table-of-contents)**

<a name="debugging"></a>
## 6. 调试 GAS
在调试 GAS 相关问题时，您通常想了解以下信息：
> * "我的属性值是多少？"
> * "我有哪些 gameplay tags？"
> * "我当前有哪些 gameplay effects？"
> * "我已授予哪些能力，哪些正在运行，哪些被阻止激活？"

GAS 提供了两种在运行时回答这些问题的技术 - [`showdebug abilitysystem`](#debugging-sd) 和 [`GameplayDebugger`](#debugging-gd) 中的钩子。

**提示**：Unreal Engine 喜欢优化 C++ 代码，这使得调试某些函数变得困难。在深入追踪代码时，您很少会遇到这种情况。如果将 Visual Studio 解决方案配置设置为 `DebugGame Editor` 仍然无法追踪代码或检查变量，可以通过使用 `UE_DISABLE_OPTIMIZATION` 和 `UE_ENABLE_OPTIMIZATION` 宏或 CoreMiscDefines.h 中定义的 ship 变体来包装优化的函数来禁用所有优化。除非您从源代码重新构建插件，否则不能在插件代码上使用此功能。这可能适用于或不适用于内联函数，具体取决于它们的作用和位置。完成调试后务必删除宏！

```c++
UE_DISABLE_OPTIMIZATION
void MyClass::MyFunction(int32 MyIntParameter)
{
	// My code
}
UE_ENABLE_OPTIMIZATION
```

**[⬆ 返回顶部](#table-of-contents)**

<a name="debugging-sd"></a>
### 6.1 showdebug abilitysystem
在游戏内控制台中输入 `showdebug abilitysystem`。此功能分为三个"页面"。所有三个页面都将显示您当前拥有的 `GameplayTags`。在控制台中输入 `AbilitySystem.Debug.NextCategory` 以在页面之间循环。

第一页显示所有 `Attributes` 的 `CurrentValue`：
![showdebug abilitysystem 第一页](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage1.png)

第二页显示您身上的所有 `Duration` 和 `Infinite` `GameplayEffects`、它们的堆叠数、它们给予的 `GameplayTags` 以及它们给予的 `Modifiers`。
![showdebug abilitysystem 第二页](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage2.png)

第三页显示已授予您的所有 `GameplayAbilities`，以及它们是否正在运行、是否被阻止激活以及当前正在运行的 `AbilityTasks` 的状态。
![showdebug abilitysystem 第三页](https://github.com/tranek/GASDocumentation/raw/master/Images/showdebugpage3.png)

要在目标之间循环（由 Actor 周围的绿色长方体表示），使用 `PageUp` 键或 `NextDebugTarget` 控制台命令转到下一个目标，使用 `PageDown` 键或 `PreviousDebugTarget` 控制台命令转到上一个目标。

**注意**：为了使能力系统信息根据当前选定的调试 Actor 更新，您需要在 `AbilitySystemGlobals` 中设置 `bUseDebugTargetFromHud=true`，如下所示在 `DefaultGame.ini` 中：
```
[/Script/GameplayAbilities.AbilitySystemGlobals]
bUseDebugTargetFromHud=true
```

**注意**：为了使 `showdebug abilitysystem` 工作，必须在 GameMode 中选择实际的 HUD 类。否则找不到该命令并返回"未知命令"。

**[⬆ 返回顶部](#table-of-contents)**

<a name="debugging-gd"></a>
### 6.2 Gameplay Debugger
GAS 向 Gameplay Debugger 添加了功能。使用撇号 (') 键访问 Gameplay Debugger。通过按数字键盘上的 3 启用 Abilities 类别。类别可能因您拥有的插件而异。如果您的键盘没有像笔记本电脑那样的数字键盘，那么您可以在项目设置中更改键绑定。

当您想查看**其他** `Characters` 上的 `GameplayTags`、`GameplayEffects` 和 `GameplayAbilities` 时，请使用 Gameplay Debugger。不幸的是，它不显示目标的 `Attributes` 的 `CurrentValue`。它将以屏幕中心的 `Character` 为目标。您可以通过在编辑器的 World Outliner 中选择它们或通过查看不同的 `Character` 并再次按撇号 (') 来更改目标。当前检查的 `Character` 上方有最大的红色圆圈。

![Gameplay Debugger](https://github.com/tranek/GASDocumentation/raw/master/Images/gameplaydebugger.png)

**[⬆ 返回顶部](#table-of-contents)**

<a name="debugging-log"></a>
### 6.3 GAS 日志记录
GAS 源代码包含许多在不同详细级别生成的日志语句。您很可能会将这些视为 `ABILITY_LOG()` 语句。默认详细级别是 `Display`。默认情况下，任何更高的级别都不会在控制台中显示。

要更改日志类别的详细级别，请在控制台中输入：

```
log [category] [verbosity]
```

例如，要打开 `ABILITY_LOG()` 语句，您会在控制台中输入：
```
log LogAbilitySystem VeryVerbose
```

要将其重置为默认值，请输入：
```
log LogAbilitySystem Display
```

要显示所有日志类别，请输入：
```
log list
```

值得注意的 GAS 相关日志类别：

| 日志类别          | 默认详细级别 |
| ------------------------- | ----------------------- |
| LogAbilitySystem          | Display                 |
| LogAbilitySystemComponent | Log                     |
| LogGameplayCueDetails     | Log                     |
| LogGameplayCueTranslator  | Display                 |
| LogGameplayEffectDetails  | Log                     |
| LogGameplayEffects        | Display                 |
| LogGameplayTags           | Log                     |
| LogGameplayTasks          | Log                     |
| VLogAbilitySystem         | Display                 |

有关更多信息，请参阅[关于日志记录的 Wiki](https://unrealcommunity.wiki/logging-lgpidy6i)。

**[⬆ 返回顶部](#table-of-contents)**

<a name="optimizations"></a>
## 7. 优化

<a name="optimizations-abilitybatching"></a>
### 7.1 能力批处理
在单个帧内激活、可选地向服务器发送 `TargetData` 并结束的 [`GameplayAbilities`](#concepts-ga) 可以[批处理以将两到三个 RPC 压缩为一个 RPC](#concepts-ga-batching)。这些类型的能力通常用于即时命中枪械。

<a name="optimizations-gameplaycuebatching"></a>
### 7.2 Gameplay Cue 批处理
如果您同时发送许多 [`GameplayCues`](#concepts-gc)，请考虑[将它们批处理为一个 RPC](#concepts-gc-batching)。目标是减少 RPC 的数量（`GameplayCues` 是不可靠的 NetMulticast）并发送尽可能少的数据。

<a name="optimizations-ascreplicationmode"></a>
### 7.3 AbilitySystemComponent 网络同步模式
默认情况下，[`ASC`](#concepts-asc) 处于[`完全复制模式`](#concepts-asc-rm)。这将把所有 [`GameplayEffects`](#concepts-ge) 复制到每个客户端（这对于单人游戏来说很好）。在多人游戏中，将玩家拥有的 `ASCs` 设置为 `混合复制模式`，将 AI 控制的角色设置为 `最小复制模式`。这将使应用于玩家角色的 `GEs` 仅复制给该角色的所有者，而应用于 AI 控制角色的 `GEs` 将永远不会向客户端复制 `GEs`。[`GameplayTags`](#concepts-gt) 仍将复制，[`GameplayCues`](#concepts-gc) 仍将是向所有客户端的不可靠 NetMulticast，无论 `复制模式` 如何。这将减少当所有客户端不需要查看它们时从 `GEs` 复制的网络数据。

<a name="optimizations-attributeproxyreplication"></a>
### 7.4 属性代理复制
在拥有许多玩家的大型游戏（如 Fortnite Battle Royale (FNBR)）中，将有许多 `ASCs` 位于始终相关的 `PlayerStates` 上，复制许多 [`Attributes`](#concepts-a)。为了优化这个瓶颈，Fortnite 在 `PlayerState::ReplicateSubobjects()` 中的**模拟玩家控制的代理**上完全禁用 `ASC` 及其 [`AttributeSets`](#concepts-as) 的复制。自主代理和 AI 控制的 `Pawns` 仍然根据其[`复制模式`](#concepts-asc-rm) 完全复制。除了复制始终相关的 `PlayerStates` 上的 `ASC` 的 `Attributes` 之外，FNBR 在玩家的 `Pawn` 上使用复制的代理结构。当服务器上的 `ASC` 的 `Attributes` 更改时，代理结构也会更改。客户端从代理结构接收复制的 `Attributes` 并将更改推回其本地 `ASC`。这允许 `Attribute` 复制使用 `Pawn` 的相关性和 `NetUpdateFrequency`。此代理结构还在位掩码中复制一小部分白名单的 `GameplayTags`。此优化减少了网络上的数据量，并允许我们利用 pawn 相关性。AI 控制的 `Pawns` 的 `ASC` 位于 `Pawn` 上，这已经使用了其相关性，因此它们不需要此优化。

> 我不确定自从那时完成的其他服务器端优化（复制图等）之后是否仍然需要它，并且这不是最可维护的模式。

*Epic 的 Dave Ratti 对[社区问题 #3](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)的回答*

<a name="optimizations-asclazyloading"></a>
### 7.5 ASC 延迟加载
Fortnite Battle Royale (FNBR) 在世界上有许多可伤害的 `AActors`（树木、建筑物等），每个都有一个 [`ASC`](#concepts-asc)。这可能会增加内存成本。FNBR 通过仅在需要时（当它们首次受到玩家伤害时）延迟加载 `ASCs` 来优化这一点。这减少了总体内存使用，因为某些 `AActors` 在比赛中可能永远不会受到伤害。

**[⬆ 返回顶部](#table-of-contents)**

<a name="qol"></a>
## 8. 生活质量建议

<a name="qol-gameplayeffectcontainers"></a>
### 8.1 Gameplay Effect 容器
[GameplayEffectContainers](#concepts-ge-containers) 将 [`GameplayEffectSpecs`](#concepts-ge-spec)、[`TargetData`](#concepts-targeting-data)、[简单瞄准](#concepts-targeting-containers) 和相关功能组合成易于使用的结构。这些非常适合将 `GameplayEffectSpecs` 传输到从技能生成的投弹物，投弹物随后将在稍后的碰撞时应用它们。

<a name="qol-asynctasksascdelegates"></a>
### 8.2 用于绑定到 ASC 委托的蓝图 AsyncTasks
为了提高设计师友好的迭代时间，特别是在设计 UI 的 UMG Widgets 时，创建蓝图 AsyncTasks（在 C++ 中）以直接从您的 UMG 蓝图图中绑定到 `ASC` 上的常见更改委托。唯一需要注意的是它们必须手动销毁（例如当 Widget 被销毁时），否则它们将永远存在于内存中。示例项目包括三个蓝图 AsyncTasks。

监听 `Attribute` 更改：

![监听属性更改 BP 节点](https://github.com/tranek/GASDocumentation/raw/master/Images/attributeschange.png)

监听冷却更改：

![监听冷却更改 BP 节点](https://github.com/tranek/GASDocumentation/raw/master/Images/cooldownchange.png)

监听 `GE` 堆叠更改：

![监听 GameplayEffect 堆叠更改 BP 节点](https://github.com/tranek/GASDocumentation/raw/master/Images/gestackchange.png)

**[⬆ 返回顶部](#table-of-contents)**

<a name="troubleshooting"></a>
## 9. 故障排除

<a name="troubleshooting-notlocal"></a>
### 9.1 `LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`
您需要[在客户端上初始化 `ASC`](#concepts-asc-setup)。

**[⬆ 返回顶部](#table-of-contents)**

<a name="troubleshooting-scriptstructcache"></a>
### 9.2 `ScriptStructCache` 错误
您需要调用 [`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata)。

**[⬆ 返回顶部](#table-of-contents)**

<a name="troubleshooting-replicatinganimmontages"></a>
### 9.3 动画蒙太奇未复制到客户端
确保您在 [GameplayAbilities](#concepts-ga) 中使用 `PlayMontageAndWait` 蓝图节点而不是 `PlayMontage`。此 [AbilityTask](#concepts-at) 会自动通过 `ASC` 复制蒙太奇，而 `PlayMontage` 节点不会。

**[⬆ 返回顶部](#table-of-contents)**

<a name="troubleshooting-duplicatingblueprintactors"></a>
### 9.4 复制蓝图 Actor 将 AttributeSets 设置为 nullptr
[Unreal Engine 中有一个错误](https://issues.unrealengine.com/issue/UE-81109)，将从现有蓝图 Actor 类复制的蓝图 Actor 类上的 `AttributeSet` 指针设置为 nullptr。有一些解决方法。我成功的方法是不在我的类上创建专用的 `AttributeSet` 指针（.h 中没有指针，不在构造函数中调用 `CreateDefaultSubobject`），而是在 `PostInitializeComponents()` 中直接将 `AttributeSets` 添加到 `ASC`（示例项目中未显示）。复制的 `AttributeSets` 仍将位于 `ASC's` `SpawnedAttributes` 数组中。它看起来像这样：

```c++
void AGDPlayerState::PostInitializeComponents()
{
	Super::PostInitializeComponents();

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->AddSet<UGDAttributeSetBase>();
		// ... 您可能拥有的任何其他 AttributeSets
	}
}
```

在这种情况下，您将使用 `ASC` 上的函数而不是[调用从宏创建的 `AttributeSet` 上的函数](#concepts-as-attributes)来读取和设置 `AttributeSet` 中的值。

```c++
/** 返回属性的当前（最终）值 */
float GetNumericAttribute(const FGameplayAttribute &Attribute) const;

/** 设置属性的基础值。现有的活动修饰符不会被清除，将作用于新的基础值。 */
void SetNumericAttributeBase(const FGameplayAttribute &Attribute, float NewBaseValue);
```

因此 `GetHealth()` 看起来像：

```c++
float AGDPlayerState::GetHealth() const
{
	if (AbilitySystemComponent)
	{
		return AbilitySystemComponent->GetNumericAttribute(UGDAttributeSetBase::GetHealthAttribute());
	}

	return 0.0f;
}
```

设置（初始化）生命值 `Attribute` 看起来像：

```c++
const float NewHealth = 100.0f;
if (AbilitySystemComponent)
{
	AbilitySystemComponent->SetNumericAttributeBase(UGDAttributeSetBase::GetHealthAttribute(), NewHealth);
}
```

提醒一下，`ASC` 期望每个 `AttributeSet` 类最多只有一个 `AttributeSet` 对象。

**[⬆ 返回顶部](#table-of-contents)**

<a name="troubleshooting-unresolvedexternalsymbolmarkpropertydirty"></a>
### 9.5 未解析的外部符号 UEPushModelPrivate::MarkPropertyDirty(int,int)

如果您收到编译器错误，如：

```
error LNK2019: unresolved external symbol "__declspec(dllimport) void __cdecl UEPushModelPrivate::MarkPropertyDirty(int,int)" (__imp_?MarkPropertyDirty@UEPushModelPrivate@@YAXHH@Z) referenced in function "public: void __cdecl FFastArraySerializer::IncrementArrayReplicationKey(void)" (?IncrementArrayReplicationKey@FFastArraySerializer@@QEAAXXZ)
```

这是由于尝试在 `FFastArraySerializer` 上调用 `MarkItemDirty()`。我在更新 `ActiveGameplayEffect` 时遇到过这种情况，例如更新冷却持续时间时。

```c++
ActiveGameplayEffects.MarkItemDirty(*AGE);
```

发生的情况是 `WITH_PUSH_MODEL` 在多个地方被定义。`PushModelMacros.h` 将其定义为 0，而在多个地方定义为 1。`PushModel.h` 将其视为 1，但 `PushModel.cpp` 将其视为 0。

解决方案是在 `Build.cs` 中将 `NetCore` 添加到项目的 `PublicDependencyModuleNames`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="troubleshooting-enumnamesarenowpathnames"></a>
### 9.6 枚举名称现在由路径名表示

如果您收到编译器警告，如：

```
warning C4996: 'FGameplayAbilityInputBinds::FGameplayAbilityInputBinds': Enum names are now represented by path names. Please use a version of FGameplayAbilityInputBinds constructor that accepts FTopLevelAssetPath. Please update your code to the new API before upgrading to the next release, otherwise your project will no longer compile.
```

UE 5.1 弃用了在 `BindAbilityActivationToInputComponent()` 的构造函数中使用 `FString`。相反，我们必须传入 `FTopLevelAssetPath`。

旧的、已弃用的方式：
```c++
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), FString("EGDAbilityInputID"), static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

新方式：
```c++
FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));
AbilitySystemComponent->BindAbilityActivationToInputComponent(InputComponent, FGameplayAbilityInputBinds(FString("ConfirmTarget"),
	FString("CancelTarget"), AbilityEnumAssetPath, static_cast<int32>(EGDAbilityInputID::Confirm), static_cast<int32>(EGDAbilityInputID::Cancel)));
```

有关更多信息，请参阅 `Engine\Source\Runtime\CoreUObject\Public\UObject\TopLevelAssetPath.h`。

**[⬆ 返回顶部](#table-of-contents)**

<a name="acronyms"></a>
## 10. 常用 GAS 缩写

| 名称                                                                                                   | 缩写            |
|------------------------------------------------------------------------------------------------------- | ------------------- |
| AbilitySystemComponent                                                                                 | ASC                 |
| AbilityTask                                                                                            | AT                  |
| [Epic 的 Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg) | ARPG, ARPG Sample   |
| CharacterMovementComponent                                                                             | CMC                 |
| GameplayAbility                                                                                        | GA                  |
| GameplayAbilitySystem                                                                                  | GAS                 |
| GameplayCue                                                                                            | GC                  |
| GameplayEffect                                                                                         | GE                  |
| GameplayEffectExecutionCalculation                                                                     | ExecCalc, Execution |
| GameplayTag                                                                                            | Tag, GT             |
| ModifierMagnitudeCalculation                                                                           | ModMagCalc, MMC     |

**[⬆ 返回顶部](#table-of-contents)**

<a name="resources"></a>
## 11. 其他资源
* [官方文档](https://docs.unrealengine.com/en-US/Gameplay/GameplayAbilitySystem/index.html)
* 源代码！
   * 特别是 `GameplayPrediction.h`
* [Epic 的 Lyra 示例项目](https://unrealengine.com/marketplace/en-US/learn/lyra)
* [Epic 的 Action RPG 示例项目](https://www.unrealengine.com/marketplace/en-US/product/action-rpg)
* [Unreal Slackers Discord](https://unrealslackers.org/) 有一个专门致力于 GAS 的文本频道 `#gameplay-ability-system`
   * 查看置顶消息
* [Dan 'Pan' 的 GitHub 资源仓库](https://github.com/Pantong51/GASContent)
* [SabreDartStudios 的 YouTube 视频](https://www.youtube.com/channel/UCCFUhQ6xQyjXDZ_d6X_H_-A)

<a name="resources-daveratti"></a>
### 11.1 与 Epic Game 的 Dave Ratti 的问答

<a name="resources-daveratti-community1"></a>
#### 11.1.1 社区问题 1
[Dave Ratti 对 Unreal Slackers Discord 服务器社区关于 GAS 问题的回应](https://epicgames.ent.box.com/s/m1egifkxv3he3u3xezb9hzbgroxyhx89)：

1. 我们如何在 `GameplayAbilities` 之外或独立于它按需创建作用域预测窗口？例如，即时抛射物如何在击中敌人时本地预测伤害 `GameplayEffect`？

> PredictionKey 系统并不是真的用于做这件事。从根本上说，这个系统的工作方式是客户端启动预测性操作，使用密钥告诉服务器，然后客户端和服务器运行相同的操作，并将预测性副作用与给定的预测密钥关联起来。例如，"我正在预测性地激活能力"或"我已经生成了目标数据，并将在 WaitTargetData 任务之后预测性地运行能力图的一部分"。
>
> 使用这种模式，PredictionKey 会从服务器"反弹"，并通过 UAbilitySystemComponent::ReplicatedPredictionKeyMap（复制属性）返回到客户端。一旦密钥从服务器复制回来，客户端就能够撤销所有本地预测性副作用（GameplayCues、GameplayEffects）：复制的版本*将存在*，如果不存在，则是预测错误。确切地知道何时撤销预测性副作用在这里至关重要：如果太早，会看到空白；如果太晚，会有"双重"。（注意，这指的是有状态预测，例如循环 GameplayCue 或基于持续时间的 Gameplay Effect。"突发" GameplayCues 和即时 Gameplay Effects 永远不会被"撤销"或回滚。如果与它们关联了预测密钥，它们只是在客户端上被跳过。）
>
> 进一步强调这一点：预测性操作必须是服务器不会自行执行，而只有在客户端告诉它时才执行的操作。因此，拥有一个通用的"按需创建密钥并告诉服务器，以便我可以运行某些内容"是行不通的，除非"某些内容"是服务器只有在客户端告诉它时才会执行的操作。
>
> 回到原始问题：像即时抛射物之类的东西。Paragon 和 Fortnite 都有使用 GameplayCues 的抛射物 actor 类。但是，我们不使用 Prediction Key 系统来执行此操作。相反，我们有一个非复制 GameplayCues 的概念。GameplayCues 只是本地触发，并被服务器完全跳过。基本上，所有这些都是对 UGameplayCueManager::HandleGameplayCue 的直接调用。它们不通过 UAbilitySystemComponent 路由，因此不进行预测密钥检查/提前返回。
>
> 非复制 GameplayCues 的缺点是，它们不会被复制。因此，抛射物类/蓝图需要确保调用这些函数的代码路径在每个人身上运行。我们有启动提示（在 BeginPlay 中调用）、爆炸、撞墙/角色等。
>
> 这些类型的事件已经在客户端生成，所以调用非复制 gameplay cue 没什么大不了的。复杂的蓝图可能很棘手，并且需要作者确保理解它们在哪里运行。

2. 当使用带有 `OnlyServerWait` 的 `WaitNetSync` `AbilityTask` 在本地预测的 `GameplayAbility` 中创建作用域预测窗口时，玩家是否可以通过延迟向服务器发送数据包来控制 `GameplayAbility` 时机从而作弊，因为服务器正在等待他们的 RPC 及其预测密钥？这在 Paragon 或 Fortnite 中曾经是一个问题吗？如果是，Epic 做了什么来解决这个问题？

> 是的，这是一个合理的担忧。在服务器上运行、等待客户端"信号"的任何能力蓝图都可能容易受到 lag switch 类型利用的影响。
>
> Paragon 有一个类似于 UAbilityTask_WaitTargetData 的自定义定位任务。在这个任务中，我们有超时，或者将在客户端等待即时定位模式的"最大延迟"。如果定位模式正在等待用户确认（按钮按下），那么它将被忽略，因为用户允许花时间。但是对于即时确认定位的能力，我们只会等待一定时间，然后 A) 在服务器端生成目标数据或 B) 取消能力。
>
> 我们从未为 WaitNetSync 提供过这样的机制，我们很少使用它。
>
> 我不相信 Fortnite 使用任何类似的东西。Fortnite 中的武器能力是特殊情况批处理为单个 Fortnite 特定的 RPC：一个 RPC 来激活能力，提供目标数据，并结束能力。因此，武器能力在大逃杀中本质上不容易受到此影响。
>
> 我的看法是，这可能是系统范围内可以解决的问题，但我不会在不久的将来看到我们自己做出改变。为提到的情况将 WaitNetSync 修复为包括最大延迟可能是一个合理的任务，但同样 - 我们不会在不久的将来在我们的最终端执行此操作。


3. Paragon 和 Fortnite 使用了哪种 `EGameplayEffectReplicationMode`，Epic 对何时使用每种有什么建议？

> 两款游戏基本上都为其玩家控制的角色使用 Mixed 模式，为 AI 控制（AI 小兵、丛林野怪、AI Husks 等）使用 Minimal。这就是我建议大多数在多人游戏中使用该系统的人。你应该在项目的早期阶段设置这些，越早越好。
>
> Fortnite 在优化方面更进一步。它实际上根本不为模拟代理复制 UAbilitySystemComponent。组件和属性子对象在拥有的 fortnite player state 类的 ::ReplicateSubobjects() 中被跳过。我们确实将能力系统组件的最低限度的复制数据推送到 pawn 本身的结构中（基本上，属性值的子集和我们在位掩码中复制下来的标签的白名单子集）。我们称之为"代理"。在接收端，我们获取在 pawn 上复制的代理数据，并将其推回 player state 上的能力系统组件。所以在 FNBR 中每个玩家确实有一个 ASC，它只是不直接复制：相反，它通过 pawn 上的最小代理结构复制数据，然后在接收端路由回 ASC。这是一个优势，因为它是 A) 更小的数据集 B) 利用 pawn 相关性。
>
> 我不确定自从那时完成的其他服务器端优化（Replication Graph 等）是否仍然需要它，它不是最可维护的模式。


4. 由于根据 `GameplayPrediction.h`，我们无法预测 `GameplayEffects` 的移除，有什么策略可以减轻延迟对移除 `GameplayEffects` 的影响吗？例如，当移除移动速度减速时，目前我们必须等待服务器复制 `GameplayEffect` 移除，导致玩家角色位置瞬间移动。

> 这是一个棘手的问题，我没有好的答案。我们通常通过容差和平滑来回避这些问题。我完全同意能力系统与角色移动系统的精确同步并不理想，这是我们确实想要修复的。
>
> 我曾有过允许预测性移除 GE 的想法，但从未能在必须继续前进之前解决所有边缘情况。但这并不能解决所有问题，因为角色移动仍然有一个内部保存的移动缓冲区，它不知道能力系统和可能的移动速度修饰符等。即使无法预测 GE 的移除，仍有可能进入更正反馈循环。
>
> 如果你认为你有一个真正绝望的情况，你可以预测性地添加一个 GE 来抑制你的移动速度 GE。我自己从未做过，但以前对此进行过理论化。它可能有助于解决某类问题。


5. 我们知道在 Paragon 和 Fortnite 中，`AbilitySystemComponent` 位于 `PlayerState` 中，而在 Action RPG 示例中位于 `Character` 中。Epic 关于 AbilitySystemComponent 应该位于何处的内部规则、指南或建议是什么，它的 `Owner` 应该是什么？

> 总的来说，我会说任何不需要重生的人都应该让 Owner 和 Avatar actor 相同。任何像 AI 敌人、建筑物、世界道具等。
>
> 任何需要重生的人都应该让 Owner 和 Avatar 不同，这样 Ability System Component 就不需要在重生后保存/重新创建/恢复。PlayerState 是合乎逻辑的选择，它复制到所有客户端（而 PlayerController 不）。缺点是 PlayerStates 总是相关的，所以你可能会在 100 人游戏中遇到问题（参见 Fortnite 在问题 #3 中所做的事情）。


6. 拥有多个具有相同所有者但不同 Avatar（例如，在 pawn 和武器/物品/抛射物上，`Owner` 设置为 `PlayerState`）的 `AbilitySystemComponents` 是否可行？

> 我在那里看到的第一个问题将是在拥有 actor 上实现 IGameplayTagAssetInterface 和 IAbilitySystemInterface。前者可能是可能的：只是聚合所有 ASC 的标签（但要注意 - HasAllMatchingGameplayTags 可能仅通过跨 ASC 聚合来满足。仅仅将调用转发到每个 ASC 并对结果进行 OR 是不够的）。但后者甚至更棘手：哪个 ASC 是权威的？如果有人想应用 GE - 应该接收哪个？也许你可以解决这些问题，但问题的一面将是最困难的：拥有多个 ASC 的所有者。
>
> pawn 和武器上的独立 ASC 本身可能是有意义的。例如，区分描述武器的标签与描述拥有 pawn 的标签。也许授予武器的标签也"应用"于所有者而不应用其他任何东西（例如，属性和 GE 是独立的，但所有者将像我上面描述的那样聚合拥有的标签）。这可能行得通，我确信。但是，拥有相同所有者的多个 ASC 可能会变得冒险。


7. 有没有办法阻止服务器覆盖拥有客户端上本地预测能力的冷却持续时间？在高延迟情况下，这将允许拥有客户端在其本地冷却到期时"尝试"再次激活能力，但在服务器上仍处于冷却状态。当拥有客户端的激活请求通过网络到达服务器时，服务器可能已关闭冷却或服务器能够将激活请求排队到剩余的毫秒数。否则，延迟较高的客户端与延迟较低的客户端相比，在重新激活能力之前有更长的延迟。这对于非常低冷却的能力（如可能少于一秒冷却的基本攻击）最为明显。如果没有办法阻止服务器覆盖本地预测能力的冷却持续时间，Epic 对减轻高延迟对重新激活能力的影响的策略是什么？用另一种基于示例的方式来说，Epic 如何设计 Paragon 的基本攻击和其他能力，以便高延迟玩家可以通过本地预测与低延迟玩家以相同的速度攻击或激活？

> 简短的回答是没有办法阻止这种情况，Paragon 肯定有这个问题。延迟较高的连接的基本攻击 ROF（射击速率）较低。
>
> 我试图通过添加"GE 协调"来修复这个问题，其中在计算 GE 持续时间时会考虑延迟。基本上允许服务器消耗一部分总 GE 时间，以便 GE 客户端侧的有效时间与任何延迟量 100% 一致（尽管波动仍可能导致问题）。然而，我从未使其处于可以发布的状态，项目进展很快，我们从未完全解决它。
>
> Fortnite 对武器射击速率进行自己的记录：它不使用 GE 作为武器上的冷却。如果这对你的游戏是一个关键问题，我建议这样做。


8. Epic 对 GameplayAbilitySystem 插件的路线图是什么？Epic 计划在 2019 年及以后添加哪些功能？

> 我们认为该系统现在总体上相当稳定，我们没有人在开发主要的新功能。偶尔会为 Fortnite 或来自 UDN/拉取请求进行错误修复和小改进，但现在就是这样。
>
> 长期来看，我认为我们最终会做"V2"或一些重大更改。我们在编写这个系统时学到了很多，觉得我们做对了很多，也做错了很多。我希望有机会纠正这些错误并改进上面指出的一些致命缺陷。
>
> 如果 V2 到来，提供升级路径将是至关重要的。我们永远不会制作 V2 并让 Fortnite 永远停留在 V1 上：会有一些路径或程序可以自动迁移尽可能多的内容，尽管肯定仍需要一些手动重新制作。
>
> 高优先级修复将是：
> * 与角色移动系统更好地互操作。统一客户端预测。
> * GE 移除预测（问题 #4）
> * GE 延迟协调（问题 #7）
> * 通用网络优化，例如批处理 RPC 和代理结构。主要是我们为 Fortnite 所做的事情，但找到将其分解为更通用形式的方法，至少使游戏可以更容易地编写自己的特定于游戏的优化。
>
> 我会考虑进行更一般的重构类型更改：
> * 我想从根本上摆脱 GEs 直接引用电子表格值，而是它们能够发出参数，这些参数可以由绑定到电子表格值的更高级别对象填充。当前模型的问题在于，由于与曲线表行的紧密耦合，GE 变得不可共享。我认为可以编写一个通用的参数化系统，并成为 V2 系统的基础。
> * 减少 UGameplayAbility 上的"策略"数量。我会删除 ReplicationPolicy 和 InstancingPolicy。复制，imho，几乎从不真正需要，并且会引起混淆。InstancingPolicy 应该被替换，而是使 FGameplayAbilitySpec 成为可以子类化的 UObject。这应该是"非实例化能力对象"，它具有事件并且是可蓝图化的。UGameplayAbility 应该是"每次执行实例化"的对象。如果需要实际实例化，它可以是可选的：相反，"非实例化"能力将通过新的 UGameplayAbilitySpec 对象实现。
> * 系统应该提供更多的"中级"构造，例如"过滤的 GE 应用容器"（数据驱动将哪些 GE 应用于哪些 actor，具有更高级别的游戏逻辑）、"重叠体积支持"（基于碰撞体重叠事件应用"过滤的 GE 应用容器"）等。这些是每个项目最终以自己的方式实现的构建块。正确实现它们并非易事，所以我认为我们应该做得更好，提供一些基本实现。
> * 一般来说，减少启动项目所需的样板。可能是一个单独的模块"Ex library"或其他，可以提供被动能力或基本命中扫描武器等功能。这个模块将是可选的，但会让你快速启动和运行。
> * 我想将 GameplayCues 移动到一个不与能力系统耦合的单独模块。我认为这里可以有很多改进。


> 这只是我个人的看法，而不是任何人的承诺。我认为最现实的行动方案是，随着新的引擎技术举措的到来，能力系统将需要更新，这将是这样做的时机。这些举措可能与脚本、网络或物理/角色移动有关。这也是非常长远的，所以我不能对时间表做出承诺或估计。


**[⬆ 返回顶部](#table-of-contents)**

<a name="resources-daveratti-community2"></a>
#### 11.1.2 社区问题 2
社区成员 [iniside](https://github.com/iniside) 与 Dave Ratti 的问答：

1. 是否计划支持解耦的固定 ticking？我想要游戏线程固定（如 30/60fps），让渲染线程自由运行。我问这是我们应该期待未来的事情还是不应该，以便对游戏玩法应该如何工作做出一些假设。我主要问是因为现在有一个固定的异步物理 tick，这提出了一个问题，系统的其余部分将来可能如何工作。我不隐瞒拥有固定 tick 游戏线程而不固定引擎其余部分的 tick 率将是超凡的。

> 没有计划解耦渲染帧速率和游戏线程 tick 帧速率。我认为由于这些系统的复杂性以及保持与以前引擎版本的向后兼容性的要求，这种情况永远不会发生。
>
> 相反，我们的方向是拥有一个异步的"物理线程"，它以固定的 tick 率运行，独立于游戏线程。需要以固定率运行的东西可以在那里运行，游戏线程/渲染可以像往常一样操作。
>
> 值得澄清的是，网络预测支持它所谓的独立 Ticking 和固定 Ticking 模式。我的长期计划是保持独立 Ticking 大致与今天的网络预测一样，它在游戏线程上以可变帧速率运行，没有"组/世界"预测，它只是经典的"客户端预测他们自己的 pawn 和拥有的 actor"模型。固定 Ticking 将是使用异步物理内容并允许你预测非客户端控制/拥有的 actor，如物理对象和其他客户端/pawn/车辆/等。


2. Network Prediction 与 Ability System 的集成计划如何？例如，固定帧能力激活（因此服务器获得激活能力和执行任务的帧，而不是预测密钥）？

> 是的，计划是重写/删除能力系统的预测密钥，并用网络预测构造替换它们。NetworkPredictionExtras 中的 MockAbility 示例显示了这可能如何工作，但它们比 GAS 所需要的更"硬编码"。
>
> 主要思想是，我们将删除 ASC 的 RPC 中显式的客户端->服务器预测密钥交换。将不再有预测窗口或作用域预测密钥。相反，一切都将围绕网络预测帧进行构建。重要的是客户端和服务器就事情何时发生达成一致。例如：
>
> * 能力何时被激活/结束/取消
> * GameplayEffects 何时被应用/移除
> * 属性值（属性在帧 X 时的值）
>
> 我认为这可以在能力系统级别通用完成。但实际上使 UGameplayAbility 内部的用户定义逻辑完全可回滚仍需要更多工作。我们最终可能会有一个 UGameplayAbility 的子类，它是完全可回滚的，并且可以访问更有限的功能集或仅标记为回滚友好的 Ability Tasks。类似的东西。动画事件和根运动以及它们的处理方式也有很多含义。
>
> 希望我能有一个更清晰的答案，但在再次触及 GAS 之前，让基础正确是非常重要的。移动和物理必须可靠，才能更改更高级别的系统。


3. 是否有计划将 Network Prediction 开发移向主分支？不撒谎，我真的很想查看最新代码。无论其状态如何。

> 我们正在努力实现这一目标。系统工作仍然全部在 NetworkPrediction 中完成（参见 NetworkPhysics.h），底层异步物理内容应该全部可用（RewindData.h 等）。但我们也有在 Fortnite 中的用例，我们一直专注于这些用例，显然不能公开。我们正在处理错误、性能优化等。
>
> 更多背景信息：在开发这个系统的早期版本时，我们非常关注"前端" - 如何定义和编写状态和模拟。我们在那里学到了很多。但随着异步物理内容的上线，我们更加专注于在系统中获得一些真实的东西，以牺牲抛弃一些早期的抽象为代价。目标是当真实的东西工作时循环回来并重新统一事物。例如，回到"前端"并在我们现在正在工作的核心技术之上制作最终版本。


4. 在主分支上有一段时间有一个用于发送 GameplayMessages 的插件（看起来像事件/消息总线），但它被删除了。有什么恢复它的计划吗？使用 Game Features/模块化游戏玩法插件，拥有通用事件总线调度程序将非常有用。

> 我认为你指的是 GameplayMessages 插件。这可能会在某个时候回来 - API 并没有真正最终确定，作者并不打算让它公开。我认为它应该对模块化游戏设计有用。但这并不是我真正关注的领域，所以我没有更多信息。


5. 我最近在玩异步固定物理，结果很有希望，尽管如果将来会有 NP 更新，我可能会只是玩弄并等待，因为为了让它工作，我仍然需要将整个引擎放入固定 tick，另一方面我尝试将物理保持在 33ms。如果一切都是 30 fps，这不会带来良好的体验（:。

我注意到有一些关于异步 CharacterMovementComponent 的工作，但不确定这是否将使用网络预测，或者它是一个单独的努力？

自从我注意到它以来，我还继续尝试以固定 tick 率实现自定义异步移动，虽然效果还不错，但除此之外，我还需要添加单独的插值更新。设置是在单独的工作线程上以固定 33ms 更新运行模拟 tick，进行计算，保存结果，并在游戏线程上对其进行插值以匹配当前帧速率。并不完美，但完成了工作。

我的问题是，这是否可能在未来更容易设置，因为只需要编写相当多的样板代码（插值部分），并且单独插值每个移动对象并不是特别高效。

异步内容真的很有趣，因为它允许你真正以固定的更新率运行游戏模拟（这将使固定线程不需要），并且结果更可预测。这是预期的前进方向，还是更多是为了选择系统的利益？据我记得，actor 转换不是异步更新的，蓝图并不完全是线程安全的。换句话说，这是计划在更多框架级别支持的东西，还是每个游戏都必须自己解决？

> 异步 CharacterMovementComponent
>
> 这基本上是将 CMC 原样移植到物理线程的早期原型/实验。我还不会将其视为 CMC 的未来，但它可能会演变成那样。现在没有网络支持，所以我不会真正关注它。执行此操作的人主要关心该系统将增加的输入延迟以及如何减轻这一点。
>
> 我仍然需要将整个引擎放入固定 tick，另一方面我尝试将物理保持在 33ms。如果一切都是 30 fps，这不会带来良好的体验（:。
>
> 异步内容真的很有趣，因为它允许你真正以固定的更新率运行游戏模拟（这将使固定线程不需要）
>
> 是的。这里的目标是，启用异步物理后，你可以让引擎以可变 tick 率运行，而物理和"核心"游戏模拟可以以固定率运行（例如角色移动、车辆、GAS 等）。
>
> 这些是现在需要设置的 cvar（我认为你已经想出来了）：
> `p.DefaultAsyncDt=0.03333`
> `p.RewindCaptureNumFrames=64`
>
> Chaos 确实为物理状态提供插值（例如，推回到 UPrimitiveComponent 并且对游戏代码可见的转换）。现在有一个 cvar `p.AsyncInterpolationMultiplier`，如果你想查看它，可以控制它。你应该看到物理物体的平滑连续运动，而无需编写任何额外的代码。
>
> 如果你想插值非物理状态，这仍然取决于你现在这样做。例如，你想要更新（tick）异步物理线程上的冷却但在游戏线程上看到平滑连续的插值，以便每个渲染帧都更新冷却可视化。我们最终会解决这个问题，但现在还没有示例。
>
> 只是需要编写相当多的样板代码，
>
> 是的，到目前为止，这一直是系统的一个大问题。我们希望提供一个接口，经验丰富的程序员可以使用它来最大化性能和安全性（能够编写"有效工作的"预测性游戏代码，而没有大量危险和你可以做但最好不要做的事情）。像 CharacterMovement 这样的事情可能会做很多自定义事情来最大化其性能 - 例如，编写模板化代码和进行批量更新，扩展，将更新循环分解为不同的阶段等。我们希望为这个用例提供良好的"低级"接口到异步线程和回滚系统。在这种情况下 - 角色移动系统本身仍然可以以自己的方式扩展。例如，提供一种以蓝图方式自定义移动模式的方法，并提供线程安全的蓝图 API。
>
> 但我们认识到，对于实际上并不真正需要自己的"系统"的更简单的游戏对象，这是不可接受的。更符合 Unreal 的是所必需的。例如，使用反射系统，具有通用的蓝图支持等。有在其他线程上使用 blueprints 的示例（参见 BlueprintThreadSafe 关键字和动画系统一直在努力的方向）。所以我认为有一天会有某种形式。但同样，我们还没有到那一步。
>
> 我意识到你只是问插值，但这只是通用的答案：现在我们让你手动完成所有事情，如 NetSerialize、ShouldReconcile、Interpolate 等，但最终我们将有一种方式，就像"如果你想只是使用反射系统，你不必手动编写这些东西"。我们不想*强迫*每个人使用反射系统，因为它施加了我们认为不想在系统最低级别承担的其他限制。
>
> 然后只是将其与我之前所说的联系起来 - 现在我们真的专注于让几个非常具体的示例工作和高效，然后我们将转回到前端并使事情友好使用和迭代，为其他人使用减少样板等。

**[⬆ 返回顶部](#table-of-contents)**


<a name="changelog"></a>
## 12. GAS 更新日志

这是从官方 Unreal Engine 升级更新日志编译的 GAS 值得注意的更改列表（修复、更改和新功能），以及我遇到的未记录的更改。如果你发现这里未列出的内容，请提出 issue 或拉取请求。

<a name="changelog-5.3"></a>
### 5.3
* 崩溃修复：修复了在无缝旅行后尝试应用 Gameplay Cues 时的崩溃。
* 崩溃修复：修复了使用 Live Coding 时由 GlobalAbilityTaskCount 引起的崩溃。
* 崩溃修复：修复了 UAbilityTask::OnDestroy，如果为 UAbilityTask_StartAbilityState 等情况递归调用，不会崩溃。
* 错误修复：现在可以在子类中安全地调用 `Super::ActivateAbility`。以前，它会调用 `CommitAbility`。
* 错误修复：添加了正确复制不同类型的 FGameplayEffectContext 的支持。
* 错误修复：FGameplayEffectContextHandle 现在在检索 "Actors" 之前检查数据是否有效。
* 错误修复：为 Gameplay Ability System 目标数据位置信息保留旋转。
* 错误修复：Gameplay Ability System 现在仅在找到有效的 PC 时才停止搜索 PC。
* 错误修复：如果存在，则在 RemoveGameplayCue_Internal 中使用现有的 GameplayCueParameters 而不是默认参数对象。
* 错误修复：GameplayAbilityWorldReticle 现在面向源 Actor 而不是 TargetingActor。
* 错误修复：如果通过 GiveAbilityAndActivateOnce 传递触发事件数据并且能力列表被锁定，则缓存该事件。
* 错误修复：添加了对 FInheritedGameplayTags 的支持，以立即更新其 CombinedTags 而不是等到保存。
* 错误修复：将 ShouldAbilityRespondToEvent 从仅客户端代码路径移动到服务器和客户端。
* 错误修复：修复了由于曲线简化导致 FAttributeSetInitterDiscreteLevels 在 Cooked Builds 中不起作用的问题。
* 错误修复：在 GameplayAbility 中设置 CurrentEventData。
* 错误修复：确保在可能执行回调之前正确设置 MinimalReplicationTags。
* 错误修复：修复了 ShouldAbilityRespondToEvent 未在实例化的 GameplayAbility 上调用的问题。
* 错误修复：在 Child Actors 上执行的 Gameplay Cue Notify Actors 在禁用 gc.PendingKill 时不再泄漏内存。
* 错误修复：修复了 GameplayCueManager 中的一个问题，其中 GameplayCueNotify_Actors 可能由于哈希冲突而"丢失"。
* 错误修复：即使 Actor 上没有 Gameplay Tags，WaitGameplayTagQuery 现在也会尊重其查询。
* 错误修复：PostAttributeChange 和 AttributeValueChangeDelegates 现在将具有正确的 OldValue。
* 错误修复：修复了如果结构由本机代码创建，FGameplayTagQuery 不显示正确查询描述的问题。
* 错误修复：确保如果使用能力系统，则调用 UAbilitySystemGlobals::InitGlobalData。以前，如果用户不调用它，Gameplay Ability System 将无法正常工作。
* 错误修复：修复了从 UGameplayAbility::EndAbility 链接/取消链接动画层时的问题。
* 错误修复：更新了 Ability System Component 函数以在使用前检查 Spec 的能力指针。
* 新增：向 FGameplayTagRequirements 添加了 GameplayTagQuery 字段，以启用更复杂的要求规范。
* 新增：引入了 FGameplayEffectQuery::SourceAggregateTagQuery 来增强 SourceTagQuery。
* 新增：扩展了从控制台命令执行和取消 Gameplay Abilities 和 GameplayEffects 的功能。
* 新增：添加了在 Gameplay Ability 蓝图上执行"审计"的功能，该功能将显示有关它们如何开发和预期使用的信息。
* 更改：对于实例化每个 Actor 的 Gameplay Abilities，OnAvatarSet 现在在主实例上调用，而不是在 CDO 上。
* 更改：允许在同一 Gameplay Ability 图中同时使用 Activate Ability 和 Activate Ability From Event。
* 更改：AnimTask_PlayMontageAndWait 现在有一个切换功能，允许在 BlendOut 事件之后完成和中断。
* 更改：ModMagnitudeCalc 包装函数已声明为 const。
* 更改：FGameplayTagQuery::Matches 现在对空查询返回 false。
* 更改：更新了 FGameplayAttribute::PostSerialize 以将包含的属性标记为可搜索名称。
* 更改：更新了 GetAbilitySystemComponent 以将默认参数设置为 Self。
* 更改：将 AbilityTask_WaitTargetData 中的函数标记为 virtual。
* 更改：删除了未使用的函数 FGameplayAbilityTargetData::AddTargetDataToGameplayCueParameters。
* 更改：删除了剩余的 GameplayAbility::SetMovementSyncPoint。
* 更改：从 Gameplay 任务和能力系统组件中删除了未使用的复制标志。
* 更改：将一些 gameplay effect 功能移动到可选组件中。所有现有内容将在 PostCDOCompiled 期间自动更新以使用组件（如有必要）。

https://docs.unrealengine.com/5.3/en-US/unreal-engine-5.3-release-notes/

<a name="changelog-5.2"></a>
### 5.2
* 错误修复：修复了 `UAbilitySystemBlueprintLibrary::MakeSpecHandle` 函数中的崩溃。
* 错误修复：修复了 Gameplay Ability System 中的逻辑，其中非控制的 Pawn 将被视为远程，即使它是在服务器上本地生成的（例如车辆）。
* 错误修复：在被服务器拒绝的预测实例化能力上正确设置激活信息。
* 错误修复：修复了导致 GameplayCues 卡在远程实例上的错误。
* 错误修复：修复了链接调用 WaitGameplayEvent 时的内存踩踏。
* 错误修复：在 Blueprint 中调用 AbilitySystemComponent `GetOwnedGameplayTags()` 函数不再在多次执行同一节点时保留先前调用的返回值。
* 错误修复：修复了 GameplayEffectContext 复制永远不会复制的动态对象引用的问题。
  * 这阻止了 GameplayEffect 调用 `Owner->HandleDeferredGameplayCues(this)`，因为 `bHasMoreUnmappedReferences` 将始终为 true。
* 新增：[Gameplay Targeting System](https://docs.unrealengine.com/en-US/gameplay-targeting-system-in-unreal-engine/) 是一种创建数据驱动定位请求的方法。
* 新增：为 GameplayTag 查询添加了自定义序列化支持。
* 新增：添加了对复制派生的 FGameplayEffectContext 类型的支持。
* 新增：资产中的 Gameplay Attributes 现在在保存时注册为可搜索名称，允许在引用查看器中查看对属性的引用。
* 新增：为 AbilitySystemComponent 添加了一些基本单元测试。
* 新增：Gameplay Ability System 属性现在尊重 Core Redirects。这意味着现在可以在代码中重命名属性集及其属性，并通过向 DefaultEngine.ini 添加重定向条目，让它们在使用旧名称保存的资产中正确加载。
* 更改：允许从代码更改 Gameplay Effect 修饰符的评估通道。
* 更改：从 Gameplay Abilities 插件中删除了先前未使用的变量 `FGameplayModifierInfo::Magnitude`。
* 更改：删除了能力系统组件和 Smart Object 实例标签之间的同步逻辑。

https://docs.unrealengine.com/5.2/en-US/unreal-engine-5.2-release-notes/

<a name="changelog-5.1"></a>
### 5.1
* 错误修复：修复了复制的松散 gameplay 标志未复制到所有者的问题。
* 错误修复：修复了 AbilityTask 错误，该错误导致能力被及时垃圾回收阻止。
* 错误修复：修复了基于标签监听激活的 gameplay ability 无法激活的问题。如果列表中有多个 Gameplay Ability 在监听此标签，并且列表中的第一个无效或没有权限激活，就会发生这种情况。
* 错误修复：修复了使用数据注册表的 GameplayEffects 在加载时正确警告并改进了警告文本。
* 错误修复：从 UGameplayAbility 中删除了代码，该错误地仅向 Blueprint 调试器注册最后一个实例化能力以进行断点。
* 错误修复：修复了在 ApplyGameplayEffectSpecToTarget 内部的锁期间调用 EndAbility 时 Gameplay Ability Ability 卡住的问题。
* 新增：添加了对 Gameplay Effects 添加被阻止的能力标签的支持。
* 新增：添加了 WaitGameplayTagQuery 节点。一个基于 UAbilityTask，另一个基于 UAbilityAsync。此节点指定一个 TagQuery，并将根据配置在查询变为 true 或 false 时触发其输出引脚。
* 新增：修改了控制台变量中的 AbilityTask 调试，以在非发布版本中默认启用调试记录和打印到日志（根据需要能够热修复开/关）。
* 新增：现在可以将 AbilitySystem.AbilityTask.Debug.RecordingEnabled 设置为 0 以禁用，1 以在非发布版本中启用，2 以在所有版本（包括发布版本）中启用。
* 新增：可以使用 AbilitySystem.AbilityTask.Debug.AbilityTaskDebugPrintTopNResults 仅在日志中打印前 N 个结果（以避免日志垃圾邮件）。
* 新增：STAT_AbilityTaskDebugRecording 可用于测试这些默认启用的调试更改的性能影响。
* 新增：添加了调试命令来过滤 GameplayCue 事件。
* 新增：向 Gameplay Ability System 添加了新的调试命令AbilitySystem.DebugAbilityTags、AbilitySystem.DebugBlockedTags 和 AbilitySystem.DebugAttribute。
* 新增：添加了蓝图函数来获取 Gameplay 属性的调试字符串表示。
* 新增：添加了新的 Gameplay 任务资源重叠策略来取消现有任务。
* 更改：现在 Ability Tasks 应该确保在对 Ability 指针执行任何所需操作之后调用 Super::OnDestroy，因为在调用之后它将被清空。
* 更改：将 FGameplayAbilitySpec/Def::SourceObject 转换为弱引用。
* 更改：将 Ability Task 中的 Ability System Component 引用设为弱指针，以便垃圾回收可以删除它。
* 更改：删除了冗余枚举 EWaitGameplayTagQueryAsyncTriggerCondition。
* 更改：GameplayTasksComponent 和 AbilitySystemComponent 现在支持注册的子对象 API。
* 更改：添加了更好的日志记录以指示 Gameplay Abilities 未能激活的原因。
* 更改：删除了 AbilitySystem.Debug.NextTarget 和 PrevTarget 命令，转而使用全局 HUD NextDebugTarget 和 PrevDebugTarget 命令。

https://docs.unrealengine.com/5.1/en-US/unreal-engine-5.1-release-notes/

<a name="changelog-5.0"></a>
### 5.0

https://docs.unrealengine.com/5.0/en-US/unreal-engine-5.0-release-notes/

<a name="changelog-4.27"></a>
### 4.27
* 崩溃修复：修复了根运动源问题，其中网络客户端可能在 Actor 完成执行使用具有随时间强度修饰符的恒定力根运动任务的能力时崩溃。
* 错误修复：修复了使用 GameplayCues 时编辑器加载时间的回归。
* 错误修复：GameplayEffectsContainer 的 `SetActiveGameplayEffectLevel` 方法如果在设置相同的 EffectLevel 时将不再使 FastArray 变脏。
* 错误修复：修复了 GameplayEffect 混合复制模式中的边缘情况，其中不是由网络连接明确拥有但使用 `GetNetConnection` 中该连接的 Actors 将不会收到混合复制更新。
* 错误修复：修复了 GameplayAbility 的类方法 `EndAbility` 中发生的无限递归，这是通过从 `K2_OnEndAbility` 再次调用 `EndAbility` 调用的。
* 错误修复：如果标签在注册之前加载，GameplayTags Blueprint 引脚将不再被静默清除。它们现在的工作方式与 GameplayTag 变量相同，并且两者的行为都可以通过项目设置中的 ClearInvalidTags 选项进行更改。
* 错误修复：提高了 GameplayTag 操作的线程安全性。
* 新增：向 GameplayAbility 的 `K2_CanActivateAbility` 方法公开了 SourceObject。
* 新增：原生 GameplayTags。引入了新的 `FNativeGameplayTag`，这使得可以创建一次性原生标签，这些标签在模块加载和卸载时正确注册和取消注册。
* 新增：更新了 `GiveAbilityAndActivateOnce` 以传入 FGameplayEventData 参数。
* 新增：改进了 GameplayAbilities 插件中的 ScalableFloats，以支持从新的数据注册表系统动态查找曲线表。添加了 ScalableFloat 头文件，以便在能力插件之外更容易地重用通用结构。
* 新增：添加了代码支持，通过 GameplayTagsEditorModule 在其他编辑器自定义中使用 GameplayTag UI。
* 新增：修改了 UGameplayAbility 的 PreActivate 方法以可选地接受触发事件数据。
* 新增：添加了更多支持，以使用项目特定的过滤器在编辑器中过滤 GameplayTags。`OnFilterGameplayTag` 提供引用属性和标签源，因此你可以根据请求标签的资产过滤标签。
* 新增：添加了选项，以在初始化后调用 GameplayEffectSpec 的类方法 `SetContext` 时保留原始捕获的 SourceTags。
* 新增：改进了从特定插件注册 GameplayTags 的 UI。新的标签 UI 现在允许你在磁盘上为新添加的 GameplayTag 源选择插件位置。
* 新增：向 Sequencer 添加了一个新轨道，以允许在使用 GameplayAbiltiySystem 构建的 Actors 上触发通知状态。像通知一样，GameplayCueTrack 可以利用基于范围的事件或基于触发的事件。
* 更改：更改了 GameplayCueInterface 以通过引用传递 GameplayCueParameters 结构。
* 优化：对加载和重新生成 GameplayTag 表进行了几项性能改进，以便优化此选项。

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_27/

<a name="changelog-4.26"></a>
### 4.26
* GAS 插件不再标记为 beta。
* 崩溃修复：修复了在没有有效标签源选择的情况下添加 gameplay 标签时的崩溃。
* 崩溃修复：向消息添加了路径字符串参数以修复 UGameplayCueManager::VerifyNotifyAssetIsInValidPath 中的崩溃。
* 崩溃修复：修复了 AbilitySystemComponent_Abilities 中使用指针而不检查它时的访问冲突崩溃。
* 错误修复：修复了堆叠 GE 不重置额外实例应用的效果持续时间的问题。
* 错误修复：修复了导致 CancelAllAbilities 仅取消非实例化能力的问题。
* 新增：向 gameplay ability 提交函数添加了可选标签参数。
* 新增：向 PlayMontageAndWait ability 任务添加了 StartTimeSeconds 并改进了注释。
* 新增：向 FGameplayAbilitySpec 添加了标签容器 "DynamicAbilityTags"。这些是可选的能力标签，随规范复制。它们还被应用的游戏效果捕获为源标签。
* 新增：GameplayAbility IsLocallyControlled 和 HasAuthority 函数现在可以从 Blueprint 调用。
* 新增：视觉记录器现在只在我们当前正在记录视觉日志数据时收集和存储即时 GE 的信息。
* 新增：添加了对蓝图节点中的 gameplay 属性引脚的重定向器的支持。
* 新增：添加了新功能，当根运动相关的 ability 任务结束时，它们将移动组件的移动模式返回到任务开始之前的移动模式。

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_26/

<a name="changelog-4.25.1"></a>
### 4.25.1
* 修复了！UE-92787 保存带有获取浮点属性节点并且属性引脚内联设置的蓝图时崩溃
* 修复了！UE-92810 生成带有内联更改的实例可编辑 gameplay 标签属性的 actor 时崩溃

<a name="changelog-4.25"></a>
### 4.25
* 修复了 `RootMotionSource` `AbilityTasks` 的预测
* [`GAMEPLAYATTRIBUTE_REPNOTIFY()`](#concepts-as-attributes) 现在还接受旧的 `Attribute` 值。我们必须将其作为可选参数提供给我们的 `OnRep` 函数。以前，它读取属性值以获取旧值。但是，如果从复制函数调用，旧值在到达 SetBaseAttributeValueFromReplication 之前已经被丢弃，所以我们会得到新值。
* 向 `UGameplayAbility` 添加了 [`NetSecurityPolicy`](#concepts-ga-netsecuritypolicy)。
* 崩溃修复：修复了在没有有效标签源选择的情况下添加 gameplay 标签时的崩溃。
* 崩溃修复：删除了攻击者通过能力系统使服务器崩溃的几种方法。
* 崩溃修复：我们现在确保在检查标签要求之前有一个 GameplayEffect 定义。
* 错误修复：修复了 gameplay 标签类别不适用于蓝图中函数参数的问题，如果它们是函数终止符节点的一部分。
* 错误修复：修复了 gameplay 效果的标签在多个视口中未复制的问题。
* 错误修复：修复了 gameplay ability 规范可能在循环遍历触发的能力时被 InternalTryActivateAbility 函数无效化的错误。
* 错误修复：更改了在标签计数容器内更新 gameplay 标签的方式。在移除 gameplay 标签时延迟父标签的更新时，我们现在将在父标签更新后调用与更改相关的委托。这确保委托广播时标签表处于一致状态。
* 错误修复：我们现在在确认目标时在迭代生成的目标 actor 数组之前创建该数组的副本，因为某些回调可能会修改该数组。
* 错误修复：修复了堆叠 GameplayEffects 不重置额外实例应用的效果持续时间并且具有由调用者设置的持续时间的问题，堆栈中只有第一个实例的持续时间会正确设置。堆栈中的所有其他 GE 规范的持续时间将为 1 秒。添加了自动化测试来检测这种情况。
* 错误修复：修复了处理游戏事件委托修改游戏事件委托列表时可能发生的错误。
* 错误修复：修复了导致 GiveAbilityAndActivateOnce 行为不一致的错误。
* 错误修复：重新排序了 FGameplayEffectSpec::Initialize 中的一些操作以处理潜在的排序依赖性。
* 新增：UGameplayAbility 现在具有 OnRemoveAbility 函数。它遵循与 OnGiveAbility 相同的模式，并且仅在能力的主实例或类默认对象上调用。
* 新增：显示被阻止的能力标签时，调试文本现在包括被阻止标签的总数。
* 新增：将 UAbilitySystemComponent::InternalServerTryActiveAbility 重命名为 UAbilitySystemComponent::InternalServerTryActivateAbility。调用 InternalServerTryActiveAbility 的代码现在应该调用 InternalServerTryActivateAbility。
* 新增：在添加或删除标签时继续使用过滤器文本来显示 gameplay 标签。以前的行为清除了过滤器。
* 新增：在编辑器中添加新标签时不重置标签源。
* 新增：添加了查询能力系统组件以获取具有一组指定标签的所有活动游戏效果的功能。新函数称为 GetActiveEffectsWithAllTags，可以通过代码或蓝图访问。
* 新增：当根运动相关的 ability 任务结束时，它们现在将移动组件的移动模式返回到任务开始之前的移动模式。
* 新增：将 SpawnedAttributes 设为 transient，以便它不会保存可能变得陈旧和不正确的数据。添加了空检查以防止任何当前保存的陈旧数据传播。这防止了坏数据存储在 SpawnedAttributes 中的问题。
* API 更改：AddDefaultSubobjectSet 已被弃用。应改用 AddAttributeSetSubobject。
* 新增：Gameplay Abilities 现在可以指定要在其上播放 montage 的 Anim 实例。

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_25/

<a name="changelog-4.24"></a>
### 4.24
* 修复了蓝图节点 `Attribute` 变量在编译时重置为 `None` 的问题。
* 需要调用 [`UAbilitySystemGlobals::InitGlobalData()`](#concepts-asg-initglobaldata) 才能使用 [`TargetData`](#concepts-targeting-data)，否则将得到 `ScriptStructCache` 错误，客户端将从服务器断开连接。我的建议是现在总是在每个项目中调用它，而在 4.24 之前它是可选的。
* 修复了将 `GameplayTag` setter 复制到以前没有定义变量的蓝图时崩溃的问题。
* `UGameplayAbility::MontageStop()` 函数现在正确使用 `OverrideBlendOutTime` 参数。
* 修复了组件上的 `GameplayTag` 查询变量在编辑时未修改的问题。
* 添加了 `GameplayEffectExecutionCalculations` 的功能，以支持针对"临时变量"的作用域修饰符，这些变量不需要由属性捕获支持。
  * 实现基本上允许创建由 `GameplayTag` 标识的聚合器，作为执行向操作公开临时值以通过作用域修饰符进行操作的一种方式；现在可以构建想要可操作值而不需要从源或目标捕获的公式。
  * 要使用，执行必须向新成员变量 `ValidTransientAggregatorIdentifiers` 添加一个标签；这些标签将显示在底部作用域修饰符的计算修饰符数组中，标记为临时变量 - 相应地更新了详细的自定义定义以支持功能。
* 添加了受限标签的质量改进。删除了受限 `GameplayTag` 源的默认选项。在添加受限标签时，我们不再重置源，以便更容易连续添加多个。
* `APawn::PossessedBy()` 现在将 `Pawn` 的所有者设置为新 `Controller`。很有用，因为 [混合复制模式](#concepts-asc-rm) 期望 `Pawn` 的所有者是 `Controller`（如果 `ASC` 位于 `Pawn` 上）。
* 修复了 `FAttributeSetInitterDiscreteLevels` 中 POD（纯旧数据）的错误。

https://docs.unrealengine.com/en-US/WhatsNew/Builds/ReleaseNotes/4_24/

**[⬆ 返回顶部](#table-of-contents)**
