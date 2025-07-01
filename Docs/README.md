# Comedot 组件系统文档

欢迎来到 Comedot 组件系统文档！本文档集合将帮助您理解和使用 Comedot 的强大组件架构。

## 📚 文档目录

### 核心文档

1. **[组件系统概览](Components-Overview.md)**
   - 了解 Comedot 的 Entity-Component 架构
   - 组件分类和核心特性
   - 生命周期管理

2. **[组件 API 文档](Components-API.md)**
   - 详细的 API 参考
   - 核心类的方法和属性
   - 使用示例

3. **[创建自定义组件指南](Creating-Custom-Components.md)**
   - 步骤化的组件开发教程
   - 不同类型组件的示例
   - 测试和调试技巧

4. **[组件最佳实践](Components-Best-Practices.md)**
   - 设计原则和架构模式
   - 性能优化建议
   - 常见陷阱和解决方案

## 🚀 快速开始

如果您是第一次使用 Comedot 组件系统，建议按以下顺序阅读：

1. 首先阅读 [组件系统概览](Components-Overview.md) 了解基本概念
2. 查看 [组件 API 文档](Components-API.md) 熟悉核心接口
3. 通过 [创建自定义组件指南](Creating-Custom-Components.md) 动手实践
4. 参考 [组件最佳实践](Components-Best-Practices.md) 优化您的设计

## 🧩 组件分类

### 控制组件 (Control)
负责处理输入和游戏控制逻辑
- `InputComponent` - 输入处理
- `ActionControlComponent` - 动作控制
- `PlatformerControlComponent` - 平台游戏控制
- `JumpComponent` - 跳跃功能

### 战斗组件 (Combat)
处理伤害、生命值和战斗相关功能
- `HealthComponent` - 生命值管理
- `DamageComponent` - 伤害处理
- `GunComponent` - 武器功能
- `FactionComponent` - 阵营系统

### 移动组件 (Movement)
处理各种移动和定位功能
- `LinearMotionComponent` - 线性运动
- `PathFollowComponent` - 路径跟随
- `TileBasedPositionComponent` - 基于瓦片的定位
- `NavigationComponent` - 导航功能

### 视觉组件 (Visual)
处理图形显示和视觉效果
- `CameraComponent` - 摄像机控制
- `PlatformerAnimationComponent` - 平台游戏动画
- `HealthVisualComponent` - 生命值显示
- `DamageVisualComponent` - 伤害视觉效果

### 游戏玩法组件 (Gameplay)
处理游戏机制和逻辑
- `ActionsComponent` - 动作系统
- `InventoryComponent` - 背包系统
- `UpgradesComponent` - 升级系统
- `InjectorComponent` - 注入器

### 物理组件 (Physics)
处理物理交互和碰撞
- `CharacterBodyComponent` - 角色物理体
- `AreaCollisionComponent` - 区域碰撞
- `PlatformerPhysicsComponent` - 平台游戏物理
- `GravityComponent` - 重力效果

## 🛠 核心基类

- **`Component`** - 所有组件的基类
- **`NodeModifierComponentBase`** - 动态修改组件和节点
- **`DebugComponent`** - 调试和可视化工具
- **`InputDependentComponentBase`** - 输入依赖组件基类
- **`AreaComponentBase`** - 区域检测组件基类

## 💡 关键概念

### Entity-Component 架构
- **Entity**: 游戏对象的容器，作为组件的"脚手架"
- **Component**: 实现具体功能的模块
- **组合优于继承**: 通过组合组件创建复杂行为

### 生命周期
1. `validateParent()` - 验证父节点
2. `_enter_tree()` - 进入场景树
3. `registerEntity()` - 注册到实体
4. `_exit_tree()` - 退出场景树
5. `unregisterEntity()` - 从实体注销

### 组件通信
- 通过 `coComponents` 字典访问同级组件
- 使用信号进行解耦通信
- 通过 `findCoComponent()` 安全查找组件

## 🔧 实用工具

### 调试功能
- `DebugComponent` - 实时属性监控和图表
- 详细的日志系统
- 可视化调试气泡

### 修改器系统
- 动态添加/删除组件
- 节点操作功能
- Payload 执行系统

## 📖 示例代码

### 创建简单组件
```gdscript
class_name MyComponent
extends Component

func getRequiredComponents() -> Array[Script]:
    return [HealthComponent]

func _ready():
    var health = coComponents.get(&"HealthComponent")
    if health:
        health.healthChanged.connect(_onHealthChanged)
```

### 使用修改器组件
```gdscript
class_name PowerUpModifier
extends NodeModifierComponentBase

func _ready():
    componentsToCreate = [SpeedBoostComponent]
    performAllModifications()
```

## 🤝 贡献

如果您发现文档中的错误或有改进建议，请：

1. 查看现有的问题和建议
2. 提交详细的问题报告
3. 提供代码示例或修复建议

## 📄 许可证

本文档遵循项目的许可证条款。

---

**Comedot** - 让游戏开发更加模块化和高效！ 