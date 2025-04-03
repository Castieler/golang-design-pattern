**生成器模式（Builder Pattern）**，它通过分离对象的构建过程和表示，使得相同的构建步骤可以生成不同的结果。以下是详细解析和实际应用示例：

---

### 一、代码结构解析
1. **`Builder` 接口**
   定义了构建对象的步骤（`Part1`、`Part2`、`Part3`），但不涉及具体实现。
   ```go
   type Builder interface {
       Part1()
       Part2()
       Part3()
   }
   ```

2. **`Director` 指导者**
   控制构建流程，按固定顺序调用 `Builder` 的方法。
   ```go
   type Director struct {
       builder Builder
   }

   func (d *Director) Construct() {
       d.builder.Part1()
       d.builder.Part2()
       d.builder.Part3()
   }
   ```

3. **具体生成器（`Builder1` 和 `Builder2`）**
   分别实现 `Builder` 接口，生成不同类型的结果（字符串和整数）。
   ```go
   // Builder1 生成字符串 "123"
   type Builder1 struct { result string }
   func (b *Builder1) Part1() { b.result += "1" }
   func (b *Builder1) Part2() { b.result += "2" }
   func (b *Builder1) Part3() { b.result += "3" }
   func (b *Builder1) GetResult() string { return b.result }

   // Builder2 生成整数 6（1+2+3）
   type Builder2 struct { result int }
   func (b *Builder2) Part1() { b.result += 1 }
   func (b *Builder2) Part2() { b.result += 2 }
   func (b *Builder2) Part3() { b.result += 3 }
   func (b *Builder2) GetResult() int { return b.result }
   ```

---

### 二、使用场景示例
假设你需要构建不同格式的报告（如文本格式和数值统计格式），通过生成器模式可以轻松实现：

#### 1. 定义具体生成器
```go
// 文本报告生成器
type TextReportBuilder struct {
    content strings.Builder
}

func (b *TextReportBuilder) Part1() {
    b.content.WriteString("Header: Report\n")
}

func (b *TextReportBuilder) Part2() {
    b.content.WriteString("Body: Data processed\n")
}

func (b *TextReportBuilder) Part3() {
    b.content.WriteString("Footer: End\n")
}

func (b *TextReportBuilder) GetResult() string {
    return b.content.String()
}

// 数值统计生成器
type StatsReportBuilder struct {
    total int
}

func (b *StatsReportBuilder) Part1() { b.total += 100 } // 初始数据
func (b *StatsReportBuilder) Part2() { b.total *= 2 }    // 处理翻倍
func (b *StatsReportBuilder) Part3() { b.total -= 50 }   // 扣除成本
func (b *StatsReportBuilder) GetResult() int { return b.total }
```

#### 2. 使用指导者生成报告
```go
func main() {
    // 生成文本报告
    textBuilder := &TextReportBuilder{}
    director := NewDirector(textBuilder)
    director.Construct()
    fmt.Println(textBuilder.GetResult())
    // 输出：
    // Header: Report
    // Body: Data processed
    // Footer: End

    // 生成数值统计报告
    statsBuilder := &StatsReportBuilder{}
    director = NewDirector(statsBuilder)
    director.Construct()
    fmt.Println(statsBuilder.GetResult()) // 输出：150（100 → 200 → 150）
}
```

---

### 三、生成器模式的核心优势
1. **流程与实现解耦**
   `Director` 只关心构建流程，不依赖具体实现。新增生成器时（如 `PDFReportBuilder`），只需实现 `Builder` 接口，无需修改 `Director`。

2. **灵活生成不同结果**
   同样的 `Construct()` 流程可以生成文本、数值、二进制等不同形态的结果。

3. **避免参数爆炸**
   替代多参数构造函数，例如：
   ```go
   // 反例：难以维护的构造函数
   NewReport(header string, body string, footer string, total int, format string) 
   ```

---

### 四、适用场景
1. **分步骤构建复杂对象**
   如 SQL 查询生成器、HTTP 请求配置器、游戏角色创建工具。

2. **生成不同格式的输出**
   如将同一数据导出为 JSON、XML、CSV 等格式。

3. **隐藏复杂对象的创建细节**
   例如配置一个包含缓存、日志、数据库连接的服务器对象。

---

### 五、生成器 vs 工厂模式
| 模式         | 关注点               | 使用场景                     |
|-------------|---------------------|----------------------------|
| **生成器**  | 分步骤构建复杂对象    | 对象需要多个步骤组合，且结果多样化 |
| **工厂**    | 直接创建完整对象      | 快速创建单一类型的对象          |

---

通过生成器模式，你可以像搭积木一样灵活构建对象，尤其适合需要精细化控制创建过程的场景。
