# 3.8 数据分析 (covanalysis)

`covanalysis` 是 CovScript 的数据分析框架，提供类似 pandas 的数据处理功能，包括 DataFrame、CSV 读取、过滤、分组、聚合等操作。

## 安装

```bash
cspkg install covanalysis
```

## 基本概念

covanalysis 提供了以下核心功能：

- **DataFrame**: 表格数据结构
- **CSV 读取**: 从 CSV 文件加载数据
- **过滤**: 按条件筛选数据
- **选择**: 选择特定列
- **分组**: 按列分组数据
- **聚合**: 统计运算（求和、平均、计数等）

## 读取 CSV 数据

```covscript
import analysis

# 读取 CSV 文件
var df = analysis.read_csv("data.csv")

# 显示数据
df.show()

# 显示特定行
df.by_index(0).show()
```

## 数据过滤

### 使用 pick 方法

```covscript
import analysis

var df = analysis.read_csv("universities.csv")

# 过滤美国的大学，且教育质量排名在前10
df = df.pick([](record) {
    return record.by_name("country") == "USA" &&
           record.by_name("quality_of_education") as number <= 10
})

# 选择特定列
df = df.select("world_rank", "institution")
df.show()
```

### 使用 filter 方法

```covscript
import analysis

var df = analysis.read_csv("universities.csv")

# 定义过滤条件
var target_country = "USA"
var target_quality = 10

df = df.filter(
    analysis::cond(
        country == target_country &&
        quality_of_education as number <= target_quality
    )
)

# 选择要显示的列
df.select("institution", "quality_of_education").show()
```

## 数据聚合

covanalysis 提供多种聚合运算符：

- `analysis.avg(col)` / `analysis.average(col)` - 平均值
- `analysis.sum(col)` / `analysis.summery(col)` - 求和
- `analysis.count(col)` - 计数
- `analysis.count_distinct(col)` - 去重计数

```covscript
import analysis

var df = analysis.read_csv("universities.csv")

# 过滤数据
df = df.filter(
    analysis::cond(
        country == "USA" &&
        quality_of_education as number <= 10
    )
)

# 计算平均分数
df.aggregate(
    analysis.avg("score").as("score_avg")
).show()
```

### 为聚合结果命名

```covscript
# 使用 as() 方法为聚合结果指定列名
df.aggregate(
    analysis.sum("patents"),
    analysis.avg("score").as("score_avg"),
    analysis.count("institution").as("total_count")
).show()
```

## 分组操作

```covscript
import analysis

var df = analysis.read_csv("universities.csv")

# 按国家和年份分组
df = df.groupby("country", "year")

# 对分组后的数据进行聚合
df.aggregate(
    analysis.sum("patents"),
    analysis.avg("score").as("score_avg")
).show()
```

输出示例：
```
# country     # year        # patents    # score_avg     #
| China      | 2015        | 12345      | 85.6          |
| China      | 2016        | 13456      | 86.2          |
| USA        | 2015        | 45678      | 92.3          |
| USA        | 2016        | 46789      | 92.8          |
```

## 遍历数据

### 使用 enumerate 遍历

```covscript
import analysis

var df = analysis.read_csv("universities.csv")

# 过滤和选择
df = df.pick([](record) {
    return record.by_name("country") == "USA" &&
           record.by_name("quality_of_education") as number <= 10
}).select("world_rank", "institution")

# 遍历每一行，打印索引和机构名称
df.enumerate([](idx, record) {
    system.out.println(idx as string + "\t" + record.by_name("institution"))
})
```

输出示例：
```
0    Harvard University
1    Massachusetts Institute of Technology
2    Stanford University
3    Princeton University
...
```

## 访问记录数据

DataFrame 中的每一行是一个 `datarecord` 对象，可以通过两种方式访问：

```covscript
# 通过列名访问
var value = record.by_name("column_name")

# 通过索引访问
var value = record.by_index(0)
```

## 实战案例：大学排名分析

```covscript
import analysis

class UniversityAnalyzer
    var df = null
    
    function construct(csv_file)
        this.df = analysis.read_csv(csv_file)
    end
    
    # 获取某个国家的顶尖大学
    function get_top_universities(country, limit)
        return this.df.filter(
            analysis::cond(country == country)
        ).select(
            "world_rank",
            "institution",
            "score",
            "quality_of_education"
        ).by_index_range(0, limit)
    end
    
    # 按国家统计大学数量和平均分数
    function country_statistics()
        return this.df.groupby("country").aggregate(
            analysis.count("institution").as("university_count"),
            analysis.avg("score").as("avg_score")
        )
    end
    
    # 分析教育质量排名
    function quality_analysis(threshold)
        return this.df.filter(
            analysis::cond(
                quality_of_education as number <= threshold
            )
        ).groupby("country").aggregate(
            analysis.count("institution").as("high_quality_count"),
            analysis.avg("quality_of_education").as("avg_quality_rank")
        )
    end
    
    # 年度趋势分析
    function yearly_trend(country)
        return this.df.filter(
            analysis::cond(country == country)
        ).groupby("year").aggregate(
            analysis.count("institution").as("university_count"),
            analysis.avg("score").as("avg_score"),
            analysis.sum("patents").as("total_patents")
        )
    end
end

# 使用分析器
var analyzer = new UniversityAnalyzer{"cwurData.csv"}

# 获取美国前10名大学
system.out.println("=== 美国前10名大学 ===")
analyzer.get_top_universities("USA", 10).show()

# 国家统计
system.out.println("\n=== 各国大学统计 ===")
analyzer.country_statistics().show()

# 高质量大学分析
system.out.println("\n=== 教育质量前10的大学分布 ===")
analyzer.quality_analysis(10).show()

# 美国大学年度趋势
system.out.println("\n=== 美国大学年度趋势 ===")
analyzer.yearly_trend("USA").show()
```

## 数据类型转换

在处理数据时，可能需要进行类型转换：

```covscript
# 字符串转数字
var numeric_value = record.by_name("score") as number

# 在过滤条件中使用类型转换
df = df.filter(
    analysis::cond(
        quality_of_education as number <= 10 &&
        score as number >= 90.0
    )
)
```

## DataFrame 操作链

covanalysis 支持链式操作，可以连续应用多个转换：

```covscript
import analysis

var result = analysis.read_csv("data.csv")
    .filter(analysis::cond(country == "China"))
    .select("institution", "score", "year")
    .groupby("year")
    .aggregate(
        analysis.count("institution").as("count"),
        analysis.avg("score").as("avg_score")
    )

result.show()
```

## 最佳实践

### 1. 合理使用过滤条件

```covscript
# 在分组前过滤可以提高性能
df = df.filter(analysis::cond(year >= 2015))
         .groupby("country")
         .aggregate(analysis.avg("score"))
```

### 2. 选择必要的列

```covscript
# 只选择需要的列，减少内存使用
df = df.select("institution", "score", "country")
```

### 3. 使用有意义的列名

```covscript
# 为聚合结果指定清晰的列名
df.aggregate(
    analysis.avg("score").as("average_score"),
    analysis.sum("patents").as("total_patents"),
    analysis.count("institution").as("institution_count")
)
```

### 4. 数据验证

```covscript
# 在处理前检查数据
if df.size() == 0
    system.out.println("警告：数据集为空")
    return
end

# 检查列是否存在
if !df.has_column("score")
    system.out.println("错误：缺少 score 列")
    return
end
```

## 总结

covanalysis 提供了强大的数据分析功能：

- **DataFrame**: 灵活的表格数据结构
- **CSV 支持**: 轻松读取和处理 CSV 文件
- **过滤和选择**: 灵活的数据筛选
- **分组和聚合**: SQL 风格的数据统计
- **链式操作**: 简洁的数据处理流程

通过 covanalysis，CovScript 可以处理中小规模的数据分析任务，适用于数据清洗、统计分析、报表生成等场景。
