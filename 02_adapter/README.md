# 适配器模式

适配器模式用于转换一种接口适配另一种接口。

实际使用中 `Adaptee` 一般为接口，并且使用工厂函数生成实例。

在 `Adapter` 中匿名组合 `Adaptee` 接口，所以 `Adapter` 类也拥有 `SpecificRequest` 实例方法，又因为 `Go` 语言中非入侵式接口特征，其实 `Adapter` 也适配 `Adaptee`
接口。

**适配器模式（Adapter Pattern）**，属于结构型设计模式。以下是对代码的解析及实际应用说明：

---

### 一、适配器模式核心概念
**目的**：将不兼容的接口转换为客户端期望的接口，使原本无法协同工作的类可以一起工作，类似电源插头转换器。

**代码中的角色**：
1. **Target（目标接口）**：客户端期望调用的接口，对应代码中的 `Target` 接口（`Request()` 方法）。
2. **Adaptee（被适配者）**：已存在的、功能可用但接口不兼容的类，对应代码中的 `Adaptee` 接口（`SpecificRequest()` 方法）。
3. **Adapter（适配器）**：通过包装 `Adaptee`，将其接口转换为 `Target` 接口，对应代码中的 `adapter` 结构体。

---

### 二、代码结构解析
```go
// 目标接口（客户端期望的接口）
type Target interface {
    Request() string
}

// 被适配的接口（不兼容的旧接口）
type Adaptee interface {
    SpecificRequest() string
}

// 被适配接口的具体实现
type adapteeImpl struct{}
func (*adapteeImpl) SpecificRequest() string {
    return "adaptee method"
}

// 适配器：将Adaptee转换为Target接口
type adapter struct {
    Adaptee  // 通过组合持有被适配对象
}
func (a *adapter) Request() string {
    return a.SpecificRequest()  // 调用被适配者的方法
}

// 使用示例
func main() {
    adaptee := NewAdaptee()          // 创建被适配对象
    target := NewAdapter(adaptee)   // 创建适配器
    fmt.Println(target.Request())   // 输出: adaptee method
}
```

---

### 三、实际应用场景

#### 场景 1：整合第三方支付接口
假设系统需要支持支付宝和微信支付，但它们的接口不一致：
```go
// 支付宝支付接口（Adaptee）
type AlipayAPI interface {
    AlipayPay(amount float64) error  // 不兼容的方法名
}

// 微信支付接口（Adaptee）
type WechatPayAPI interface {
    WechatPay(money int) error       // 不兼容的参数类型
}

// 统一的目标支付接口（Target）
type PaymentTarget interface {
    Pay(amount float64) error
}

// 支付宝适配器
type AlipayAdapter struct {
    AlipayAPI
}
func (a *AlipayAdapter) Pay(amount float64) error {
    return a.AlipayPay(amount)  // 适配调用
}

// 微信支付适配器
type WechatAdapter struct {
    WechatPayAPI
}
func (w *WechatAdapter) Pay(amount float64) error {
    return w.WechatPay(int(amount * 100))  // 转换参数（元→分）
}

// 使用适配器统一调用
func main() {
    alipay := NewAlipay()
    wechat := NewWechatPay()

    // 创建适配器
    payment1 := &AlipayAdapter{alipay}
    payment2 := &WechatAdapter{wechat}

    // 统一调用Pay方法
    payment1.Pay(100.5)
    payment2.Pay(200.8)
}
```

#### 场景 2：兼容旧系统接口
旧系统返回XML数据，新系统需使用JSON：
```go
// 旧系统接口（Adaptee）
type LegacyXMLService interface {
    GetDataXML() string  // 返回XML格式数据
}

// 新系统目标接口（Target）
type JSONService interface {
    GetDataJSON() string
}

// 适配器：将XML转换为JSON
type XMLToJSONAdapter struct {
    LegacyXMLService
}
func (a *XMLToJSONAdapter) GetDataJSON() string {
    xmlData := a.GetDataXML()
    // 假设将XML转换为JSON（伪代码）
    jsonData := ConvertXMLToJSON(xmlData)
    return jsonData
}

// 客户端调用
func main() {
    legacyService := NewLegacyService()
    adapter := &XMLToJSONAdapter{legacyService}
    data := adapter.GetDataJSON()  // 获取JSON数据
}
```

---

### 四、适配器模式优势
| 优势                | 说明                                                                 |
|---------------------|----------------------------------------------------------------------|
| **接口兼容**         | 解决新旧系统或第三方库接口不兼容问题                                  |
| **复用现有代码**     | 无需修改原有代码即可重用功能                                           |
| **解耦客户端与实现** | 客户端只依赖目标接口，不直接耦合具体实现                               |

---

### 五、适配器 vs 外观模式
| 模式          | 目标                          | 特点                                                                 |
|---------------|-------------------------------|----------------------------------------------------------------------|
| **适配器**    | **接口转换**                  | 解决两个已有接口不兼容问题，通常1:1适配                              |
| **外观**      | **简化复杂调用**              | 封装多个子系统接口，提供更高层的统一入口（1:N关系）                  |

---

### 六、扩展建议
1. **支持多个Adaptee**
   适配器可以组合多个被适配对象：
   ```go
   type MultiAdapter struct {
       AlipayAPI
       WechatPayAPI
   }
   func (m *MultiAdapter) Pay(amount float64) error {
       // 根据策略选择调用哪个支付
   }
   ```

2. **链式适配**
   多层适配器解决复杂转换：
   ```go
   // 旧系统返回XML → 转换为JSON → 再转换为Protobuf
   jsonAdapter := &XMLToJSONAdapter{legacyService}
   protobufAdapter := &JSONToProtobufAdapter{jsonAdapter}
   ```

---

### 总结
适配器模式是整合不兼容接口的桥梁，在系统升级、多平台兼容、第三方服务集成等场景中不可或缺。通过灵活使用适配器，可以最大化复用现有代码，降低系统耦合度。
