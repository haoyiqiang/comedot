# Comedot 组件系统概览

## 简介

Comedot 是一个基于组件-实体架构的 Godot 游戏开发框架。它采用组合模式而非继承模式来构建游戏对象，提供了高度模块化和可重用的游戏开发解决方案。

## 核心架构

### Entity-Component 系统

- **Entity（实体）**: 游戏对象的"脚手架"，作为各种组件的容器
- **Component（组件）**: 实现具体行为或属性的功能模块
- **组合优于继承**: 通过组合不同的组件来创建复杂的游戏对象

### 核心类层次结构

```
Component (abstract)
├── NodeModifierComponentBase (abstract)
├── DebugComponent
├── InputDependentComponentBase (abstract)
├── CharacterBodyDependentComponentBase (abstract)
├── AreaComponentBase (abstract)
├── ActionTargetingComponentBase (abstract)
├── TimerComponentBase (abstract)
└── 各种具体组件...
```

## 组件分类

### 1. 控制组件 (Control)
处理输入和游戏控制逻辑
- `ActionControlComponent` - 动作控制
- `InputComponent` - 输入处理
- `PlatformerControlComponent` - 平台游戏控制
- `TileBasedControlComponent` - 基于瓦片的控制
- `MouseTrackingComponent` - 鼠标跟踪
- `JumpComponent` - 跳跃功能

### 2. 战斗组件 (Combat)
处理伤害、生命值和战斗相关功能
- `HealthComponent` - 生命值管理
- `DamageComponent` - 伤害处理
- `GunComponent` - 武器功能
- `FactionComponent` - 阵营系统
- `ShieldedHealthComponent` - 护盾生命值

### 3. 移动组件 (Movement)
处理各种移动和定位功能
- `LinearMotionComponent` - 线性运动
- `PathFollowComponent` - 路径跟随
- `ChaseComponent` - 追逐行为
- `TileBasedPositionComponent` - 基于瓦片的定位
- `WaveMotionComponent` - 波浪运动
- `NavigationComponent` - 导航功能

### 4. 视觉组件 (Visual)
处理图形显示和视觉效果
- `CameraComponent` - 摄像机控制
- `HealthVisualComponent` - 生命值显示
- `PlatformerAnimationComponent` - 平台游戏动画
- `DamageVisualComponent` - 伤害视觉效果
- `LabelComponent` - 标签显示

### 5. 游戏玩法组件 (Gameplay)
处理游戏机制和逻辑
- `ActionsComponent` - 动作系统
- `InventoryComponent` - 背包系统
- `UpgradesComponent` - 升级系统
- `CooldownComponent` - 冷却时间
- `InjectorComponent` - 注入器

### 6. 物理组件 (Physics)
处理物理交互和碰撞
- `CharacterBodyComponent` - 角色物理体
- `AreaCollisionComponent` - 区域碰撞
- `PlatformerPhysicsComponent` - 平台游戏物理
- `GravityComponent` - 重力效果
- `VelocityClampComponent` - 速度限制

### 7. 回合制组件 (TurnBased)
处理回合制游戏机制
- `TurnBasedAnimationComponent` - 回合制动画
- 其他回合制相关组件

### 8. AI 组件 (AI)
处理人工智能行为
- `TimerAgentComponentBase` - 基于定时器的AI代理

### 9. 数据组件 (Data)
处理数据和统计
- `StatModifierComponent` - 属性修改器

### 10. 对象组件 (Objects)
处理游戏对象相关功能
- `CollectibleComponent` - 可收集物品
- 其他对象相关组件

## 组件生命周期

### 初始化顺序
1. `validateParent()` - 验证父节点（通过 NOTIFICATION_PARENTED）
2. `_enter_tree()` - 进入场景树
3. `registerEntity()` - 注册到实体
4. `checkRequiredComponents()` - 检查依赖组件

### 销毁顺序
1. `_exit_tree()` - 退出场景树
2. `unregisterEntity()` - 从实体注销（通过 NOTIFICATION_UNPARENTED）
3. `willRemoveFromEntity` 信号发出

## 核心特性

### 组件依赖
- 通过 `getRequiredComponents()` 定义依赖关系
- 自动验证组件依赖
- 支持可选组件检查

### 实体查找
- `parentEntity` - 获取父实体
- `coComponents` - 访问同级组件
- `findCoComponent()` - 查找特定组件

### 调试支持
- `DebugComponent` - 可视化调试
- `debugMode` - 调试模式开关
- 详细的日志系统
- 实时属性监控

### 修改器支持
- `NodeModifierComponentBase` - 动态修改节点和组件
- 支持添加/删除组件
- 支持节点操作
- Payload 系统支持

## 使用建议

1. **组件设计原则**: 每个组件应该专注于单一职责
2. **依赖管理**: 明确定义组件间的依赖关系
3. **性能考虑**: 合理使用调试功能，避免在生产环境中启用
4. **扩展性**: 继承适当的基类来创建自定义组件

## 相关文档

- [组件详细 API 文档](Components-API.md)
- [创建自定义组件指南](Creating-Custom-Components.md)
- [组件最佳实践](Components-Best-Practices.md) 