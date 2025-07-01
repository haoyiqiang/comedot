# 创建自定义组件指南

## 概述

本指南将引导您创建自定义的 Comedot 组件，涵盖从基础组件到复杂功能组件的开发过程。

## 基础组件创建

### 1. 选择基类

根据组件功能选择合适的基类：

- `Component` - 基础组件（最常用）
- `NodeModifierComponentBase` - 需要修改节点/组件的功能
- `InputDependentComponentBase` - 依赖输入的组件
- `CharacterBodyDependentComponentBase` - 依赖角色物理体的组件
- `AreaComponentBase` - 区域检测组件
- `ActionTargetingComponentBase` - 动作瞄准组件
- `TimerComponentBase` - 定时器组件

### 2. 基础模板

```gdscript
## 组件功能描述
## 更详细的使用说明和注意事项

class_name MyCustomComponent
extends Component

#region Parameters
@export var myParameter: float = 1.0
@export var isEnabled: bool = true
#endregion

#region State
var currentState: int = 0
#endregion

#region Required Components
func getRequiredComponents() -> Array[Script]:
    # 返回此组件依赖的其他组件
    return [HealthComponent] # 示例：依赖生命值组件
#endregion

#region Life Cycle
func _ready() -> void:
    # 组件初始化逻辑
    if not isEnabled: return
    
    # 访问依赖的组件
    var healthComp = coComponents.get(&"HealthComponent")
    if healthComp:
        # 连接信号或初始化
        pass

func _process(delta: float) -> void:
    if not isEnabled: return
    # 每帧更新逻辑
#endregion
```

## 不同类型组件示例

### 1. 简单状态组件

```gdscript
## 管理实体的能量值
class_name EnergyComponent
extends Component

@export var maxEnergy: float = 100.0
@export var currentEnergy: float = 100.0
@export var regenRate: float = 5.0

signal energyChanged(newValue: float, maxValue: float)
signal energyDepleted

func _ready() -> void:
    energyChanged.emit(currentEnergy, maxEnergy)

func _process(delta: float) -> void:
    if currentEnergy < maxEnergy:
        addEnergy(regenRate * delta)

func consumeEnergy(amount: float) -> bool:
    if currentEnergy >= amount:
        currentEnergy -= amount
        energyChanged.emit(currentEnergy, maxEnergy)
        if currentEnergy <= 0:
            energyDepleted.emit()
        return true
    return false

func addEnergy(amount: float) -> void:
    currentEnergy = min(currentEnergy + amount, maxEnergy)
    energyChanged.emit(currentEnergy, maxEnergy)
```

### 2. 输入依赖组件

```gdscript
## 处理特殊技能输入
class_name SkillControlComponent
extends InputDependentComponentBase

@export var skillAction: StringName = &"special_skill"
@export var cooldownTime: float = 3.0

var isOnCooldown: bool = false

func getRequiredComponents() -> Array[Script]:
    return [ActionsComponent]

func _ready() -> void:
    super._ready()
    # 连接输入事件
    if InputComponent:
        InputComponent.connectInputAction(skillAction, _onSkillPressed)

func _onSkillPressed() -> void:
    if isOnCooldown: return
    
    var actionsComp = coComponents.get(&"ActionsComponent")
    if actionsComp:
        actionsComp.executeSkill("special")
        _startCooldown()

func _startCooldown() -> void:
    isOnCooldown = true
    await get_tree().create_timer(cooldownTime).timeout
    isOnCooldown = false
```

### 3. 物理依赖组件

```gdscript
## 磁力吸引组件
class_name MagnetComponent
extends AreaComponentBase

@export var attractionForce: float = 500.0
@export var maxDistance: float = 100.0
@export var targetLayers: int = 1

var attractedBodies: Array[RigidBody2D] = []

func getRequiredComponents() -> Array[Script]:
    return [AreaCollisionComponent]

func _ready() -> void:
    super._ready()
    
    # 连接区域信号
    var areaComp = coComponents.get(&"AreaCollisionComponent")
    if areaComp:
        areaComp.bodyEntered.connect(_onBodyEntered)
        areaComp.bodyExited.connect(_onBodyExited)

func _process(delta: float) -> void:
    for body in attractedBodies:
        if not is_instance_valid(body): continue
        
        var direction = (parentEntity.global_position - body.global_position)
        var distance = direction.length()
        
        if distance <= maxDistance:
            var force = direction.normalized() * attractionForce / distance
            body.apply_central_force(force)

func _onBodyEntered(body: Node2D) -> void:
    if body is RigidBody2D and body.collision_layer & targetLayers:
        attractedBodies.append(body)

func _onBodyExited(body: Node2D) -> void:
    attractedBodies.erase(body)
```

### 4. 修改器组件

```gdscript
## 受伤时添加无敌效果的组件
class_name InvincibilityOnHitModifier
extends NodeModifierComponentBase

@export var invincibilityDuration: float = 1.0
@export var invincibilityComponent: Script = InvulnerabilityComponent

func getRequiredComponents() -> Array[Script]:
    return [DamageReceivingComponent]

func _ready() -> void:
    # 连接受伤信号
    var damageComp = coComponents.get(&"DamageReceivingComponent")
    if damageComp:
        damageComp.damageReceived.connect(_onDamageReceived)

func _onDamageReceived(damageAmount: float, damageSource: Node) -> void:
    # 临时添加无敌组件
    componentsToCreate = [invincibilityComponent]
    createComponents()
    
    # 定时移除
    await get_tree().create_timer(invincibilityDuration).timeout
    
    var invComp = parentEntity.findFirstComponentByType(invincibilityComponent)
    if invComp:
        invComp.requestDeletion()
```

## 高级功能

### 1. 组件通信

```gdscript
# 使用信号进行组件通信
class_name WeaponComponent
extends Component

signal weaponFired(weaponData: WeaponData)

func fireWeapon() -> void:
    weaponFired.emit(weaponData)

# 在其他组件中监听
func _ready() -> void:
    var weaponComp = coComponents.get(&"WeaponComponent")
    if weaponComp:
        weaponComp.weaponFired.connect(_onWeaponFired)
```

### 2. 数据驱动配置

```gdscript
## 可配置的效果组件
class_name EffectComponent
extends Component

@export var effectResource: EffectResource

func _ready() -> void:
    if effectResource:
        applyEffect(effectResource)

func applyEffect(effect: EffectResource) -> void:
    # 根据资源配置应用效果
    match effect.effectType:
        EffectResource.Type.DAMAGE:
            _applyDamageEffect(effect)
        EffectResource.Type.HEAL:
            _applyHealEffect(effect)
```

### 3. 调试支持

```gdscript
func _ready() -> void:
    # 添加调试信息
    if debugMode:
        printDebug("MyComponent initialized")
        emitDebugBubble("Ready!")

func _process(delta: float) -> void:
    if debugMode:
        # 显示调试信息
        printChange("currentValue", oldValue, newValue)
```

## 测试组件

### 1. 创建测试场景

```gdscript
# TestScene.gd
extends Node2D

func _ready() -> void:
    # 创建测试实体
    var testEntity = Entity.new()
    add_child(testEntity)
    
    # 添加要测试的组件
    var myComponent = MyCustomComponent.new()
    testEntity.add_child(myComponent)
    
    # 添加依赖组件
    var healthComp = HealthComponent.new()
    testEntity.add_child(healthComp)
```

### 2. 单元测试示例

```gdscript
func test_component_initialization():
    var entity = Entity.new()
    var component = MyCustomComponent.new()
    
    entity.add_child(component)
    
    assert(component.parentEntity == entity)
    assert(component.isEnabled == true)
```

## 性能优化

### 1. 避免每帧查找

```gdscript
# 差的做法
func _process(delta: float) -> void:
    var healthComp = coComponents.get(&"HealthComponent")
    # ...

# 好的做法
var healthComp: HealthComponent

func _ready() -> void:
    healthComp = coComponents.get(&"HealthComponent")

func _process(delta: float) -> void:
    if healthComp:
        # 使用缓存的引用
        pass
```

### 2. 条件性处理

```gdscript
func _ready() -> void:
    # 只在需要时启用处理
    set_process(needsProcessing)
    set_physics_process(needsPhysicsProcessing)
```

## 最佳实践

1. **单一职责**: 每个组件只负责一个明确的功能
2. **依赖声明**: 清楚声明组件依赖关系
3. **信号使用**: 使用信号进行组件间通信
4. **配置驱动**: 使用导出变量和资源进行配置
5. **错误处理**: 检查组件引用的有效性
6. **文档注释**: 为组件添加清晰的文档注释
7. **调试支持**: 提供适当的调试信息
8. **性能考虑**: 避免不必要的计算和查找

## 常见错误

1. **忘记声明依赖**: 导致运行时错误
2. **循环依赖**: 造成初始化问题
3. **过度耦合**: 组件间过于依赖
4. **内存泄漏**: 忘记断开信号连接
5. **性能问题**: 在 _process 中进行重复查找

## 调试技巧

1. 使用 `DebugComponent` 监控属性
2. 启用 `debugMode` 查看详细日志
3. 使用 `printTrace` 追踪执行流程
4. 检查组件依赖关系
5. 使用 Godot 调试器设置断点 