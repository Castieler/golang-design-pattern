# 抽象工厂模式

抽象工厂模式用于生成产品族的工厂，所生成的对象是有关联的。

如果抽象工厂退化成生成的对象无关联则成为工厂函数模式。

比如本例子中使用 `RDB` 和 `XML` 存储订单信息，抽象工厂分别能生成相关的主订单信息和订单详情信息。 如果业务逻辑中需要替换使用的时候只需要改动工厂函数相关的类就能替换使用不同的存储方式了。

抽象工厂模式（Abstract Factory Pattern）通过**封装多类产品的创建逻辑**、**保证产品族一致性**、**屏蔽底层实现细节**三大核心能力，实现**复杂对象家族的统一创建**。以下结合代码详细解析：

---

### **一、模式结构解析**
#### 1. **核心角色定义**
```go
// 抽象产品1：订单主记录操作接口
type OrderMainDAO interface {
    SaveOrderMain()
}

// 抽象产品2：订单详情操作接口
type OrderDetailDAO interface {
    SaveOrderDetail()
}

// 抽象工厂接口（核心）
type DAOFactory interface {
    CreateOrderMainDAO() OrderMainDAO  // 创建产品1
    CreateOrderDetailDAO() OrderDetailDAO // 创建产品2
}
```
- **产品族**：`OrderMainDAO` + `OrderDetailDAO` 构成一个完整订单存储方案
- **工厂职责**：每个具体工厂必须同时实现两种产品的创建方法

#### 2. **具体产品实现**
```go
// RDB产品族实现
type RDBMainDAO struct{}    // 关系型数据库主记录存储
func (*RDBMainDAO) SaveOrderMain() { fmt.Print("rdb main save\n") }

type RDBDetailDAO struct{}  // 关系型数据库详情存储
func (*RDBDetailDAO) SaveOrderDetail() { fmt.Print("rdb detail save\n") }

// XML产品族实现
type XMLMainDAO struct{}    // XML文件主记录存储
func (*XMLMainDAO) SaveOrderMain() { fmt.Print("xml main save\n") }

type XMLDetailDAO struct{}  // XML文件详情存储
func (*XMLDetailDAO) SaveOrderDetail() { fmt.Print("xml detail save") }
```

#### 3. **具体工厂实现**
```go
// RDB工厂（生产同一技术栈的产品）
type RDBDAOFactory struct{}
func (*RDBDAOFactory) CreateOrderMainDAO() OrderMainDAO {
    return &RDBMainDAO{} // 返回RDB主记录操作实例
}
func (*RDBDAOFactory) CreateOrderDetailDAO() OrderDetailDAO {
    return &RDBDetailDAO{} // 返回RDB详情操作实例
}

// XML工厂（生产另一技术栈的产品）
type XMLDAOFactory struct{}
func (*XMLDAOFactory) CreateOrderMainDAO() OrderMainDAO {
    return &XMLMainDAO{}
}
func (*XMLDAOFactory) CreateOrderDetailDAO() OrderDetailDAO {
    return &XMLDetailDAO{}
}
```

---

### **二、模式核心价值**
#### 1. **强制产品族一致性**
```go
// 客户端代码示例
func SaveOrder(factory DAOFactory) {
    mainDAO := factory.CreateOrderMainDAO()
    detailDAO := factory.CreateOrderDetailDAO()
    
    mainDAO.SaveOrderMain()   // 主存储方式
    detailDAO.SaveOrderDetail() // 配套的详情存储方式
}

// 使用RDB工厂（确保主从存储均为RDB实现）
SaveOrder(&RDBDAOFactory{}) 
// 输出：
// rdb main save
// rdb detail save

// 使用XML工厂（确保主从存储均为XML实现）
SaveOrder(&XMLDAOFactory{})
// 输出：
// xml main save
// xml detail save
```
- **防止混用不同技术栈**：通过工厂约束，不会出现主记录用RDB而详情用XML的错误组合

#### 2. **切换存储方案的便捷性**
```go
// 只需修改工厂参数即可切换整套方案
var config = "rdb" // 可配置为 "xml"

func GetFactory() DAOFactory {
    switch config {
    case "rdb":
        return &RDBDAOFactory{}
    case "xml":
        return &XMLDAOFactory{}
    default:
        panic("unsupported storage type")
    }
}

// 所有业务代码无需修改
SaveOrder(GetFactory())
```

#### 3. **新增产品族的扩展性**
```go
// 新增JSON存储方案只需三步：
// 1. 实现JSON产品
type JSONMainDAO struct{}
func (*JSONMainDAO) SaveOrderMain() { fmt.Print("json main save") }

type JSONDetailDAO struct{}
func (*JSONDetailDAO) SaveOrderDetail() { fmt.Print("json detail save") }

// 2. 实现JSON工厂
type JSONDAOFactory struct{}
func (JSONDAOFactory) CreateOrderMainDAO() OrderMainDAO { return &JSONMainDAO{} }
func (JSONDAOFactory) CreateOrderDetailDAO() OrderDetailDAO { return &JSONDetailDAO{} }

// 3. 修改配置
config = "json"
SaveOrder(&JSONDAOFactory{})
```
- **符合开闭原则**：扩展时无需修改已有工厂和产品代码

---

### **三、与工厂方法模式的关键差异**
| 维度             | 工厂方法模式                     | 抽象工厂模式                     |
|------------------|----------------------------------|----------------------------------|
| **核心目标**     | 创建单一产品                     | 创建相关产品族                   |
| **接口复杂度**   | 工厂接口只含1个创建方法          | 工厂接口包含多个创建方法          |
| **应用场景**     | 需要灵活扩展单个产品类型         | 需要约束多个产品的配套使用        |
| **代码示例**     | 加法/减法运算符工厂              | RDB/XML整套存储方案工厂           |

---

### **四、Go语言实现特点**
#### 1. **隐式接口实现**
```go
// 无需显式声明实现关系，只要方法匹配即视为实现
type RDBDAOFactory struct{} 
// 自动满足DAOFactory接口要求
```
- 减少样板代码，新增产品族时更灵活

#### 2. **结构体嵌套（Embedding）的替代方案**
```go
// 若多个产品有公共逻辑，可通过组合复用
type BaseDAO struct { /* 公共字段和方法 */ }

type RDBMainDAO struct {
    BaseDAO // 嵌套公共结构
    // 特有实现
}
```

---

### **五、典型应用场景**
1. **跨平台UI组件库**
   - 抽象工厂：`GUIFactory`
   - 具体工厂：`WindowsFactory` / `MacFactory`
   - 产品族：`Button` + `Menu` + `Dialog`

2. **数据库访问套件**
   - 抽象工厂：`DBFactory`
   - 具体工厂：`MySQLFactory` / `PostgreSQLFactory`
   - 产品族：`Connection` + `Command` + `Transaction`

3. **游戏角色装备系统**
   - 抽象工厂：`EquipmentFactory`
   - 具体工厂：`WarriorFactory` / `MageFactory`
   - 产品族：`Weapon` + `Armor` + `Accessory`

---

### **总结**
在示例代码中，抽象工厂模式通过：
1. **`DAOFactory` 接口** 统一约束存储方案的创建规则
2. **`RDBDAOFactory`/`XMLDAOFactory`** 保证同一技术栈产品的配套生成
3. **接口隔离** 使客户端代码仅依赖抽象层

这种模式特别适合需要**多维度扩展**（如同时支持不同数据库类型、不同文件格式）且**产品间存在强关联**的系统设计。在微服务架构中，常见于多数据源切换、多协议适配等场景。
