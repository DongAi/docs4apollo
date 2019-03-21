用户数据读表器。支持读取以excel进行编辑，并保存为txt的数据表，方便用户对数据表的维护。

数据表的数据之间以TAB键作为分割。

### 程序结构
`ContianerFactory`，为数据容器工厂类。负责管理每个数据表的容器。

`ContainerBase`，数据表容器的基类，定义了数据表容器的接口。

`ContainerImpl`，继承自`ContainerBase`，实现了数据表的数据加载或数据获取接口。

`TableContainer`，继承自`ContainerImpl`。每个数据表对应一个`TableContainer`对象，负责创建每个数据表对象。

`DataStreamer`，数据表文件读取类。负责读取每个数据表文件，并调用数据表对应的`TableContainer`对象的数据加载函数加载数据。

`DataTypeParser`，数据类型转换类。负责将数据表中配置的字符串记录的数据类型转化为代码中的enum类型，以减少代码中过多的字符串比较。

`TableBase`，数据表对象基类。

`TablePlanningTable`，继承自`TableBase`。Planning数据表的数据对象，该对象由`TableContainer`创建和管理，数据表的每一行对应一个`TablePlanningTable`对象。

### 数据表结构
为了保证数据表的可读性，要求每个数据表的第一行为数据列的注释名。注释名的存在可以方便以后对数据表读取代码生成工具的扩展。

数据表的第二行为数据列的数据类型。

从第三行开始才是需要用户配置的自定义数据。

另外，该程序提供了全局的通过ID获取数据表对象的接口，所以要求数据表的第一列为ID，类型为整型，取值不得小于0.

### 数据类型支持
目前支持读取整型，浮点型，字符串，整型数据，用户自定义类型的数据。

用户在数据表的第二行配置每个数据列的数据类型。在数据表中，数据类型和配置字符串的匹配规则为：
```bash
整型-->"INT"
浮点型-->"FLOAT"
字符串-->"STRING"
整形数组-->"INTARRAY"
自定义类型-->"CUSTOM"
```
数据表加载阶段会强制匹配上述的规则字符串，无法匹配时视为读表失败，用户可以查看错误日志确认错误原因。

用户需要确认每一列的数据类型，并按照规则字符串进行配置。

### 添加新的数据表
程序中使用宏定义了数据表的加载和数据获取逻辑，所以用户可以改动很少的代码以增删数据表。

以增加数据表`table_perception.txt`为例。
1. `ContainerFactory::register_all()`中增加：
   ```bash
   REGISTER_CONTAINER(TablePerception, TablePerceptionContainer);
   ```
2. `container_impl.h`的末尾处增加：
   ```bash
   CONTAINER_DECL(TablePerception);
   TABLE_GET_DECL(TablePerception);
   ```
3. `container_impl.cpp`的末尾处增加：
   ```bash
   CONTAINER_IMPL(TablePerception);
   TABLE_GET_IMPL(TablePerception);
   ```
4. 在`table_reader/table`文件夹下新增文件，`table_perception.h`和`table_perception.cpp`。
   
   实现`TablePerceptionTable`类继承自`TableBase`，重写接口`load_content`。该类包含`TablePerceptionContainer`指针对象。
   在`load_content`函数中，调用`TablePerceptionContainer`指针对象的`load_data`接口加载对应的数据。用户需要对每一个`load_data`
   的返回值进行`ASSERT_LOG`的判断，以尽早发现数据表配置错误。


### 新增自定义类型
该读表器支持加载用户自定义类型结构，但是需要用户单独实现数据结构和数据加载规则。

在数据表中配置用户自定义结构的数据时，数据结构的各个变量以逗号分隔，例如3,44,10.43表示包含两个整型和一个字符型的数据结构。

以新增自定义结构`Perception`为例。
1. 在`table_reader/logic`文件夹下，新增文件`logic_perception.h`和`logic_perception.cpp`。
   实现`Perception`类，继承自`LogicBase`。
2. `Perception`类重写接口`parse`，`parse`函数的`std::string`参数为从数据表中读取的原生的字符串，需要在`parse`函数中进行分割和读取。
3. 在`TablePerceptionTable`的`load_content`接口中获取原生字符串，并调用`Perception`对象的`parse`函数，例如：
   ```bash
   std::string dataitem;
   ret = container_->load_data(data_index++, &dataitem);
   ASSERT_LOG(ret, "load perception_data failed!");
   perception_data_.parse(dataitem);
   ```

### 获取数据
程序为每个数据表提供了全局的数据获取接口。用户可以通过数据表中配置的ID或者数据行所在的索引值获取数据。
以`Planning`数据表为例。
获取ID为1的数据表对象的接口为：
```bash
GetTablePlanningById(1)
```
获取索引为2的数据表对象的接口为：
```bash
GetTablePlanningByIndex(2)
```
上述两个函数返回值为`std::shared_ptr<TablePlanningTable>`，数据不存在时返回`nullptr`。

### 编译和运行
程序集成了`BCLOUD`构建系统，程序的依赖库包括`boost`，`gtest`和`comlog`，也全部引用自`iCode`仓库。

程序的本地构建命令为：
```bash
$ bcloud local
$ make
```
构建成功后，可执行文件为`./output/bin/table_reader`

运行`./output/bin/table_reader`，在屏幕上将输出`Planning`数据表ID为1和索引为2的数据。并且输出`gtest`的单元测试结果。

程序运行的日志文件为`log/table_reader.log`。用户可以通过查看该日志获取数据表加载的信息。

### 错误记录
为了增加程序的鲁棒性，尽早的发现和提醒用户数据表错误。程序中增加了几处错误检测机制。
1. 无法打开数据表文件时，程序退出，日志输出`cannot load file`
2. 无法获取数据表容器时，程序退出，日志输出`cannot get table container`
3. 数据类型配制错误时，程序assert crash退出，日志输出`cannot parse datatype, check the spelling!`
4. 数据列数和数据类型的列数不相等时，程序assert crash退出，日志输出`length of datatype is not equal to length of data!`
5. 配置的数据和数据类型不对应时，程序assert crash退出，日志输出`error type!......`
6. 获取数组数据或自定义数据，且配置不符合规则的，程序assert crash退出，日志输出`error format!`
