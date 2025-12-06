# 3.7 picasso-ui - 现代化图形界面库

Picasso UI 是 CovScript 的现代化图形用户界面框架，提供了声明式 UI 构建方式。

### 安装

```bash
cspkg install picasso
```

### 基本窗口

```covscript
import picasso

# 创建应用
var app = picasso.Application()

# 创建主窗口
var window = picasso.Window()
window.setTitle("Picasso UI 示例")
window.setSize(800, 600)

# 添加组件
var layout = picasso.VBoxLayout()

var label = picasso.Label("欢迎使用 Picasso UI")
label.setFontSize(24)
layout.add(label)

var button = picasso.Button("点击我")
button.onClick([]() {
    system.out.println("按钮被点击！")
})
layout.add(button)

window.setLayout(layout)
window.show()

# 运行应用
app.exec()
```

### 声明式 UI 构建

```covscript
import picasso

class TodoApp
    var items = new list
    var window = null
    var listView = null
    
    function construct()
        this.window = picasso.Window()
        this.window.setTitle("待办事项")
        this.window.setSize(400, 600)
        
        this.buildUI()
    end
    
    function buildUI()
        var layout = picasso.VBoxLayout()
        
        # 标题
        var title = picasso.Label("我的待办事项")
        title.setFontSize(20)
        title.setAlignment(picasso.Align.Center)
        layout.add(title)
        
        # 输入框和添加按钮
        var inputLayout = picasso.HBoxLayout()
        var input = picasso.TextEdit()
        input.setPlaceholder("输入新任务...")
        inputLayout.add(input)
        
        var addBtn = picasso.Button("添加")
        addBtn.onClick([](app, inp) {
            var text = inp.getText()
            if text.size > 0
                app.addItem(text)
                inp.clear()
            end
        }, this, input)
        inputLayout.add(addBtn)
        
        layout.add(inputLayout)
        
        # 任务列表
        this.listView = picasso.ListView()
        layout.add(this.listView)
        
        # 统计信息
        var statsLabel = picasso.Label("总计: 0 项")
        layout.add(statsLabel)
        
        this.window.setLayout(layout)
    end
    
    function addItem(text)
        this.items.push_back({
            "text": text,
            "completed": false
        })
        this.updateList()
    end
    
    function updateList()
        this.listView.clear()
        
        foreach item in this.items
            var itemWidget = picasso.HBoxLayout()
            
            var checkbox = picasso.CheckBox(item["text"])
            checkbox.setChecked(item["completed"])
            itemWidget.add(checkbox)
            
            var deleteBtn = picasso.Button("删除")
            itemWidget.add(deleteBtn)
            
            this.listView.addItem(itemWidget)
        end
    end
    
    function show()
        this.window.show()
    end
end

# 运行应用
var app = picasso.Application()
var todoApp = new TodoApp{}
todoApp.show()
app.exec()
```

### 响应式布局

```covscript
import picasso

# 创建响应式布局
var window = picasso.Window()
window.setTitle("响应式布局")

var layout = picasso.GridLayout(3, 3)  # 3x3 网格

# 添加按钮到网格
for i = 0, i < 9, ++i
    var btn = picasso.Button("按钮 " + to_string(i + 1))
    btn.onClick([](index) {
        system.out.println("点击了按钮 " + to_string(index))
    }, i + 1)
    layout.add(btn, i / 3, i % 3)
end

window.setLayout(layout)
window.show()

var app = picasso.Application()
app.exec()
```

### 自定义样式

```covscript
import picasso

# 设置应用主题
picasso.setTheme("dark")  # 或 "light"

var window = picasso.Window()
window.setTitle("自定义样式")

var button = picasso.Button("样式化按钮")

# 设置自定义样式
button.setStyle({
    "background-color": "#4CAF50",
    "color": "white",
    "border-radius": "5px",
    "padding": "10px 20px",
    "font-size": "16px"
})

# 悬停效果
button.onHover([]() {
    button.setStyle({"background-color": "#45a049"})
}, []() {
    button.setStyle({"background-color": "#4CAF50"})
})

window.add(button)
window.show()

var app = picasso.Application()
app.exec()
```

