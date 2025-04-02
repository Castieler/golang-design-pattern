# 工厂方法模式

工厂方法模式（Factory Method Pattern）在代码中通过**解耦对象创建与使用逻辑**、**支持扩展新类型**、**规范对象构造流程**三大核心作用，提升代码的灵活性和可维护性。以下是结合代码的详细解析：

---

### **一、核心作用解析**
#### 1. **解耦客户端与具体实现类**
```go
// 客户端代码示例（不依赖具体实现类）
func Calculate(factory OperatorFactory, a, b int) int {
    op := factory.Create() // 通过接口创建对象
    op.SetA(a)
    op.SetB(b)
    return op.Result()
}

// 使用加法工厂
result := Calculate(PlusOperatorFactory{}, 5, 3) // 输出8

// 使用减法工厂
result := Calculate(MinusOperatorFactory{}, 5, 3) // 输出2
```
- **客户端仅依赖接口**：`Calculate` 函数只与 `OperatorFactory` 和 `Operator` 接口交互，无需知道 `PlusOperator` 或 `MinusOperator` 的存在。
- **切换实现无需修改客户端**：新增乘除运算时，只需添加新工厂和运算符类，无需改动 `Calculate` 函数。

#### 2. **统一对象创建规范**
```go
// 工厂接口强制实现类必须提供 Create 方法
type OperatorFactory interface {
    Create() Operator
}

// 所有具体工厂必须实现 Create
type PlusOperatorFactory struct{}
func (PlusOperatorFactory) Create() Operator { /*...*/ }

type MinusOperatorFactory struct{}
func (MinusOperatorFactory) Create() Operator { /*...*/ }
```
- **强制创建逻辑标准化**：通过接口约束，所有具体工厂必须实现相同的创建方法，避免因开发人员随意创建对象导致的代码混乱。
- **隐藏构造细节**：`Create()` 方法封装了 `OperatorBase` 的初始化过程（如内存分配），客户端无需关心如何构造对象。

#### 3. **支持灵活扩展**
```go
// 新增乘法运算只需两步：
// 1. 定义乘法运算符
type MultiplyOperator struct { *OperatorBase }
func (o MultiplyOperator) Result() int { return o.a * o.b }

// 2. 定义乘法工厂
type MultiplyOperatorFactory struct{}
func (MultiplyOperatorFactory) Create() Operator {
    return &MultiplyOperator{OperatorBase: &OperatorBase{}}
}

// 客户端代码无需修改即可使用
result := Calculate(MultiplyOperatorFactory{}, 5, 3) // 输出15
```
- **符合开闭原则**（Open-Closed Principle）：扩展新功能时**不修改已有代码**，仅通过新增类实现扩展。
- **降低系统耦合**：新增功能对原有系统的侵入性趋近于零。

---

### **二、模式结构对比**
#### **没有工厂方法的问题**
```go
// 直接依赖具体类的客户端代码
func Calculate(opType string, a, b int) int {
    var op Operator
    switch opType {
    case "plus":
        op = &PlusOperator{OperatorBase: &OperatorBase{}}
    case "minus":
        op = &MinusOperator{OperatorBase: &OperatorBase{}}
    // 每新增一种运算，都要修改此处代码
    }
    op.SetA(a)
    op.SetB(b)
    return op.Result()
}
```
- **违反开闭原则**：每次新增运算符都需修改 `switch` 逻辑。
- **客户端与具体类强耦合**：直接引用 `PlusOperator` 等具体类型。

#### **引入工厂方法的优势**
| 场景               | 耦合度 | 扩展成本 | 代码可读性 |
|--------------------|--------|----------|------------|
| 直接实例化对象     | 高     | 高       | 低（业务与创建逻辑混合） |
| 使用工厂方法       | 低     | 低       | 高（创建逻辑集中管理） |

---

### **三、适用场景分析**
#### 1. **需要动态选择实现类的场景**
```go
// 根据配置文件决定使用哪种运算
config := LoadConfig() // 返回 "multiply"
var factory OperatorFactory
switch config.OperatorType {
case "plus":
    factory = PlusOperatorFactory{}
case "minus":
    factory = MinusOperatorFactory{}
case "multiply":
    factory = MultiplyOperatorFactory{}
}
result := Calculate(factory, 5, 3)
```
- **运行时动态绑定**：通过配置、用户输入或环境变量动态选择工厂。

#### 2. **对象创建过程复杂的场景**
```go
// 假设 Operator 需要连接数据库初始化
func (f ComplexOperatorFactory) Create() Operator {
    conn := ConnectDatabase()      // 复杂初始化步骤
    validator := NewValidator(conn)
    return &ComplexOperator{
        OperatorBase: &OperatorBase{},
        conn: conn,
        validator: validator,
    }
}
```
- **封装复杂初始化**：将数据库连接、依赖注入等逻辑隐藏在工厂中。

#### 3. **需要对象复用或池化的场景**
```go
// 实现带对象池的工厂
type PooledPlusOperatorFactory struct {
    pool sync.Pool
}

func (f *PooledPlusOperatorFactory) Create() Operator {
    return f.pool.Get().(*PlusOperator)
}

func (f *PooledPlusOperatorFactory) Release(op Operator) {
    f.pool.Put(op)
}
```
- **资源复用**：通过工厂管理对象池，提升性能。

---

### **四、Go 语言的实现特点**
1. **隐式接口实现**
   ```go
   // 无需显式声明实现接口
   type PlusOperatorFactory struct{}
   func (PlusOperatorFactory) Create() Operator { /*...*/ } // 自动满足 OperatorFactory
   ```
   - Go 的接口是 **Duck Typing** 模式，减少模板代码。

2. **结构体嵌套（Embedding）**
   ```go
   type PlusOperator struct {
       *OperatorBase // 复用基类字段和方法
   }
   ```
   - 通过嵌套 `OperatorBase` 实现代码复用，类似继承的效果。

---

### **总结**
工厂方法模式在示例代码中通过 **接口隔离** 和 **多态创建** 实现了：
1. 客户端代码与具体实现类的解耦
2. 新增运算符类型的零侵入扩展
3. 对象创建流程的集中管控

在 Go 语言中，这种模式常被用于依赖注入框架、数据库驱动切换（如 `sql.DB` 抽象）等场景，是构建高扩展性架构的基础模式之一。
