# 3.9 argparse - 命令行参数解析

`argparse` 是一个功能强大的命令行参数解析库，可以轻松处理复杂的命令行选项。

### 基本使用

```covscript
import argparse

# 创建解析器
var parser = new argparse.ArgumentParser
parser.program_name = "myapp"
parser.description = "这是一个示例程序"

# 添加位置参数
parser.add_argument("input", true, "输入文件路径")
parser.add_argument("output", false, "输出文件路径")

# 添加选项
parser.add_option("--verbose", true, false, "显示详细信息")
parser.add_option("--format", false, true, "输出格式")

# 设置别名
parser.set_option_alias("--verbose", "-v")
parser.set_option_alias("--format", "-f")

# 设置默认值
parser.set_defaults("output", "output.txt")
parser.set_defaults("--format", "json")

# 解析命令行参数
try
    var args = parser.parse_args(context.cmd_args)
    
    system.out.println("输入文件: " + args.input)
    system.out.println("输出文件: " + args.output)
    system.out.println("详细模式: " + to_string(args.verbose))
    system.out.println("输出格式: " + args.format)
    
catch e
    system.out.println("参数解析错误: " + e.what())
    parser.print_help()
    system.exit(1)
end
```

### 完整示例：文件处理工具

```covscript
import argparse

class FileProcessor
    var verbose = false
    var format = "text"
    
    function process(input_file, output_file)
        if this.verbose
            system.out.println("处理文件: " + input_file)
        end
        
        # 读取输入文件
        var in_file = iostream.fstream(input_file, iostream.openmode.in)
        var content = new string
        
        loop
            var line = in_file.getline()
            if in_file.eof()
                break
            end
            content += line + "\n"
        end
        in_file.close()
        
        if this.verbose
            system.out.println("读取了 " + to_string(content.size) + " 字节")
        end
        
        # 根据格式处理
        var processed = null
        switch this.format
            case "upper"
                processed = content.toupper()
            end
            case "lower"
                processed = content.tolower()
            end
            default
                processed = content
            end
        end
        
        # 写入输出文件
        var out_file = iostream.fstream(output_file, iostream.openmode.out)
        out_file.print(processed)
        out_file.close()
        
        if this.verbose
            system.out.println("输出到: " + output_file)
        end
    end
end

# 主程序
function main(args)
    var parser = new argparse.ArgumentParser
    parser.program_name = "fileproc"
    parser.description = "文件处理工具 - 转换文本格式"
    
    parser.add_argument("input", true, "输入文件路径")
    
    parser.add_option("--verbose", true, false, "显示详细处理信息")
    parser.set_option_alias("--verbose", "-v")
    
    parser.add_option("--format", false, false, "转换格式 (text/upper/lower)")
    parser.set_option_alias("--format", "-f")
    parser.set_defaults("--format", "text")
    
    parser.add_option("--output", false, false, "输出文件路径")
    parser.set_option_alias("--output", "-o")
    parser.set_defaults("--output", null)
    
    try
        var parsed = parser.parse_args(args)
        
        var processor = new FileProcessor
        processor.verbose = parsed.verbose
        processor.format = parsed.format
        
        var output = parsed.output
        if output is null
            output = parsed.input + ".out"
        end
        
        processor.process(parsed.input, output)
        
        system.out.println("处理完成！")
        
    catch e
        system.out.println("错误: " + e.what())
        system.exit(1)
    end
end

# 运行主程序
main(context.cmd_args)
```

使用方式：
```bash
# 显示帮助
cs fileproc.csc -h

# 基本使用
cs fileproc.csc input.txt

# 指定输出和格式
cs fileproc.csc input.txt -o output.txt -f upper

# 详细模式
cs fileproc.csc input.txt -v --format=lower
```

