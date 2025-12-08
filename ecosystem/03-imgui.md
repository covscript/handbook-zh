# 3.3 covscript-imgui - GUI 编程

`covscript-imgui` 是基于 Dear ImGui 的图形用户界面库，适合创建工具和调试界面。

### 安装

```bash
cspkg install imgui
```

### 基本窗口

```covscript
import imgui
import imgui_font
using imgui

# 创建窗口应用
var app = window_application(800, 600, "CovScript ImGui 应用")

# 添加中文字体支持
var font = add_font_extend_cn(imgui_font.source_han_sans, 32)
set_font_scale(0.5)

var window_opened = true

# 主循环
while !app.is_closed()
    app.prepare()
    
    push_font(font)
    
    # 开始窗口
    begin_window("Hello 窗口", window_opened, {flags.always_auto_resize})
        if !window_opened
            break
        end
        
        text("欢迎使用 CovScript ImGui!")
        separator()
        
        if button("点击我")
            system.out.println("按钮被点击!")
        end
    end_window()
    
    pop_font()
    window_opened = true
    app.render()
end
```

### 交互控件

```covscript
import imgui
import imgui_font
using imgui

var app = window_application(800, 600, "控件示例")
var font = add_font_extend_cn(imgui_font.source_han_sans, 32)
set_font_scale(0.5)

# 状态变量
var progress = 0.0
var radio_choice = 0
var combo_choice = 1
var texts = new string
var window_opened = true

while !app.is_closed()
    app.prepare()
    push_font(font)
    
    begin_window("控件演示", window_opened, {flags.always_auto_resize})
        if !window_opened
            break
        end
        
        # 滑动条和进度条
        slider_float("进度条", progress, 0, 1)
        progress_bar(progress, "进度")
        separator()
        
        # 折线图
        plot_lines("折线图", "", {1, 3, 2, 3, 1})
        separator()
        
        # 单选按钮
        radio_button("选项 A", radio_choice, 0)
        same_line()
        radio_button("选项 B", radio_choice, 1)
        same_line()
        radio_button("选项 C", radio_choice, 2)
        
        switch radio_choice
            case 0
                text("你选择了 A")
            end
            case 1
                text("你选择了 B")
            end
            case 2
                text("你选择了 C")
            end
        end
        separator()
        
        # 下拉框
        combo_box("选择主题", combo_choice, {"经典", "亮色", "暗色"})
        switch combo_choice
            case 0
                style_color_classic()
            end
            case 1
                style_color_light()
            end
            case 2
                style_color_dark()
            end
        end
        separator()
        
        # 多行文本输入
        input_text_multiline_s("##input", texts, 512, {flags.allow_tab})
        
    end_window()
    
    pop_font()
    window_opened = true
    app.render()
end
```

### 窗口属性和布局

```covscript
import imgui
import imgui_font
using imgui

var app = window_application(0.75 * get_monitor_width(0), 
                             0.75 * get_monitor_height(0), 
                             "布局示例")
var font = add_font_extend_cn(imgui_font.source_han_sans, 32)
set_font_scale(0.5)

var window_opened = true

while !app.is_closed()
    app.prepare()
    push_font(font)
    
    begin_window("主窗口", window_opened, {flags.always_auto_resize})
        if !window_opened
            break
        end
        
        text("窗口位置和大小演示")
        separator()
        
        # 获取窗口属性
        var pos = {get_window_pos_x(), get_window_pos_y()}
        var size = {get_window_width(), get_window_height()}
        
        text("位置: (" + to_string(pos[0]) + ", " + to_string(pos[1]) + ")")
        text("大小: " + to_string(size[0]) + " x " + to_string(size[1]))
        
    end_window()
    
    # 创建浮动窗口
    begin_window("浮动窗口", window_opened, {flags.no_title_bar, flags.no_move})
        set_window_pos(vec2(pos[0] + size[0] + 10, pos[1]))
        set_window_size(vec2(200, 100))
        text("这是一个固定位置的窗口")
    end_window()
    
    pop_font()
    window_opened = true
    app.render()
end
```

### 实际应用：简单文本编辑器

```covscript
import imgui
import imgui_font
using imgui

var app = window_application(900, 600, "文本编辑器")
var font = add_font_extend_cn(imgui_font.source_han_sans, 24)
set_font_scale(0.6)

var content = new string
var filename = "untitled.txt"
var window_opened = true

function save_file(path, text)
    var file = iostream.fstream(path, iostream.openmode.out)
    file.print(text)
    file.close()
    system.out.println("文件已保存: " + path)
end

function load_file(path)
    try
        var file = iostream.fstream(path, iostream.openmode.in)
        var text = new string
        loop
            var line = file.getline()
            if file.eof()
                break
            end
            text += line + "\n"
        end
        file.close()
        return text
    catch e
        system.out.println("无法读取文件: " + path)
        return ""
    end
end

while !app.is_closed()
    app.prepare()
    push_font(font)
    
    begin_window("编辑器", window_opened, {})
        if !window_opened
            break
        end
        
        text("文件: " + filename)
        separator()
        
        # 菜单按钮
        if button("保存")
            save_file(filename, content)
        end
        same_line()
        if button("另存为...")
            # 实际应用中这里应该有文件对话框
            save_file("output.txt", content)
        end
        same_line()
        if button("加载")
            # 实际应用中这里应该有文件对话框
            if system.path.exist(filename)
                content = load_file(filename)
            end
        end
        
        separator()
        
        # 文本编辑区域
        input_text_multiline_s("##editor", content, 4096, 
                              {flags.allow_tab})
        
        separator()
        text("字符数: " + to_string(content.size))
        
    end_window()
    
    pop_font()
    window_opened = true
    app.render()
end
```

### 获取显示器信息

```covscript
import imgui
using imgui

# 获取主显示器信息
var monitor_count = get_monitor_count()
system.out.println("显示器数量: " + to_string(monitor_count))

for i = 0, i < monitor_count, ++i
    var width = get_monitor_width(i)
    var height = get_monitor_height(i)
    system.out.println("显示器 " + to_string(i) + ": " + 
                      to_string(width) + " x " + to_string(height))
end
```

