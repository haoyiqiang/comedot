# Comedot 组件最佳实践

## 设计原则

### 1. 单一职责原则 (Single Responsibility Principle)

每个组件应该只负责一个明确的功能领域。

**好的例子：**
```gdscript
# ✅ 专注于生命值管理
class_name HealthComponent
extends Component

# ✅ 专注于移动控制
class_name MovementComponent
extends Component
```

**避免的例子：**
```gdscript
# ❌ 混合了多种职责
class_name PlayerManagerComponent
extends Component
# 处理生命值、移动、输入、AI... 太多功能
```

### 2. 组合优于继承

使用组件组合而不是深层继承来构建复杂行为。

```gdscript
# ✅ 通过组合创建复杂实体
func createPlayer():
    var player = Entity.new()
    player.add_child(HealthComponent.new())
    player.add_child(InputComponent.new())
    player.add_child(MovementComponent.new())
    player.add_child(WeaponComponent.new())
    return player
```

### 3. 松耦合原则

组件间应通过信号或接口通信，避免直接依赖。

```gdscript
# ✅ 使用信号通信
class_name WeaponComponent
extends Component

signal weaponFired(damage: float)

# 其他组件监听信号
func _ready():
    var weaponComp = coComponents.get(&"WeaponComponent")
    if weaponComp:
        weaponComp.weaponFired.connect(_onWeaponFired)
```

## 组件依赖管理

### 1. 明确声明依赖

总是在 `getRequiredComponents()` 中声明必需的组件。

```gdscript
func getRequiredComponents() -> Array[Script]:
    return [
        HealthComponent,      # 必需的生命值组件
        CharacterBodyComponent, # 必需的物理体组件
    ]
```

### 2. 安全的组件访问

使用 `get()` 方法安全访问可选组件。

```gdscript
# ✅ 安全访问
var weaponComp = coComponents.get(&"WeaponComponent")
if weaponComp:
    weaponComp.fire()

# ❌ 可能崩溃
var weaponComp = coComponents.WeaponComponent  # 如果不存在会崩溃
```

### 3. 避免循环依赖

设计组件时避免相互依赖。

```gdscript
# ❌ 循环依赖
class_name ComponentA:
    func getRequiredComponents(): return [ComponentB]

class_name ComponentB:
    func getRequiredComponents(): return [ComponentA]

# ✅ 使用中介者模式或信号解耦
```

## 性能优化

### 1. 缓存组件引用

在 `_ready()` 中缓存经常使用的组件引用。

```gdscript
class_name DamageComponent
extends Component

var healthComp: HealthComponent

func _ready():
    healthComp = coComponents.get(&"HealthComponent")

func dealDamage(amount: float):
    if healthComp:  # 使用缓存的引用
        healthComp.takeDamage(amount)
```

### 2. 条件性处理

只在必要时启用 `_process()` 和 `_physics_process()`。

```gdscript
func _ready():
    # 只有在需要时才启用处理
    set_process(needsContinuousUpdate)
    set_physics_process(needsPhysicsUpdate)
    
func setEnabled(enabled: bool):
    isEnabled = enabled
    set_process(enabled and needsContinuousUpdate)
```

### 3. 对象池化

为频繁创建/销毁的组件使用对象池。

```gdscript
# 子弹组件池
class_name BulletPool
extends Node

var bulletPool: Array[BulletComponent] = []

func getBullet() -> BulletComponent:
    if bulletPool.is_empty():
        return BulletComponent.new()
    return bulletPool.pop_back()

func returnBullet(bullet: BulletComponent):
    bullet.reset()
    bulletPool.append(bullet)
```

## 调试和测试

### 1. 调试支持

为组件添加适当的调试功能。

```gdscript
class_name MovementComponent
extends Component

func _ready():
    if debugMode:
        printDebug("MovementComponent initialized")
        # 添加到调试观察列表
        if has_method("addToWatchList"):
            addToWatchList("velocity", self, "velocity")

func _process(delta):
    if debugMode:
        emitDebugBubble(str("Speed: ", velocity.length()))
```

### 2. 单元测试

为复杂组件编写测试。

```gdscript
# TestMovementComponent.gd
extends GutTest

func test_movement_component_initialization():
    var entity = Entity.new()
    var movement = MovementComponent.new()
    entity.add_child(movement)
    
    assert_not_null(movement.parentEntity)
    assert_eq(movement.velocity, Vector2.ZERO)

func test_movement_with_input():
    var entity = setup_test_entity()
    var movement = entity.get_component("MovementComponent")
    
    movement.setDirection(Vector2.RIGHT)
    movement._process(0.016)  # Simulate one frame
    
    assert_gt(movement.velocity.x, 0)
```

### 3. 配置验证

验证组件配置的正确性。

```gdscript
func _get_configuration_warnings() -> PackedStringArray:
    var warnings = super._get_configuration_warnings()
    
    if maxHealth <= 0:
        warnings.append("maxHealth must be greater than 0")
    
    if not has_required_resources():
        warnings.append("Missing required resources")
    
    return warnings
```

## 架构模式

### 1. 事件驱动架构

使用信号进行组件间通信。

```gdscript
# 事件总线模式
class_name GameEvents
extends Node

signal player_died(player: Entity)
signal enemy_spawned(enemy: Entity)
signal item_collected(item: Entity, player: Entity)

# 组件监听全局事件
func _ready():
    GameEvents.player_died.connect(_onPlayerDied)
```

### 2. 状态机模式

为复杂行为使用状态机。

```gdscript
class_name AIComponent
extends Component

enum State { IDLE, PATROL, CHASE, ATTACK }
var currentState: State = State.IDLE

func _process(delta):
    match currentState:
        State.IDLE: _processIdle(delta)
        State.PATROL: _processPatrol(delta)
        State.CHASE: _processChase(delta)
        State.ATTACK: _processAttack(delta)
```

### 3. 命令模式

为动作系统使用命令模式。

```gdscript
class_name ActionCommand
extends Resource

func execute(entity: Entity) -> void:
    pass

class_name MoveCommand
extends ActionCommand

var direction: Vector2

func execute(entity: Entity):
    var movement = entity.get_component("MovementComponent")
    if movement:
        movement.move(direction)
```

## 代码组织

### 1. 文件结构

```
Components/
├── Core/           # 核心基类
├── Combat/         # 战斗相关
├── Movement/       # 移动相关
├── Control/        # 控制相关
├── Visual/         # 视觉效果
├── AI/            # 人工智能
└── Utility/       # 工具组件
```

### 2. 命名约定

- 组件类名以 `Component` 结尾
- 信号使用动词过去式或现在进行时
- 方法名使用动词开头
- 属性使用名词或形容词

```gdscript
class_name HealthComponent  # ✅
class_name Health          # ❌

signal healthChanged       # ✅ 
signal health_change       # ❌

func takeDamage()         # ✅
func damage()             # ❌

var maxHealth: float      # ✅
var max_hp: float         # ❌
```

### 3. 文档注释

为每个组件添加详细的文档注释。

```gdscript
## 管理实体的生命值，包括受伤、治疗和死亡逻辑。
##
## 这个组件处理生命值的核心逻辑，包括：
## - 受到伤害时减少生命值
## - 治疗时恢复生命值  
## - 生命值为零时触发死亡
##
## 依赖组件：无
## 可选组件：ShieldComponent（如果存在，伤害将先扣除护盾）
##
## 信号：
## - healthChanged(newHealth, maxHealth) - 生命值改变时发出
## - died() - 死亡时发出
##
## 示例用法：
## [codeblock]
## var health = entity.get_component("HealthComponent")
## health.takeDamage(25.0)
## health.heal(10.0)
## [/codeblock]

class_name HealthComponent
extends Component
```

## 常见陷阱和解决方案

### 1. 初始化顺序问题

**问题：** 组件A依赖组件B，但B还未初始化。

**解决方案：** 使用延迟初始化或信号通知。

```gdscript
func _ready():
    # 延迟到下一帧确保所有组件都已初始化
    call_deferred("initialize")

func initialize():
    var requiredComp = coComponents.get(&"RequiredComponent")
    if requiredComp:
        setup_with_component(requiredComp)
```

### 2. 内存泄漏

**问题：** 组件之间的信号连接没有正确断开。

**解决方案：** 在组件销毁时断开连接。

```gdscript
func _exit_tree():
    # 断开所有信号连接
    if connectedSignal.is_connected(self._onSignal):
        connectedSignal.disconnect(self._onSignal)
```

### 3. 过度依赖

**问题：** 组件依赖太多其他组件。

**解决方案：** 重新设计，拆分职责或使用事件系统。

```gdscript
# ❌ 过度依赖
func getRequiredComponents():
    return [CompA, CompB, CompC, CompD, CompE]  # 太多依赖

# ✅ 减少依赖，使用事件
func _ready():
    GameEvents.someEvent.connect(_handleEvent)  # 通过事件系统通信
```

## 总结

遵循这些最佳实践将帮助您：

1. 创建可维护和可扩展的组件
2. 避免常见的架构问题
3. 提高代码的可读性和可测试性
4. 优化游戏性能
5. 简化调试过程

记住，好的组件设计是迭代的过程。随着项目的发展，不断重构和改进您的组件架构，以保持代码的清洁和高效。 