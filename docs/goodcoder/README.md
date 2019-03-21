# zhangdongai
完成goodcoder的C++作业，设计的一个通用的读表器。

支持用户自定义表格的内容，支持读取int，float，char*，数组和自定义的数据类型。

用户可以方便的添加一个新的数据表。

## 快速开始
该项目依赖于boost，gtest，comlog。

使用bcloud进行本地的编译，

编译命令为：

$ bcloud local

$ make

运行./output/bin/table_reader即可运行程序，并输出数据表的配置结果和GTest结果

## 测试
运行./output/bin/table_reader即可运行程序，并输出数据表的测试结果。

日志系统使用comlog，程序运行后，日志文件为log/table_reader.log

## 如何快速添加一个新的数据表
该项目允许用户快速的添加数据表。例如添加数据表table_perception.txt，步骤为：

1. 在container_impl.h中添加宏CONTAINER_DECL(TablePerception)和TABLE_GET_DECL(TablePerception)

2. 在container_impl.cpp中添加宏CONTAINER_IMPL(TablePerception)和TABLE_GET_IMPL(TablePerception)

3. 在container_factory.cpp中添加宏REGISTER_CONTAINER(TablePerception, TablePerceptionContainer)
（步骤3可以优化点，目前版本中未实现）

4. 在data_streamer.cpp中添加宏TABLE_READER(TablePerception, "./config/table_perception.txt");

5. 增加数据表，和数据表类文件（数据表类的代码可以使用工具批量自动生成）

## 如何获取数据表
用户添加数据表后，可以使用全局函数GetTablePerceptionById和GetTablePerceptionByIndex获取某一行的数据


