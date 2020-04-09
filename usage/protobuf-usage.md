# 语言指南（proto3）

-   [定义消息类型](https://developers.google.com/protocol-buffers/docs/proto3#simple)
-   [标量值类型](https://developers.google.com/protocol-buffers/docs/proto3#scalar)
-   [默认值](https://developers.google.com/protocol-buffers/docs/proto3#default)
-   [枚举](https://developers.google.com/protocol-buffers/docs/proto3#enum)
-   [使用其他消息类型](https://developers.google.com/protocol-buffers/docs/proto3#other)
-   [嵌套类型](https://developers.google.com/protocol-buffers/docs/proto3#nested)
-   [更新消息类型](https://developers.google.com/protocol-buffers/docs/proto3#updating)
-   [未知字段](https://developers.google.com/protocol-buffers/docs/proto3#unknowns)
-   [任何](https://developers.google.com/protocol-buffers/docs/proto3#any)
-   [一个](https://developers.google.com/protocol-buffers/docs/proto3#oneof)
-   [地图](https://developers.google.com/protocol-buffers/docs/proto3#maps)
-   [配套](https://developers.google.com/protocol-buffers/docs/proto3#packages)
-   [定义服务](https://developers.google.com/protocol-buffers/docs/proto3#services)
-   [JSON对应](https://developers.google.com/protocol-buffers/docs/proto3#json)
-   [选件](https://developers.google.com/protocol-buffers/docs/proto3#options)
-   [生成课程](https://developers.google.com/protocol-buffers/docs/proto3#generating)

本指南介绍了如何使用 protobuf 语言来构造 protobuf 数据，包括`.proto`文件语法以及如何从`.proto`文件中生成数据访问类。它涵盖了 protobuf 语言的**proto3**版本：有关旧的**proto2**语法的信息，请参见《[Proto2语言指南》](https://developers.google.com/protocol-buffers/docs/proto)。

这是参考指南–有关使用本文档中描述的许多功能的分步示例，请参见所选择语言的[教程](https://developers.google.com/protocol-buffers/docs/tutorials)（当前仅适用于proto2；即将推出更多proto3文档）。

## 定义消息类型

首先让我们看一个非常简单的例子。假设您要定义一个搜索请求消息格式，其中每个搜索请求都有一个查询字符string，您感兴趣的特定结果页面以及每页包含的结果数量。这是`.proto`用于定义消息类型的文件。
```
syntax = "proto3";

message SearchRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
}
```
-   文件的第一行指定您正在使用`proto3`语法：如果不这样做，则 protobuf 编译器会假定您正在使用[proto2](https://developers.google.com/protocol-buffers/docs/proto)。这必须是文件的第一非空行，非注释行。
-   所述`SearchRequest`消息定义指定了三个字段（名称/值对），每个字段用于在此类型的消息包括的数据。每个字段都有一个名称和类型。

### 指定字段类型

在上面的示例中，所有字段均为[标量类型](https://developers.google.com/protocol-buffers/docs/proto3#scalar)：两个整数（`page_number`和`result_per_page`）和一个字符string（`query`）。但是，您也可以为字段指定复合类型，包括[枚举](https://developers.google.com/protocol-buffers/docs/proto3#enum)和其他消息类型。

### 分配字段编号

如您所见，消息定义中的每个字段都有一个**唯一的编号**。这些字段号用于标识[消息二进制格式的](https://developers.google.com/protocol-buffers/docs/encoding)字段，一旦使用了消息类型，就不应更改这些字段号。请注意，范围为1到15的字段编号需要一个字节来编码，包括字段编号和字段的类型（您可以在[Protocol Buffer Encoding中](https://developers.google.com/protocol-buffers/docs/encoding.html#structure)找到有关此内容的更多信息）。16到2047之间的字段编号占用两个字节。因此，您应该为经常出现的消息元素保留数字1到15。切记为将来可能添加的频繁出现的元素留出一些空间。

您可以指定最小的字段编号是1，最大为 $2^{29}$ - 1，或536870911。您也不能使用数字 1900 0到19999（`FieldDescriptor::kFirstReservedNumber`至`FieldDescriptor::kLastReservedNumber`），因为它们是为 protobuf 实现保留的-如果您在中使用这些保留数之一，则 protobuf 编译器会对`.proto`文件给出告警提示。同样，您不能使用任何以前[保留的](https://developers.google.com/protocol-buffers/docs/proto3#reserved)字段号。

### 指定字段规则

消息字段可以是以下之一：

-   `singular`：格式正确的消息可以包含零个或一个此字段（但不能超过一个）。这是proto3语法的默认字段规则。
-   `repeated`：在格式正确的消息中，此字段可以重复任意次（包括零次）。重复值的顺序将保留。

在proto3中，`repeated`标量数字类型的字段默认情况下使用`packed`编码。

您可以在[ protobuf 编码中](https://developers.google.com/protocol-buffers/docs/encoding.html#packed)找到有关`packed`编码的更多信息。

### 添加更多消息类型

可以在单个`.proto`文件中定义多种消息类型。如果要定义多个相关消息，这很有用–例如，如果要定义与您的`SearchResponse`消息类型相对应的回复消息格式，可以将其添加到相同的消息中`.proto`：
```
message SearchRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
}

message SearchResponse {
    ...
}
```
### 添加注释

要将注释添加到`.proto`文件中，请使用 C/C++ 样式的`//`和`/* ... */`语法。
```
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```

### 保留字段

如果您通过完全删除字段或将其注释掉来[更新](https://developers.google.com/protocol-buffers/docs/proto3#updating)消息类型，则将来的用户在对类型进行更新时可以重用该字段号。但如果他们以后要加载此`.proto`的旧版本，可能会导致严重的问题，包括数据损坏，隐私错误等。确保不会发生这种情况的一种方法是，将已删除字段的字段编号（和/或名称，也可能导致JSON序列化的问题）指定为`reserved`。如果将来有用户尝试使用这些字段标识符，则 protobuf 编译器会告警提示。
```
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

请注意，您不能在同一条`reserved`语句中混用字段名称和字段编号。

### 您的`.proto`产生了什么？

在`.proto`上运行[ protobuf 编译器](https://developers.google.com/protocol-buffers/docs/proto3#generating)时，编译器会以您指定的语言生成能够处理在文件中`.proto`描述的消息类型的代码片段，包括获取和设置字段值，将消息序列化为输出流，并从输入流中解析消息。

-   对于**C ++**，编译器从每个`.proto`生成一个`.h`和`.cc`文件，并为文件中描述的每种消息类型提供一个类。
-   对于**Java**，编译器会生成一个`.java`文件，其中包含每种消息类型的类以及`Builder`用于创建消息类实例的特殊类。
-   **Python**有点不同– Python编译器会在您的`.proto`中生成一个模块，其中包含每种消息类型的静态描述符，然后该模块与_元类_一起使用，以在运行时创建必要的Python数据访问类。
-   对于**Go**，编译器`.pb.go`将为文件中的每种消息类型生成一个具有相应类型的文件。
-   对于**Ruby**，编译器将`.rb`使用包含您的消息类型的Ruby模块生成一个文件。
-   对于**Objective-C**，编译器从每个`.proto`生成一个`pbobjc.h`和`pbobjc.m`文件，并为文件中描述的每种消息类型提供一个类。
-   对于**C＃**，编译器会从每个`.proto`生成一个`.cs文件，并为文件中描述的每种消息类型提供一个类。
-   对于**Dart**，编译器会为`.proto`文件中的每种消息类型生成一个带有类的`.pb.dart`文件。

您可以按照所选语言的教程（即将推出proto3版本）找到有关每种语言使用API​​的更多信息。有关API的更多详细信息，请参见相关的[API参考](https://developers.google.com/protocol-buffers/docs/reference/overview)（proto3版本也即将推出）。

## 标量值类型

标量消息字段可以具有以下类型之一-该表显示`.proto`文件中指定的类型，以及自动生成的类中的相应类型：

| .proto类型 | 说明                                        | C++ 类型 | Java 类型 | Python 类型\[2]      | Go 类型     | Ruby 类型                  | C＃ 类型 | PHP 类型      | Dart 类型       |
| -------- | ----------------------------------------- | ------ | ------ | ----------------- | ------- | ---------------------- | ---- | ---------- | ---------- |
| double        |                                           | double      | double      | float                | float64 | float                     | double    | float         | double          |
| float       |                                           | float     | float     | float                | float32 | float                     | float   | float         | double          |
| int32    | 使用可变长度编码。负数编码效率低下–如果您的字段可能具有负值，请改用sint32。 | int32  | int     | int                | int32   | Fixnum或Bignum（根据需要）    | int   | integer         | int         |
| int64    | 使用可变长度编码。负数编码效率低下–如果您的字段可能具有负值，请改用sint64。 | int64  | long      | int / long\[3]    | int64   | Bignum                   | long    | integer/string\[5] | int64       |
| uint32   | 使用可变长度编码。                                 | uint32 | int\[1] | int / long\[3]    | uint32  | Fixnum或Bignum（根据需要）    | int  | integer         | int         |
| uint64   | 使用可变长度编码。                                 | uint64 | long\[1]  | int / long\[3]    | uint64  | Bignum                   | ulong   | integer/string\[5] | int64       |
| sint32   | 使用可变长度编码。有符号的int值。与常规int32相比，它们更有效地编码负数。  | int32  | int     | int                | int32   | Fixnum或Bignum（根据需要）    | int   | integer         | int         |
| sint64   | 使用可变长度编码。有符号的int值。与常规int64相比，它们更有效地编码负数。  | int64  | long      | int / long\[3]    | int64   | Bignum                   | long    | integer/string\[5] | int64       |
| fixed      | 始终为四个字节。如果值通常大于228，则比uint32更有效。           | uint32 | int\[1] | int / long\[3]    | uint32  | Fixnum或Bignum（根据需要）    | int  | integer         | int         |
| fixed64     | 始终为八个字节。如果值通常大于256，则比uint64更有效。           | uint64 | long\[1]  | int / long\[3]    | uint64  | Bignum                   | ulong   | integer/string\[5] | int64       |
| sfixed32     | 始终为四个字节。                                  | int32  | int     | int                | int32   | Fixnum或Bignum（根据需要）    | int   | integer         | int         |
| sfixed64      | 始终为八个字节。                                  | int64  | long      | int / long\[3]    | int64   | Bignum                   | long    | integer/string\[5] | int64       |
| bool       |                                           | bool     | boolean    | bool                | bool      | TrueClass / FalseClass | bool   | boolean        | bool         |
| string        | 字符string必须始终包含UTF-8编码或7位ASCII文本，并且不能超过$2^{32}$。     | string      | string      | str / unicode\[4] | string       | String（UTF-8）              | string    | string          | string          |
| bytes      | 可以包含不超过$2^{32}$的任意字节序列。                        | string      | ByteString    | str                | \[]byte   | String（ASCII-8BIT）         | ByteString  | string          | list\<int\> |

当您在[Protocol Buffer Encoding中](https://developers.google.com/protocol-buffers/docs/encoding)对消息进行序列化时，可以找到更多有关如何编码这些类型的信息。

\[1]在Java中，无符号的32位和64位整数使用带符号的对等体表示，最高位仅存储在符号位中。

\[2]在所有情况下，将为字段赋值都会执行类型检查以确保其有效。

\[3]64位或无符号32位整数在解码时始终表示为long，但如果在设置字段时给出int，则可以为int。在所有情况下，该值都必须符合赋值时表示的类型。参见\[2]。

\[4]Python字符string在解码时表示为unicode，但如果给出了ASCII字符string，则可以为str（此字符string可能会发生变化）。

\[5]在64位计算机上使用Integer，在32位计算机上使用string。

## 默认值

解析消息时，如果编码的消息不包含特定的`singular `元素，则已解析对象中的相应字段将设置为该字段的默认值。这些默认值是特定于类型的：

-   对于字符string，默认值为空字符string。
-   对于字节，默认值为空字节。
-   对于boolean，默认值为false。
-   对于数字类型，默认值为零。
-   对于[枚举](https://developers.google.com/protocol-buffers/docs/proto3#enum)，默认值为第**一个定义的枚举值**，必须为0。
-   对于消息字段，未设置该字段。它的确切值取决于语言。有关详细信息，请参见[生成的代码指南](https://developers.google.com/protocol-buffers/docs/reference/overview)。

重复字段的默认值为空（通常为相应语言的空列表）。

请注意，对于标量消息字段，一旦解析了一条消息，就无法告诉该字段是显式设置为默认值（例如，boolean是否设置为`false`）还是根本没有设置：在定义消息类型时您应该牢记这一点。

有关默认值在生成的代码中如何工作的更多详细信息，请参见所选语言的[生成的代码指南](https://developers.google.com/protocol-buffers/docs/reference/overview)。

## 枚举

在定义消息类型时，您可能希望其某个字段仅具有一个预定义的值列表。例如，假设你想为每个`SearchRequest`添加一个`corpus`字段，其中`corpus`可以为`UNIVERSAL`，`WEB`，`IMAGES`，`LOCAL`，`NEWS`，`PRODUCTS` 或 `VIDEO`。您可以通过`enum`在消息定义中为每个可能的值添加一个常量来非常简单地执行此操作。

在下面的示例中，我们添加了一个带有所有可能值的`enum`被叫项`Corpus`，以及一个type字段`Corpus`：
```
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```
如您所见，`Corpus`枚举的第一个常量映射为零：每个枚举定义**必须**包含一个映射为零的常量作为其第一个元素。这是因为：

-   必须有一个零值，以便我们可以使用0作为数字[默认值](https://developers.google.com/protocol-buffers/docs/proto3#default)。
-   零值必须是第一个元素，以便与[proto2](https://developers.google.com/protocol-buffers/docs/proto)语义兼容，其中第一个枚举值始终是默认值。

您可以通过将相同的值分配给不同的枚举常量来定义别名。为此，您需要将该`allow_alias`选项设置为`true`，否则编译其会在找到别名时生成一条错误消息。
```
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```
枚举器常量必须在32位整数范围内。由于`enum`值使用[varint编码](https://developers.google.com/protocol-buffers/docs/encoding)，对负值编码效率不高，因此不建议使用负值。您可以在消息中定义`enum`，如上面的示例所示，也可以在外部定义`enum`-这些枚举类型可以在`.proto`文件中的任何消息定义中重复使用。您还可以使用`MessageType.EnumType`语法将一条消息中声明的`enum`类型用作另一条消息中字段的类型。

当您使用 protobuf 编译器编译定义了`enum`类型的`.proto`文件上时，将为 Java或C++ 生成对应的`enum`代码，为Python生成一个特殊类`EnumDescriptor`，用于在运行时生成的类中创建带有整数值的符号常量集。

在反序列化期间，无法识别的枚举值将保留在消息中，尽管在反序列化消息时如何表示该值取决于语言。在支持超出指定符号范围的值的开放式枚举类型的语言（例如C++和Go）中，未知的枚举值仅存储为其基础整数表示形式。在具有封闭枚举类型的语言（例如Java）中，枚举中的大小写用于表示无法识别的值，并且可以使用特殊的访问器访问基础整数。无论哪种情况，如果消息已序列化，则无法识别的值仍将与消息一起序列化。

有关如何在应用程序中使用`enum`的更多信息，请参见针对所选语言的[生成的代码指南](https://developers.google.com/protocol-buffers/docs/reference/overview)。

### 保留值

如果通过完全删除枚举条目或将其注释掉来[更新](https://developers.google.com/protocol-buffers/docs/proto3#updating)枚举类型，则将来的用户在对类型进行更新时可以重用数值，但如果他们以后再加载`.proto`的旧版本时，可能会导致严重的问题，包括数据损坏，隐私错误等。确保不会发生这种情况的一种方法是，将已删除的条目的数字值（和/或名称，也可能导致JSON序列化的问题）指定为`reserved`。如果将来有任何用户尝试使用这些标识符，则 protobuf 编译器将告警提示。您可以使用`max`关键字指定保留的数值范围可达到最大可能值。
```
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```
请注意，您不能在同一条`reserved`语句中混合使用字段名和数字值。

## 使用其他消息类型

您可以使用其他消息类型作为字段类型。例如，假设你想每个`SearchResponse`消息都包括`Result`消息类型-要做到这一点，你可以在同一个`.proto`定义一个`Result`消息类型，然后在`SearchResponse`中指定`Result`类型的字段：
```
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```
### 导入定义

在上面的示例中，`Result`消息类型与`SearchResponse`定义在同一文件中–如果要用作字段类型的消息类型已在另一个`.proto`文件中定义，该怎么办？

您可以通过_导入_其他`.proto`文件来使用它们的定义。要导入另一个`.proto`的定义，请在文件顶部添加一个import语句：
```
import "myproject/other_protos.proto";
```
默认情况下，您只能使用直接导入`.proto`文件中的定义。但是，有时您可能需要将`.proto`文件移动到新位置。现在，您可以直接在原位置放置一个虚拟`.proto`文件，并使用`import public`概念将所有导入此文件请求都转发到新位置，而不是直接移动文件并一次更改所有导入站点。任何导入包含`import public`语句的原型的人都可以传递地依赖依赖项。例如：
```
// new.proto
// All definitions are moved here
```
```
// old.proto
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
```
```
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

协议编译器使用`-I`/`--proto_path`标志在协议编译器命令行上指定的一组目录中搜索导入的文件。如果未给出标志，它将在调用编译器的目录中查找。通常，您应该将`--proto_path`标志设置为项目的根目录，并对所有导入使用完全限定的名称。

### 使用proto2消息类型

可以导入[proto2](https://developers.google.com/protocol-buffers/docs/proto)消息类型并在proto3消息中使用它们，反之亦然。但是，不能在proto3语法中直接使用proto2枚举（如果导入的proto2消息使用它们，也可以）。

## 嵌套类型

您可以在其他消息类型内定义和使用消息类型，如以下示例所示–在此处，`Result`消息是在`SearchResponse`消息内定义的：
```
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```
如果要在其父消息类型之外重用此消息类型，则将其称为：`Parent.Type`
```
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```
您可以根据需要深度嵌套消息：
```
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```
## 更新消息类型

如果现有的消息类型不再满足您的所有需求（例如，您希望消息格式具有一个额外的字段），但是您仍然希望使用以旧格式创建的代码，请不要担心！在不破坏任何现有代码的情况下更新消息类型非常简单。只要记住以下规则：

-   不要更改任何现有字段的字段编号。
-   如果添加新字段，仍可以使用新生成的代码来解析使用“旧”消息格式序列化的任何消息。您应记住这些元素的[默认值](https://developers.google.com/protocol-buffers/docs/proto3#default)，以便新代码可以与旧代码生成的消息正确交互。同样，由新代码创建的消息也可以由旧代码解析：旧的二进制文件在解析时只会忽略新字段。有关详细信息，请参见“[未知字段”](https://developers.google.com/protocol-buffers/docs/proto3#unknowns)部分。
-   只要在更新的消息类型中不再使用字段号，就可以删除字段。您可能想要改名字段，或者添加前缀“ OBSOLETE\_”，或者将字段编号[保留](https://developers.google.com/protocol-buffers/docs/proto3#reserved)，以便将来的用户`.proto`不会意外地重用该编号。
-   `int32`，`uint32`，`int64`，`uint64`，和`bool`都是兼容的-这意味着你可以在这些类型间转换而不破坏前向或向后兼容。如果从消息中解析出一个数字但不匹配对应指定类型，则将获得与在C++中将该数字强制转换为该类型一样的效果（例如，如果将64位数字读取为int32，它将被截断为32位）。
-   `sint32`和`sint64`彼此兼容，但与其他整数类型_不_兼容。
-   `string`和`bytes`只要字节是有效的UTF-8即可兼容。
-   `bytes`如果字节包含消息的编码版本，则嵌套消息与之兼容。
-   `fixed32`与`sfixed32`，`fixed64`和`sfixed64`兼容。
-   `enum`与`int32`，`uint32`，`int64`，和`uint64`兼容，（请注意，不适合的值将被截断）。但是，请注意，客户端代码在反序列化消息时可能会以不同的方式对待它们：例如，无法识别的proto3 `enum`类型将保留在消息中，但是反序列化消息时如何表示这取决于语言。Int字段始终只是保留其值。
-   将单个值更改为**新** 值的成员`oneof`是安全且二进制兼容的。如果您确定没有代码一次设置多个字段，那么将多个字段移动到新`oneof`字段中可能是安全的。将任何字段移到现有`oneof`字段中都是不安全的。

## 未知字段

未知字段是格式正确的 protobuf 序列化数据，表示解析器无法识别的字段。例如，当旧二进制文件使用新字段解析新二进制文件发送的数据时，这些新字段将成为旧二进制文件中的未知字段。

最初，proto3消息在解析过程中始终丢弃未知字段，但是在版本3.5中，我们重新引入了保留未知字段以匹配proto2行为的功能。在版本3.5和更高版本中，未知字段将在解析期间保留并包含在序列化输出中。

## 任何

**翻译保存点**
`Any`消息类型，可以让你使用消息作为嵌套类型，而不必自己定义.proto。一个含有`Any`的序列化消息`bytes`，以充当一个全局唯一标识符和解析为消息的类型的URL一起。要使用该`Any`类型，您需要[导入](https://developers.google.com/protocol-buffers/docs/proto3#other) `google/protobuf/any.proto`。

    导入“ google / protobuf / any.proto” ；消息ErrorStatus { 字符string消息= 1 ;   重复谷歌。protobuf的。任何细节= 2 ; }++IAMASPACE++++IAMASPACE++  ++IAMASPACE++++IAMASPACE++

给定消息类型的默认类型URL为。`type.googleapis.com/packagename.messagename`

不同的语言实现将支持运行时库佣工类型安全的方式打包和解包的任何值-例如，在Java中，任何类型都会有特殊`pack()`和`unpack()`存取，而在C ++中有`PackFrom()`和`UnpackTo()`方法：

    //在Any中存储任意消息类型。NetworkErrorDetails 详细信息= ...; ErrorStatus 状态; 地位。add_details （）-> PackFrom （details ）; //从Any读取任意消息。ErrorStatus 状态= ...; 对于（const的任何及细节：状态。细节（））{ 如果（细节。是< NetworkErrorDetails >（））{ NetworkErrorDetails++IAMASPACE++++IAMASPACE++++IAMASPACE++++IAMASPACE++++IAMASPACE++  ++IAMASPACE++++IAMASPACE++    network_error ;     细节。UnpackTo （＆network_error ）; ...正在处理network_error ... } }      

**当前，正在开发用于任何类型的运行时库**。

如果您已经熟悉[proto2语法](https://developers.google.com/protocol-buffers/docs/proto)，则Any类型将替换[扩展名](https://developers.google.com/protocol-buffers/docs/proto#extensions)。

## 一个

如果您有一则消息包含许多字段，并且最多可以同时设置一个字段，则可以使用oneof功能强制执行此行为并节省内存。

除了共享内存中的所有字段外，一个字段与常规字段类似，并且最多可以同时设置一个字段。设置oneof中的任何成员会自动清除所有其他成员。您可以根据所选择的语言，使用特殊`case()`或`WhichOneof()`方法来检查其中一个设置的值（如果有）。

### 使用Oneof

要在您的文件中定义oneof，请`.proto`使用`oneof`关键字，后跟您的oneof名称，在这种情况下`test_oneof`：

    消息SampleMessage {   testofoneof { 字符string名称= 4 ; SubMessage sub_message = 9 ; } }++IAMASPACE++    ++IAMASPACE++    ++IAMASPACE++  

然后，将oneof字段添加到oneof定义。您可以添加任何类型的字段，但不能使用`repeated`字段。

在您生成的代码中，ofof字段具有与常规字段相同的getter和setter。您还将获得一种特殊的方法来检查oneof中的哪个值（如果有）。您可以在相关[API参考中](https://developers.google.com/protocol-buffers/docs/reference/overview)找到有关所选语言的oneof API的更多信息。

### 功能之一

-   设置oneof字段将自动清除oneof的所有其他成员。因此，如果您设置几个字段中的一个，则仅您设置的_最后一个_字段仍将具有值。

        SampleMessage 消息; 消息。set_name （“ name” ）; CHECK （消息。has_name （））; 消息。mutable_sub_message （）; //将清除名称字段。CHECK （！消息。has_name （））;++IAMASPACE++  

-   如果解析器在线路上遇到同一个对象的多个成员，则在解析的消息中仅使用最后看到的成员。
-   一个不能是`repeated`。
-   反射API适用于其中一个字段。
-   如果将oneof字段设置为默认值（例如将int32 oneof字段设置为0），则将设置该oneof字段的“大小写”，并且该值将在线路上序列化。
-   如果您使用的是C ++，请确保您的代码不会导致内存崩溃。以下示例代码将崩溃，因为`sub_message`已通过调用该`set_name()`方法将其删除。

        SampleMessage 消息; SubMessage * sub_message = 消息。mutable_sub_message （）; 消息。set_name （“ name” ）; //将删除sub_message sub_message - > SET_ ... //这里崩溃++IAMASPACE++     ++IAMASPACE++           

-   再次在C ++中，如果`Swap()`与oneofs两个消息，每个消息将结束与对方的oneof情况下：在下面的例子中，`msg1`将具有`sub_message`与`msg2`将有一个`name`。

        SampleMessage msg1 ; msg1 。set_name （“ name” ）; SampleMessage msg2 ; msg2 。mutable_sub_message （）; msg1 。交换（＆msg2 ）; CHECK （MSG1 。has_sub_message （））; CHECK （MSG2 。has_name （））;

### 向后兼容问题

添加或删除字段之一时请多加注意。如果检查oneof的值返回`None`/`NOT_SET`，则可能意味着尚未设置oneof或已将其设置为oneof的不同版本中的字段。由于无法知道导线上的未知字段是否是oneof的成员，因此无法分辨出差异。

#### 标签重用问题

-   **将字段移入或移出oneof**：在对消息进行序列化和解析后，您可能会丢失一些信息（某些字段将被清除）。但是，您可以安全地将单个字段移动到**新**字段中，并且如果知道只设置了一个字段，则可以移动多个字段。
-   **删除一个oneof字段并将其添加回**：序列化和解析邮件后，这可能会清除您当前设置的oneof字段。
-   **拆分或合并其中一个**：与移动常规字段有类似的问题。

## 地图

如果要在数据定义中创建关联映射，则 protobuf 提供了方便的快捷方式语法：
```
map < key_type ，value_type > map_field = N ;
```
...其中`key_type`可以是任何整数或字符string类型（除浮点类型和`bytes`以外的任何[标量](https://developers.google.com/protocol-buffers/docs/proto3#scalar)类型）。请注意，枚举类型是无效的`key_type`。`value_type`可以是除另一`map`外的任何类型。

因此，例如，如果您想创建一个项目`map`，其中每个`Project`消息都与一个字符string键相关联，则可以这样定义它：
```
map<string, Project> projects = 3;
```
-   `map`字段不能为`repeated`。
-   `map`值的排序和`map`迭代排序是不确定的，因此您不能依赖于`map`的特定顺序。
-   为`.proto`生成文本格式时，`map`会按键排序。数字键按数字排序。
-   对进行消息解析或合并时，如果存在重复的映射键，则使用最后看到的键。从文本格式解析到`map`时，如果键重复，则解析可能会失败。
-   如果为映射字段提供键但没有值，则序列化字段时的行为取决于语言。在C++，Java和Python中，类型的默认值是序列化的，而在其他语言中，则没有序列化的值。

生成的映射API当前可用于所有proto3支持的语言。您可以在相关[API参考中](https://developers.google.com/protocol-buffers/docs/reference/overview)找到有关所选语言的map API的更多信息。

### 向后兼容

映射语法与线上的以下语法等效，因此不支持映射的 protobuf 实现仍可以处理您的数据：
```
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```
任何支持映射的 protobuf 实现都必须产生并接受可以被上述定义接受的数据。

## 配套

您可以向`.proto`文件添加可选的`package`说明符，以防止协议消息类型之间的名称冲突。
```
package foo.bar;
message Open { ... }
```
然后，可以在定义消息类型的字段时使用`package`说明符：
```
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```
`package`说明符影响生成的代码的方式取决于您选择的语言：

-   在**C ++中**，生成的类包装在C++名称空间中。例如，`Open`将在`foo::bar`命名空间中。
-   在**Java中**，除非您`在`.proto`文件中明确提供了`option java_package，否则该包将用作Java包。
-   在**Python中**，package指令将被忽略，因为Python模块是根据其在文件系统中的位置进行组织的。
-   在**Go中**，除非您在`.proto`文件中明确提供`option go_package`，否则该包将用作Go包名称。
-   在**Ruby中**，生成的类被包装在嵌套的Ruby名称空间中，并转换为所需的Ruby大写样式（首字母大写；如果首字符不是字母，`PB_`则为前缀）。例如，`Open`将在`Foo::Bar`命名空间中。
-   在**C＃中**，除非您在`.proto`文件中明确提供`option csharp_namespace`，否则在转换为PascalCase后，该程序包将用作命名空间。例如，`Open`将在`Foo.Bar`命名空间中。

### 软件包和名称解析

protobuf 语言中的类型名称解析类似于C++：首先搜索最内层的作用域，然后搜索下一个最内层的作用域，依此类推，每个包都被视为其父包“内在”的。头部的“.”（例如，`.foo.bar.Baz`）表示从最外面的范围开始。

 protobuf 编译器通过解析导入的`.proto`文件来解析所有类型名称。每种语言的代码生成器都知道如何引用该语言中的每种类型，即使它具有不同的范围规则。

## 定义服务

如果要将消息类型与RPC（远程过程调用）系统一起使用，则可以在`.proto`文件中定义RPC服务接口，并且 protobuf 编译器将以您选择的语言生成服务接口代码和存根。因此，例如，如果您想使用一种方法来定义RPC服务，该方法接受您的方法`SearchRequest`并返回`SearchResponse`，则可以在`.proto`文件中定义它，如下所示：
```
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```
与 protobuf 一起使用的最简单的RPC系统是[gRPC](https://grpc.io/)：这是Google开发的与语言和平台[无关的](https://grpc.io/)开源RPC系统。gRPC与 protobuf 配合使用特别好，并允许您`.proto`使用特殊的 protobuf 编译器插件直接从文件中生成相关的RPC代码。

如果您不想使用gRPC，也可以在自己的RPC实现中使用 protobuf 。您可以在《[Proto2语言指南》中](https://developers.google.com/protocol-buffers/docs/proto#services)找到有关此内容的更多信息。

还有许多正在进行的第三方项目正在为 protobuf 开发RPC实现。有关我们知道的项目的链接列表，请参阅[第三方加载项Wiki页面](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md)。

## JSON对应

Proto3支持JSON中的规范编码，使在系统之间共享数据更加容易。下表按类型对编码进行了描述。

如果JSON编码的数据中缺少某个值，或者如果该值为`null`，则在解析为 protobuf 时，它将被解释为适当的[默认值](https://developers.google.com/protocol-buffers/docs/proto3#default)。如果字段在 protobuf 中具有默认值，则默认情况下会在JSON编码的数据中将其省略以节省空间。一个实现可以提供选项，以在JSON编码的输出中发出具有默认值的字段。

| 原型3                  | JSON格式    | JSON范例                             | 笔记                                                                                                                                                                      |
| -------------------- | --------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 信息                   | 宾语        | `{"fooBar": v,"g": null,…}`        | 生成JSON对象。消息字段名称被映射到lowerCamelCase并成为JSON对象键。如果`json_name`指定了field选项，则将使用指定的值作为键。解析器接受lowerCamelCase名称（或该`json_name`选项指定的名称）和原始原型字段名称。`null`是所有字段类型的可接受值，并被视为相应字段类型的默认值。 |
| 枚举                   | string         | `"FOO_BAR"`                        | 使用在proto中指定的枚举值的名称。解析器接受枚举名称和整数值。                                                                                                                                       |
| 映射&lt;K，V>           | 宾语        | `{"k": v,…}`                       | 所有键都转换为字符string。                                                                                                                                                             |
| 重复的V                 | 数组        | `[v,…]`                            | `null`被接受为空列表\[]。                                                                                                                                                       |
| bool                   | 真假        | `true,false`                       |                                                                                                                                                                         |
| string                    | string         | `"Hello World!"`                   |                                                                                                                                                                         |
| 个字节                  | base64字符string | `"YWJjMTIzIT8kKiYoKSctPUB+"`       | JSON值将是使用带有填充的标准base64编码编码为字符string的数据。接受带/不带填充的标准或URL安全base64编码。                                                                                                            |
| int32，fixed32，uint32 | 数         | `1,-10,0`                          | JSON值为十进制数字。接受数字或字符string。                                                                                                                                                   |
| int64，fixed64，uint64 | string         | `"1","-10"`                        | JSON值将是一个十进制字符string。接受数字或字符string。                                                                                                                                               |
| 浮点double                  | 数         | `1.1,-10.0,0,"NaN","Infinity"`     | JSON值将是数字或特殊字符string值“ NaN”，“ Infinity”和“ -Infinity”之一。接受数字或字符string。指数表示法也被接受。                                                                                                   |
| 任何                   | `object`  | `{"@type": "url","f": v,… }`       | 如果Any包含具有特殊JSON映射的值，则将按以下方式进行转换：。否则，该值将转换为JSON对象，并将插入该字段以指示实际的数据类型。`{"@type": xxx,"value": yyy}``"@type"`                                                               |
| 时间戳记                 | string         | `"1972-01-01T10:00:20.021Z"`       | 使用RFC 3339，其中生成的输出将始终进行Z归一化，并使用0、3、6或9个小数位。也可以接受“ Z”以外的偏移。                                                                                                              |
| 持续时间                 | string         | `"1.000340012s","1s"`              | 生成的输出始终包含0、3、6或9个小数位数，具体取决于所需的精度，后跟后缀“ s”。可接受的任何小数位数（也无），只要它们适合纳秒精度且需要后缀“ s”即可。                                                                                         |
| 结构                   | `object`  | `{ … }`                            | 任何JSON对象。请参阅。`struct.proto`                                                                                                                                             |
| 包装类型                 | 各种类型      | `2,"2","foo",true,"true",null,0,…` | 包装器在JSON中使用与包装后的原始类型相同的表示形式，但`null`在数据转换和传输期间允许并保留该表示形式。                                                                                                                |
| 现场面具                 | string         | `"f.fooBar,h"`                     | 请参阅。`field_mask.proto`                                                                                                                                                  |
| ListValue            | 数组        | `[foo,bar,…]`                      |                                                                                                                                                                         |
| 值                    | 值         |                                    | 任何JSON值                                                                                                                                                                 |
| 空值                   | 空值        |                                    | JSON空                                                                                                                                                                   |
| 空的                   | 宾语        | {}                                 | 空的JSON对象                                                                                                                                                                |

### JSON选项

一个proto3 JSON实现可以提供以下选项：

-   **发出具有默认值的**字段：默认情况下，proto3 JSON输出中省略**具有默认值的**字段。一个实现可以提供一个选项，以使用其默认值覆盖此行为和输出字段。
-   **忽略未知字段**：默认情况下，Proto3 JSON解析器应拒绝未知字段，但可以提供在解析时忽略未知字段的选项。
-   **使用proto字段名称而不是lowerCamelCase名称**：默认情况下，proto3 JSON打印机应将字段名称转换为lowerCamelCase并将其用作JSON名称。一个实现可以提供一个选项，改为使用原型字段名称作为JSON名称。Proto3 JSON解析器需要接受转换后的lowerCamelCase名称和原型字段名称。
-   **将枚举值作为整数而不是字符string发送**：JSON输出中默认使用枚举值的名称。可以提供一个选项来代替使用枚举值的数字值。

## 选件

`.proto`文件中的各个声明可以使用许多_选项_来注释。选项不会改变声明的整体含义，但可能会影响在特定上下文中处理声明的方式。可用选项的完整列表在中定义`google/protobuf/descriptor.proto`。

一些选项是文件级选项，这意味着它们应该在顶级范围内编写，而不是在任何消息，枚举或服务定义内。一些选项是消息级别的选项，这意味着它们应该写在消息定义中。一些选项是字段级选项，这意味着它们应在字段定义中编写。选项也可以写在枚举类型，枚举值，服务类型和服务方法上。但是，目前对于这些功能都不存在有用的选项。

以下是一些最常用的选项：

-   `java_package`（文件选项）：您要用于生成的Java类的包。如果文件中未`java_package`指定任何显式选项`.proto`，则默认情况下将使用proto软件包（在`.proto`文件中使用“ package”关键字指定）。但是，proto软件包通常不能成为良好的Java软件包，因为proto软件包不应以反向域名开头。如果未生成Java代码，则此选项无效。

        选项java_package =“ com.example.foo”;

-   `java_multiple_files`（文件选项）：导致在包级别定义顶级消息，枚举和服务，而不是在以.proto文件命名的外部类中定义。
-       选项java_multiple_files = true;
-   `java_outer_classname`（文件选项）：您要生成的最外层Java类的类名（以及文件名）。如果`java_outer_classname`在`.proto`文件中未指定任何显式名称，则通过将`.proto`文件名转换为驼峰大小写来构造类名（因此`foo_bar.proto`成为`FooBar.java`）。如果未生成Java代码，则此选项无效。

        选项java_outer_classname =“ Ponycopter”;

-   `optimize_for`（文件选项）：可以设置为`SPEED`，`CODE_SIZE`或`LITE_RUNTIME`。这会以下列方式影响C ++和Java代码生成器（可能还有第三方生成器）：

    -   `SPEED`（默认）： protobuf 编译器将生成代码，用于对消息类型进行序列化，解析和执行其他常见操作。此代码已高度优化。
    -   `CODE_SIZE`： protobuf 编译器将生成最少的类，并将依赖于基于反射的共享代码来实现序列化，解析和其他各种操作。因此，生成的代码将比使用的代码小得多`SPEED`，但是操作会更慢。类仍将实现与`SPEED`模式下完全相同的公共API。此模式在包含大量`.proto`文件且不需要所有文件快速快速运行的应用程序中最有用。
    -   `LITE_RUNTIME`： protobuf 编译器将生成仅依赖于“精简版”运行时库（`libprotobuf-lite`而不是`libprotobuf`）的类。精简版运行时比完整库要小得多（大约小一个数量级），但省略了某些功能，例如描述符和反射。这对于在受限平台（如手机）上运行的应用程序特别有用。编译器仍将像在`SPEED`模式下一样快速生成所有方法的实现。生成的类将仅以`MessageLite`每种语言实现接口，这仅提供完整`Message`接口方法的子集。

        选项optimize_for = CODE_SIZE；

-   `cc_enable_arenas`（文件选项）：启用C ++生成代码的[舞台分配](https://developers.google.com/protocol-buffers/docs/reference/arenas)。
-   `objc_class_prefix`（文件选项）：设置Objective-C类的前缀，该前缀将添加到所有Objective-C生成的类以及此.proto枚举。没有默认值。您应该使用[Apple推荐的](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4)3-5个大写字符之间的前缀。请注意，Apple保留所有2个字母前缀。
-   `deprecated`（字段选项）：如果设置为`true`，则表明该字段已弃用，并且不应被新代码使用。在大多数语言中，这没有实际效果。在Java中，这成为`@Deprecated`注释。将来，其他特定于语言的代码生成器可能会在字段的访问器上生成弃用注释，这反过来将导致在编译尝试使用该字段的代码时发出警告。如果该字段未被任何人使用，并且您想阻止新用户使用该字段，请考虑使用[保留](https://developers.google.com/protocol-buffers/docs/proto3#reserved)语句替换该字段声明。

        int32 old_field = 6 [deprecated = true];

### 自订选项

 protobuf 还允许您定义和使用自己的选项。这是大多数人不需要的**高级功能**。如果您确实需要创建自己的选项，请参阅《[Proto2语言指南》](https://developers.google.com/protocol-buffers/docs/proto.html#customoptions)以了解详细信息。请注意，创建自定义选项使用[扩展名](https://developers.google.com/protocol-buffers/docs/proto.html#extensions)，只有proto3中的自定义选项才允许使用[扩展名](https://developers.google.com/protocol-buffers/docs/proto.html#extensions)。

## 生成课程

要生成，你需要工作，在规定的消息类型的使用Java，Python，C ++，围棋，红宝石，Objective-C的，或C＃代码`.proto`文件，你需要运行协议缓冲编译器`protoc`上`.proto`。如果尚未安装编译器，请[下载软件包](https://developers.google.com/protocol-buffers/docs/downloads.html)并按照自述文件中的说明进行操作。对于Go，还需要为编译器安装一个特殊的代码生成器插件：您可以在GitHub上的[golang / protobuf](https://github.com/golang/protobuf/)存储库中找到此代码和安装说明。

协议编译器的调用如下：

    protoc --proto_path = IMPORT_PATH++IAMASPACE++--cpp_out = DST_DIR++IAMASPACE++--java_out = DST_DIR++IAMASPACE++--python_out = DST_DIR++IAMASPACE++--go_out = DST_DIR++IAMASPACE++--ruby_out = DST_DIR++IAMASPACE++--objc_out = DST_DIR++IAMASPACE++--csharp_out = DST_DIR ++IAMASPACE++path / to / file++IAMASPACE++.proto

-   `IMPORT_PATH`指定`.proto`解析`import`指令时要在其中查找文件的目录。如果省略，则使用当前目录。可以通过`--proto_path`多次传递选项来指定多个导入目录。将按顺序搜索它们。可以用作的简写形式。`-I=IMPORT_PATH``--proto_path`
-   您可以提供一个或多个_输出指令_：

    -   `--cpp_out`在中生成C ++代码`DST_DIR`。有关更多信息，请参见[C ++生成的代码参考](https://developers.google.com/protocol-buffers/docs/reference/cpp-generated)。
    -   `--java_out`在中生成Java代码`DST_DIR`。有关更多信息，请参见[Java生成的代码参考](https://developers.google.com/protocol-buffers/docs/reference/java-generated)。
    -   `--python_out`在中生成Python代码`DST_DIR`。有关更多信息，请参见[Python生成的代码参考](https://developers.google.com/protocol-buffers/docs/reference/python-generated)。
    -   `--go_out`在中生成Go代码`DST_DIR`。有关更多信息，请参见[Go生成的代码参考](https://developers.google.com/protocol-buffers/docs/reference/go-generated)。
    -   `--ruby_out`在中生成Ruby代码`DST_DIR`。Ruby生成的代码参考即将推出！
    -   `--objc_out`在中生成Objective-C代码`DST_DIR`。有关更多信息，请参见[Objective-C生成的代码参考](https://developers.google.com/protocol-buffers/docs/reference/objective-c-generated)。
    -   `--csharp_out`在中生成C＃代码`DST_DIR`。有关更多信息，请参见[C＃生成的代码参考](https://developers.google.com/protocol-buffers/docs/reference/csharp-generated)。
    -   `--php_out`在中生成PHP代码`DST_DIR`。有关更多信息，请参见[PHP生成的代码参考](https://developers.google.com/protocol-buffers/docs/reference/php-generated)。

    为了更加方便，如果`DST_DIR`结尾为`.zip`或`.jar`，编译器会将输出写入具有给定名称的单个ZIP格式的存档文件。`.jar`根据Java JAR规范的要求，还将为输出提供清单文件。请注意，如果输出存档已经存在，它将被覆盖；编译器不够智能，无法将文件添加到现有存档中。

-   您必须提供一个或多个`.proto`文件作为输入。`.proto`可以一次指定多个文件。尽管这些文件是相对于当前目录命名的，但是每个文件都必须位于`IMPORT_PATH`s之一中，以便编译器可以确定其规范名称。
