# 外观模式

`API` 为 `facade` 模块的外观接口，大部分代码使用此接口简化对 `facade` 类的访问。

`facade` 模块同时暴露了 `a` 和 `b` 两个 `Module` 的 `NewXXX` 和 `interface`，其它代码如果需要使用细节功能时可以直接调用。

**外观模式（Facade Pattern）**，属于结构型设计模式。以下是逐层解析和实际应用说明：

---

### 一、模式解析
外观模式的核心思想是**为复杂子系统提供统一的高层接口**，隐藏子系统的实现细节，让调用方只需关注最顶层的接口，降低系统耦合度。

#### 代码中的角色划分：
1. **Facade（外观接口 `API`）**
   - 定义了一个简化入口（`Test()`方法），隐藏子系统内部的复杂性。
   - 示例中的 `apiImpl` 实现了该接口。

2. **Subsystems（子系统 `AModuleAPI` 和 `BModuleAPI`）**
   - 代表独立的子系统功能模块（如 `TestA()` 和 `TestB()`）。
   - 子系统之间可能互相依赖或独立，但对外暴露统一的接口。

3. **Facade Implementation（外观实现 `apiImpl`）**
   - 持有子系统的实例（`a` 和 `b`）。
   - 协调子系统的调用顺序（例如先调用 `TestA()` 再调用 `TestB()`），并组合结果。

---

### 二、使用场景
外观模式适用于以下场景：

#### 1. **简化复杂系统的调用**
当系统有多个模块（如支付系统中的订单、库存、支付网关），调用方需要依次调用多个接口时，外观模式可以封装这些步骤：
```go
// 支付系统外观
type PaymentFacade struct {
    order  OrderSubsystem
    stock  StockSubsystem
    payment PaymentSubsystem
}

func (p *PaymentFacade) Pay(userID string, itemID int) error {
    if err := p.order.CreateOrder(userID, itemID); err != nil {
        return err
    }
    if err := p.stock.ReduceStock(itemID); err != nil {
        return err
    }
    if err := p.payment.Charge(userID); err != nil {
        return err
    }
    return nil
}

// 调用方只需调用外观接口
facade := NewPaymentFacade()
facade.Pay("user123", 1001) // 隐藏了订单、库存、支付的复杂交互
```

#### 2. **统一第三方服务接口**
对接不同云服务商（如阿里云、AWS）时，可以用外观模式屏蔽底层差异：
```go
// 统一云存储接口
type CloudStorageFacade interface {
    UploadFile(file []byte) (url string, err error)
}

// 阿里云实现
type AliyunFacade struct { /*...*/ }
func (a *AliyunFacade) UploadFile(file []byte) (string, error) {
    // 调用阿里云OSS的复杂SDK接口...
}

// AWS实现
type AWSFacade struct { /*...*/ }
func (a *AWSFacade) UploadFile(file []byte) (string, error) {
    // 调用AWS S3的复杂SDK接口...
}

// 调用方无需关心具体实现
var storage CloudStorageFacade = NewAliyunFacade()
url, _ := storage.UploadFile([]byte("data"))
```

#### 3. **分层设计**
在Web服务中，可以用外观模式封装控制器、服务层、DAO层的调用：
```go
// UserFacade 封装用户相关操作
type UserFacade struct {
    service UserService
    cache   CacheService
}

func (u *UserFacade) GetUser(id int) (*User, error) {
    // 1. 查缓存
    if user := u.cache.GetUser(id); user != nil {
        return user, nil
    }
    // 2. 查数据库
    user, err := u.service.GetUser(id)
    if err != nil {
        return nil, err
    }
    // 3. 写缓存
    u.cache.SetUser(id, user)
    return user, nil
}
```

---

### 三、代码示例分析
在提供的代码中：
- **子系统**：`AModuleAPI` 和 `BModuleAPI` 是两个独立模块，各自实现 `TestA()` 和 `TestB()`。
- **外观接口**：`API` 定义了高层接口 `Test()`。
- **外观实现**：`apiImpl` 的 `Test()` 方法协调两个子系统的调用，并返回组合结果。
- **调用方**：只需通过 `NewAPI().Test()` 即可完成多个子系统的操作，无需了解内部细节。

执行效果：
```go
facade := NewAPI()
result := facade.Test() 
// 输出：
// A module running
// B module running
```

---

### 四、外观模式的优势
| 优势                | 说明                                                                 |
|---------------------|----------------------------------------------------------------------|
| **简化调用**         | 调用方只需与外观接口交互，无需理解多个子系统的细节                     |
| **解耦**            | 子系统和调用方之间通过外观接口通信，降低依赖                          |
| **可维护性**         | 子系统内部修改时，只要外观接口不变，调用方无需改动                    |
| **分层清晰**         | 明确划分系统层次，外观层负责协调，子系统专注于自身逻辑                |

---

### 五、与其他模式对比
| 模式          | 目标                          | 特点                                                                 |
|---------------|-------------------------------|----------------------------------------------------------------------|
| **外观模式**  | 简化复杂系统调用              | 提供统一入口，隐藏子系统细节                                         |
| **适配器模式**| 转换接口兼容性                | 解决接口不兼容问题（例如旧系统改造）                                  |
| **中介者模式**| 减少对象间直接通信            | 通过中介者协调多个对象交互（例如聊天室中用户不直接发消息，而是通过服务器） |

---

### 六、扩展性建议
1. **动态切换子系统**
   通过依赖注入，允许运行时更换子系统实现：
   ```go
   func NewCustomAPI(a AModuleAPI, b BModuleAPI) API {
       return &apiImpl{a: a, b: b}
   }
   ```
2. **增加错误处理**
   在外观层统一处理子系统的错误：
   ```go
   func (a *apiImpl) Test() (string, error) {
       if aRet, err := a.a.TestA(); err != nil {
           return "", fmt.Errorf("A模块失败: %v", err)
       }
       if bRet, err := a.b.TestB(); err != nil {
           return "", fmt.Errorf("B模块失败: %v", err)
       }
       return fmt.Sprintf("%s\n%s", aRet, bRet), nil
   }
   ```

---

### 总结
外观模式通过封装复杂子系统的调用细节，提供简洁的高层接口，是降低系统耦合度、提升可维护性的利器。在微服务架构、第三方服务集成、分层设计等场景中广泛应用。
