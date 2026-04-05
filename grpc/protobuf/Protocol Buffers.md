# protobuf

Protocol Buffers（简称为protobuf）是Google开发的用于**序列化结构化数据**的语言无关、平台无关、可扩展的机制。与JSON、XML等序列化方式相比，Protocol Buffers更小、更快、更简单(是一种**二进制的格式**)。只需定义一次数据的结构化方式，之后就可以使用特殊生成的源代码很容易地将结构化数据读取和写入到各种数据流，并使用各种编程语言。

## 如何使用

1. 首先编写`.proto`文件来定义数据结构
2. 然后用proto编译器将`.proto`文件生成你所需要的编程语言的类文件
3. 然后在代码中可以直接操作proto生成的类

### `.proto`文件编写示例

```protobuf
syntax = "proto2";

package tutorial;

message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    PHONE_TYPE_UNSPECIFIED = 0;
    PHONE_TYPE_MOBILE = 1;
    PHONE_TYPE_HOME = 2;
    PHONE_TYPE_WORK = 3;
  }

  message PhoneNumber {
    optional string number = 1;
    optional PhoneType type = 2 [default = PHONE_TYPE_HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

- `syntax = "proto2";`：声明使用的 Protobuf 版本。目前主流有 `proto2` 和 `proto3`。必须放在第一行
- `package tutorial;`：声明包名。这类似于 Java 的 `package` 或 C++ 的 `namespace`，目的是为了防止不同项目之间出现相同的结构名称（比如两个项目都有 `Person`）从而导致冲突。

- `Message`这是 Protobuf 中最重要的关键字。在这里，我们定义了一个名为 `Person` 的数据结构。

- `Message`内部的属性, 遵循格式：`[修饰符] [数据类型] [字段名] = [标签编号];`

  - 修饰符

    ```txt
    1. optional
    	该字段在消息中可以出现 0 次或 1 次。如果没有设置该字段，解析时会使用默认值。
    2. repeated
    	该字段在消息中可以出现任意次数（包括 0 次）。在生成的代码中，它通常被映射为数组
    3. required
    	发送方在发送消息时必须提供该字段的值，接收方在解析时也会检查该字段是否存在。如果不包含，会导致消息解析失败。仅在 proto2 中支持，在 proto3 中已被彻底废弃。
    4. map
    	声明字典数据
    ```

  - 类型

    | Protobuf 类型 | C++ 类型              | 备注                                                         |
    | ------------- | --------------------- | ------------------------------------------------------------ |
    | double        | double                | 64位浮点数                                                   |
    | float         | float                 | 32位浮点数                                                   |
    | int32         | int                   | 32位整数、使用 Varint 变长编码                               |
    | int64         | long                  | 64位整数、使用 Varint 变长编码                               |
    | uint32        | unsigned int          | 32位无符号整数、使用 Varint 变长编码                         |
    | uint64        | unsigned long         | 64位无符号整数、使用 Varint 变长编码                         |
    | sint32        | signed int            | 32位整数，处理负数效率比 int32 更高、底层使用 ZigZag 编码，将负数映射为正数后再进行 Varint 压缩，效率极高。 |
    | sint64        | signed long           | 64位整数，处理负数效率比 int64 更高、底层使用 ZigZag 编码    |
    | fixed32       | unsigned int（32位）  | 总是占 4 个字节，如果数值通常较大（接近或超过 2²⁸），比 uint32 更高效 |
    | fixed64       | unsigned long（64位） | 总是占 8 个字节，如果数值通常较大（接近或超过 2⁵⁶），比 uint64 更高效 |
    | sfixed32      | int（32位）           | 总是占 4 个字节                                              |
    | sfixed64      | long（64位）          | 总是占 8 个字节                                              |
    | bool          | bool                  | 布尔类型                                                     |
    | string        | string                | 必须是 UTF-8 或 7-bit ASCII 编码文本                         |
    | bytes         | string                | **用于多字节数据（如中文），建议字符数据用 bytes**           |
    | enum          | enum                  | 枚举类型                                                     |
    | message       | object of class       | 自定义消息类型                                               |

  - 标签编号
    这不是变量的默认值！它被称为“字段标签号”（Tag Number）。Protobuf 序列化后的数据是二进制的，它不保存 "name" 或 "id" 这种字符串，而是用 1、2、3 这些数字来代替字段名，以此来极大地压缩数据体积。注意：1 到 15 的数字在编码时占用空间最小，通常留给最常用的字段。

- 枚举

  `= 0`, `= 1`：和前面的标签号不同，这里的数字是真正的**枚举值**。关键规则：在 Protobuf 的枚举中，第一个枚举值必须是 0。

- 嵌套定义

  嵌套 `message`：你可以在一个 `message` 内部定义另一个 `message`（如 `PhoneNumber`）。

- 指定默认值

  `[default = ...]`：为 `optional` 字段设置一个默认值。如果发送方没有提供 `type` 数据，接收方在读取时，会自动将其视为 `PHONE_TYPE_HOME`。

### 编译

- 方法一：直接使用protoc命令：`protoc -I=. --cpp_out=. addressbook.proto`

  - `-I`指定`.protoc`文件的路径
  - `--cpp_out` 表示生成C++代码

- 方法二：使用cmake自动生成

  ```cmake
  # 1. 指定你的 .proto 文件路径
  set(PROTO_FILE ${CMAKE_CURRENT_SOURCE_DIR}/addressbook.proto)
  
  # 2. 让 CMake 自动调用 protoc 编译器！
  # 这个宏会自动生成 addressbook.pb.cc 和 addressbook.pb.h，并将路径存入 PROTO_SRCS 和 PROTO_HDRS 变量中
  protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS ${PROTO_FILE})
  ```

### 生成的C++代码

生成的`addressbook.pb.h`生成的文件结构如下:
```C++
namespace tutorial {  			//package tutorial
    enum Person_PhoneType;  	//renum PhoneType
    class Person_PhoneNumber;	//message PhoneNumbe
    class Person;				//message Person
    class AddressBook;			//message AddressBook
};
```

person类的实现如下：

```C++
class Person{
  // optional string name = 1;
  public:
  bool has_name() const;
  void clear_name();
  const std::string& name() const;
  void set_name(const std::string& value);
  void set_name(std::string&& value);
  void set_name(const char* value);
  void set_name(const char* value, size_t size);
  std::string* mutable_name();
  std::string* release_name();
  void set_allocated_name(std::string* name);
    
  // optional int32 id = 2;
  bool has_id() const;
  void clear_id();
  ::PROTOBUF_NAMESPACE_ID::int32 id() const;
  void set_id(::PROTOBUF_NAMESPACE_ID::int32 value);
    
  // optional string email = 3;
  bool has_email() const;
  void clear_email();
  const std::string& email() const;
  void set_email(const std::string& value);
  void set_email(std::string&& value);
  void set_email(const char* value);
  void set_email(const char* value, size_t size);
  std::string* mutable_email();
  std::string* release_email();
  void set_allocated_email(std::string* email);
 
  // repeated .tutorial.Person.PhoneNumber phones = 4;
  typedef Person_PhoneNumber PhoneNumber;  //注意这里C++代码，没有将PhoneNumber声明内部类
  public:
  int phones_size() const;
  void clear_phones();
  ::tutorial::Person_PhoneNumber* mutable_phones(int index);
  ::PROTOBUF_NAMESPACE_ID::RepeatedPtrField< ::tutorial::Person_PhoneNumber >*mutable_phones();
  const ::tutorial::Person_PhoneNumber& phones(int index) const;
  ::tutorial::Person_PhoneNumber* add_phones();
  const ::PROTOBUF_NAMESPACE_ID::RepeatedPtrField<::tutorial::Person_PhoneNumber>& phones() const;
};
```

上面属性的相关方法分为四大类：

1. **状态管理**

- `has_name`: 用来判断这个对象是否被设置了 `name` 字段。因为它是 `optional`（可选）的，接收端拿到数据时，可以通过这个方法判断发送端到底有没有传这个值。
- `clear_name()`：将 `name` 字段清空。调用后，不仅字符串内容被清空，`has_name()` 也会变回 `false`。
- 对于`repeated`类型：`phones_size()`表示数组大小。`clear_phones()`清空数组。没有`phones_name`。

2. **读取数据**

- `name()\id()\email()` : 读取当前的属性值
- 对于`repeated`类型，通过下标来读取:`phones(int index)`.

3. **写数据**

- `set_`
- 注意对于字符串提供了多个`set_`方法

4. **内存管理**

- `std::string* mutable_name();`
  - 作用：直接获取内部字符串的**可修改指针**。
  - 使用场景：假设你要在原来的名字后面追加字符。如果用传统方法：`set_name(name() + " Junior")` 会产生临时变量和拷贝。如果用 `mutable_name()->append(" Junior")`，直接在原地修改，**零拷贝**。调用这个方法后，`has_name()` 会自动变成 `true`。

- `std::string* release_name();`
  - 作用：**放弃所有权**。Protobuf 把内部存放字符串的内存指针交给你，然后它自己把记录清空（`has_name()` 变回 false）。
  - 拿到了这个指针，**你必须手动 `delete` 它**，否则就会内存泄漏！它常用于把数据从 Protobuf 对象中“抠”出来交给别的地方，而不发生内存拷贝。
- `void set_allocated_name(std::string* name);`
  - **接收所有权**。这和 `release` 正好反过来。你如果在外面用 `new std::string("John")` 创建了一个字符串，可以直接把指针塞给 Protobuf。
  - Protobuf 会接管这块内存。当这个 Protobuf 对象被销毁时，**Protobuf 会负责帮你 `delete` 这块内存**。绝对不要把你栈上的指针（比如局部变量的地址）传给它，否则程序运行时必崩！

### 使用protobuf生成的代码

**序列化和反序列化**

每个消息类都有使用[二进制格式](https://protobuf.dev/programming-guides/encoding/)读写消息的函数，包括：

- `bool SerializeToString(string* output) const`：序列化消息，并将字节序列存储在字符串中（注意字节序列是二进制格式，不是文本格式，只是使用`string`类作为容器）
- `string SerializeAsString() const`：序列化消息，并返回字节序列
- `bool SerializeToOstream(ostream* output) const`：序列化消息，并写入给定的输出流
- `bool SerializeToArray(void* data, int size) const`：序列化消息，并存储在字节数组中
- `bool ParseFromString(const string& data)`：从字符串解析消息
- `bool ParseFromIstream(istream* input)`：从输入流解析消息
- `bool ParseFromArray(const void* data, int size)`：从字节数组解析消息

**标准消息函数**

每个消息类还包含许多其他函数，可以检查或操作整个消息，包括：

- `bool IsInitialized() const`：检查是否已设置所有`required`字段
- `string DebugString() const`：返回消息的人类可读的表示，对于调试特别有用
- `void CopyFrom(const Message& from)`：用给定消息覆盖该消息
- `void MergeFrom(const Message& from)`：将给定消息与该消息合并
- `void Clear()`：将所有字段重置为空状态

**示例代码**

```C++
#include <iostream>
#include <string>
#include "addressbook.pb.h" // 引入 protoc 生成的头文件

int main() {
    // ---------------------------------------------------------
    // 阶段 1：创建并赋值 (构建数据)
    // ---------------------------------------------------------
    tutorial::Person person;
    
    person.set_id(1234);
    person.set_name("John Doe");
    person.set_email("jdoe@example.com");

    // 添加第一个电话 (Home，使用默认类型)
    tutorial::Person::PhoneNumber* phone1 = person.add_phones();
    phone1->set_number("555-4321");
    // 不设置 type，它会自动默认为 PHONE_TYPE_HOME

    // 添加第二个电话 (Mobile)
    tutorial::Person::PhoneNumber* phone2 = person.add_phones();
    phone2->set_number("123-4567");
    phone2->set_type(tutorial::Person::PHONE_TYPE_MOBILE); // 使用生成的枚举常量

    // ---------------------------------------------------------
    // 阶段 2：序列化 (将对象变成二进制，准备发送或存储)
    // ---------------------------------------------------------
    std::string serialized_data;
    if (person.SerializeToString(&serialized_data)) {
        std::cout << "序列化成功！二进制数据大小: " << serialized_data.size() << " bytes." << std::endl;
    }

    // ---------------------------------------------------------
    // 阶段 3：反序列化 (模拟接收端收到二进制数据并还原)
    // ---------------------------------------------------------
    tutorial::Person received_person;
    if (received_person.ParseFromString(serialized_data)) {
        std::cout << "\n反序列化成功！读取数据：" << std::endl;
        std::cout << "Name: " << received_person.name() << std::endl;
        std::cout << "ID: " << received_person.id() << std::endl;
        
        // 遍历 repeated 字段
        for (int i = 0; i < received_person.phones_size(); ++i) {
            const tutorial::Person::PhoneNumber& phone = received_person.phones(i);
            std::cout << "Phone " << i << " number: " << phone.number() << std::endl;
        }
    }

    // 可选：在程序结束前清理 Protobuf 库申请的全局内存
    google::protobuf::ShutdownProtobufLibrary();

    return 0;
}
```

### 字典数据

在 Protobuf 中定义字典（键值对 / Key-Value对）非常简单，它专门提供了一个 **`map`** 关键字。你可以把它完全等同于 C++ 中的 `std::map` 或 `std::unordered_map`，以及 Python 中的 `dict`。

```protobuf
message Person {
  optional string name = 1;
  
  // 【基础字典】：记录个人的扩展属性，比如 "hobby" -> "coding"
  // 语法：map<键类型, 值类型> 字段名 = 标签号;
  map<string, string> attributes = 5;

  // 【高级字典】：值类型也可以是你自己定义的 Message 类型
  // 比如记录不同身份的紧急联系人："boss" -> PhoneNumber, "wife" -> PhoneNumber
  message PhoneNumber {
    optional string number = 1;
  }
  map<string, PhoneNumber> emergency_contacts = 6;
}
```

**性质**

- 键（Key）的类型受限：Key 只能是整数类型（如 `int32`, `int64` 等）或 字符串类型（`string`）。它绝对不能是浮点数（`float`/`double`）、`bytes`、枚举（`enum`）或自定义的 `message`。

- 值（Value）的类型随意：Value 可以是除了 `map` 以外的任何类型（对，你不能直接嵌套写 `map<string, map<string, int32>>`，如果有这种需求，需要用一个 `message` 包裹一下里层的 map）。

- 不能加修饰符：`map` 字段天生就是包含多个元素的，所以你千万不能在它前面加 `repeated`、`optional` 或 `required`，直接写 `map` 即可。

- 无序性：就像 C++ 的 `std::unordered_map` 一样，当你序列化然后反序列化后，字典里面的键值对顺序是不保证的。

**C++中的使用**

Protobuf 编译器为 `map` 生成的 C++ 底层数据结构是 `google::protobuf::Map<Key, Value>`。好消息是，它的用法和 C++ 标准库的 `std::map` 几乎一模一样！

针对上面的 `attributes` 字段，生成的关键 C++ 接口如下：

```C++
// 获取只读字典 (Getter)
const ::google::protobuf::Map<std::string, std::string>& attributes() const;

// 获取可修改字典的指针 (Mutable Getter)
::google::protobuf::Map<std::string, std::string>* mutable_attributes();
```

写入数据

```C++
tutorial::Person person;

// 方法 1：使用经典的 insert (类似 std::map)
person.mutable_attributes()->insert({"hobby", "reading"});

// 方法 2：使用更直观的方括号 [] 重载语法 (最推荐！)
(*person.mutable_attributes())["role"] = "admin";
(*person.mutable_attributes())["city"] = "Beijing";
```

读取数据

```C++
// 1. 判断某个 Key 是否存在并读取
const auto& attrs = person.attributes();
if (attrs.find("role") != attrs.end()) {
    std::cout << "Role is: " << attrs.at("role") << std::endl;
}

// 2. 使用 C++11 的 range-based for 循环遍历整个字典
for (const auto& pair : person.attributes()) {
    std::cout << "Key: " << pair.first << ", Value: " << pair.second << std::endl;
}

// 3. 获取字典的元素个数
int map_size = person.attributes().size();
```

### Protobuf 为什么比 JSON 或 XML 快？体积为什么小？

| **特性对比** | **JSON / XML (文本序列化)**                                  | **Protobuf (二进制序列化)**                                  |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **数据形态** | 纯文本，携带大量如 `{`, `"`, `:` 的结构符。                  | 紧凑的二进制字节流，无任何结构符。                           |
| **字段标识** | 每次传输都必须携带完整的字段名（如 `"userName": "Tom"`）。   | **不传输字段名**，只传输提前约定好的**标签号**（如 `1` 代表 userName）。 |
| **解析速度** | 需进行复杂的字符串匹配、正则解析和类型转换（如字符串转整型）。 | 根据类型（Wire Type）按位读取偏移量，直接内存映射，几乎无需类型转换。 |

### 为什么 Google 在 Protobuf 3 中彻底废弃了 `required` 关键字？
 在实际业务中，微服务的升级是不可能同时进行的（通常是先升级服务端，再升级客户端）。如果定义了 `required`，一旦未来业务变化需要删除这个字段，老版本的程序在反序列化时就会直接报错甚至崩溃。因此，所有字段都应该被视为可选的（`optional`），数据的必填校验应当交给业务逻辑层去处理，而不是死板地绑定在通信协议上。

### 如果业务需求变了，我需要废弃掉一个旧字段（比如标签号为 2 的 id），我能直接把它删掉，然后把标签号 2 给新字段用吗？

绝对不能！如果复用旧的标签号，老客户端发来的旧数据会被新服务端错误地解析到新字段中，导致严重的数据错乱。正确的做法是将旧字段注释掉，并使用 `reserved` 关键字把旧的标签号和字段名“封印”起来，防止后人误用： `reserved 2, 15, 9 to 11;` `reserved "old_id", "obsolete_name";`

**如果客户端发送的字段，而服务端没有会怎么样**

服务端绝对不会崩溃，也不会报错，它会默默地把已知字段解析出来，并对不认识的字段进行“特殊处理”。在 Proto2 以及现在的 Proto3 (v3.5 及更高版本) 中：默认保留！Protobuf 会把这些不认识的字节流默默存放在 C++ 对象底层的 `UnknownFieldSet`（未知字段集合）中。为什么这么设计？（重点） 假设你的服务器是一个中间件（Proxy/网关）。它收到客户端的数据，自己不处理这个新字段，但需要把整个 Message 转发给后端的最终服务器。因为 Protobuf 保留了未知字段，所以当网关再次调用 `SerializeToString()` 发送数据时，那些它不认识的新字段会被原封不动地重新打包发出去！ 这种机制保证了中间节点不需要随着业务节点频繁升级。早期 Proto3 (v3.0 - v3.4)在解析时直接丢弃所有未知字段。