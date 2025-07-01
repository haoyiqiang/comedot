# Comedot 组件 API 文档

## Component (基类)

### 概述
所有组件的抽象基类，提供核心的组件功能和生命周期管理。

### 属性

#### 高级参数
- `shouldCheckGrandparentsForEntity: bool = false` - 是否检查祖父节点寻找 Entity
- `allowNonEntityParent: bool = false` - 是否允许非 Entity 父节点

#### 核心属性
- `parentEntity: Entity` - 父实体引用
- `coComponents: Dictionary[StringName, Component]` - 同级组件字典

#### 调试属性
- `debugMode: bool` - 调试模式开关
- `debugModeTrace: bool` - 跟踪模式开关
- `isLoggingEnabled: bool` - 日志启用状态

### 信号
- `willRemoveFromEntity` - 即将从实体移除时发出

### 方法

#### 生命周期方法
```gdscript
func validateParent() -> void
```
验证父节点，在 NOTIFICATION_PARENTED 时调用。

```gdscript
func _enter_tree() -> void
```
进入场景树时调用，处理组件注册。

```gdscript
func findParentEntity(checkGrandparents: bool = false) -> Entity
```
查找父实体，可选择是否检查祖父节点。

```gdscript
func registerEntity(newParentEntity: Entity) -> void
```
注册到指定实体。

```gdscript
func removeFromEntity(shouldFree: bool = true) -> void
```
从实体移除组件，可选择是否释放内存。

```gdscript
func requestDeletion() -> bool
```
请求删除自身，返回是否成功。

```gdscript
func requestDeletionOfParentEntity() -> bool
```
请求删除父实体，返回是否成功。

```gdscript
func unregisterEntity() -> void
```
从实体注销，在 NOTIFICATION_UNPARENTED 时调用。

#### 验证方法
```gdscript
func getRequiredComponents() -> Array[Script]
```
**需要在子类中重写**。返回此组件依赖的其他组件类型列表。

```gdscript
func checkRequiredComponents() -> bool
```
检查是否满足所有依赖组件，返回检查结果。

#### 组件查找方法
```gdscript
func findCoComponent(type: Script, includeSubclasses: bool = true) -> Component
```
查找同级组件，支持子类查找。

```gdscript
func removeSiblingComponentsOfSameType() -> int
```
移除同类型的兄弟组件，返回移除数量。

#### 实用方法
```gdscript
func toggleEnabled(overrideIsEnabled: Variant = null, togglePause: bool = false) -> bool
```
切换启用状态，支持暂停功能。

#### 静态方法
```gdscript
static func castOrFindComponent(node: Node, componentType: GDScript, findInParentEntity: bool = true) -> Component
```
尝试将节点转换为组件或在父实体中查找。

#### 日志方法
```gdscript
func printLog(message: String = "", object: Variant = self.logName) -> void
func printDebug(message: String = "") -> void
func printWarning(message: String = "") -> void
func printError(message: String = "") -> void
func printTrace(...values: Array[Variant]) -> void
func printChange(variableName: String, previousValue: Variant, newValue: Variant, logAsDebug: bool = true) -> void
func emitDebugBubble(textOrObject: Variant, color: Color = self.randomDebugColor, ignoreDebugMode: bool = false) -> void
```

---

## NodeModifierComponentBase

### 概述
用于动态添加、移除组件和节点的基类。

### 属性
- `shouldRemoveEntity: bool` - 是否移除整个实体
- `nodesToRemove: Array[Node]` - 要移除的节点列表
- `componentsToRemove: Array[Script]` - 要移除的组件类型列表
- `componentsToCreate: Array[Script]` - 要创建的组件类型列表
- `payload: Payload` - 可选的载荷对象
- `isEnabled: bool = true` - 是否启用

### 信号
- `willRemoveEntity` - 即将移除实体时发出
- `didAddComponents(components: Array[Component])` - 添加组件后发出

### 方法
```gdscript
func removeEntity() -> void
```
移除父实体。

```gdscript
func removeNodes() -> void
```
移除指定的节点。

```gdscript
func removeComponents() -> void
```
移除指定的组件。

```gdscript
func createComponents(entityOverride: Entity = self.savedParentEntity) -> Array[Component]
```
创建新组件，返回创建的组件数组。

```gdscript
func executePayload(target: Variant) -> void
```
执行载荷逻辑。

```gdscript
func performAllModifications() -> void
```
按顺序执行所有修改操作。

---

## DebugComponent

### 概述
提供调试信息显示和图表功能的组件。

### 属性
- `isEnabled: bool = true` - 是否启用调试显示
- `propertiesToLabel: Array[NodePath]` - 要显示为标签的属性路径
- `shouldHideLabelsUntilHover: bool = false` - 是否在悬停前隐藏标签
- `propertiesToChart: Array[NodePath]` - 要创建图表的属性路径
- `chartVerticalHeight: float = 100` - 图表垂直高度
- `chartValueScale: float = 0.5` - 图表值缩放

### 方法
```gdscript
func convertPathsToAbsolute(relativePaths: Array[NodePath]) -> Array[NodePath]
```
将相对路径转换为绝对路径。

```gdscript
func createCharts() -> void
```
创建属性图表窗口。

```gdscript
func updateLabelsVisibility() -> void
```
更新标签可见性。

```gdscript
func updatePropertiesLabel() -> void
```
更新属性标签内容。

---

## 重要基类

### InputDependentComponentBase
所有依赖输入的组件的基类。

### CharacterBodyDependentComponentBase
依赖 CharacterBody 的组件基类，提供对角色物理体的便捷访问。

### AreaComponentBase
所有区域相关组件的基类，提供区域检测功能。

### ActionTargetingComponentBase
动作瞄准组件的基类，处理目标选择逻辑。

### TimerComponentBase
基于定时器的组件基类，提供定时功能。

---

## 使用示例

### 创建自定义组件
```gdscript
class_name CustomComponent
extends Component

func getRequiredComponents() -> Array[Script]:
    return [HealthComponent, MovementComponent]

func _ready():
    # 访问同级组件
    var healthComp = coComponents.HealthComponent
    var movementComp = coComponents.get(&"MovementComponent")
```

### 使用修改器组件
```gdscript
class_name CustomModifier
extends NodeModifierComponentBase

func _ready():
    # 配置要创建的组件
    componentsToCreate = [DamageComponent, HealthComponent]
    # 执行修改
    performAllModifications()
```

### 调试组件使用
```gdscript
# 在场景中添加 DebugComponent
# 设置要监控的属性
debug_component.propertiesToLabel = [
    "../HealthComponent:health:value",
    "../MovementComponent:velocity"
]
```

---

## 最佳实践

1. **依赖声明**: 总是在 `getRequiredComponents()` 中声明组件依赖
2. **组件通信**: 使用 `coComponents` 字典访问同级组件
3. **错误处理**: 使用 `findCoComponent()` 进行安全的组件查找
4. **调试支持**: 在组件中适当使用调试方法
5. **生命周期**: 正确处理组件的初始化和清理

---

## 注意事项

- 组件应该专注于单一职责
- 避免组件间的循环依赖
- 合理使用调试功能，避免性能影响
- 在生产环境中关闭详细日志 