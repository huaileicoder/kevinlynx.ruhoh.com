本文档用于描述一种C++编码风格规范。编程风格没有好坏之分。如果你本身遵循自己的一套风格，并且精确到对空格的运用，那么你可以很快地理解并遵循本规范。本规范参考/取自部分(《Google C++编程规范》)(https://github.com/brantyoung/zh-google-styleguide/)

# 文件

大部分时候一个实现文件(.cpp/.c)都会对应一个同名的头文件(.h)，但不排除例外，例如包含程序入口(main)的文件。每一对这样的文件在大部分时候只包含一个类的定义及其接口实现。文件的组织应尽量清晰地表达模块的划分。

文件名的命名需要遵守大小写约定，命名方式使用连续的单词并且首字母大写，例如：

    LexParser.h
    LexParser.cpp

## 头文件

头文件使用``#define``来避免重复包含，不使用``#pragma once``。其命名格式大部分情况下使用文件名即可，但对于可能存在冲突的文件名，建议添加模块名或命名空间名作为其前缀，例如对于文件analyzer/Lex.h：

    #ifndef ANALYZER_LEX_H
    #define ANALYZER_LEX_H
    ...
    #endif // ANALYZER_LEX_H

## 头文件依赖

应尽最大可能地减少头文件之间的依赖，意即应尽可能地使用前置声明，而不是在没必要的情况下包含其头文件。有时候为了实现这一点，甚至需要在代码中使用指针而引用，而不是对象，例如：

    class LexParser;

    void Parse(LexParser *parser);

## 包含文件的次序

这里并不严格要求按字母序来包含头文件，但需保证首先包含系统（或第三方库）头文件，然后才是自己写的头文件，例如：

    #include <stdio.h>
    #include <stdlib.h>
    #include "LexParser.h"

## 模板实现

对于模板类的实现而言，这里建议使用STL的方式，即将模板类接口的实现代码直接放置在类定义中，而不是单独抽离出来。如果确实需要抽离，也是直接放在类定义之后，同样位于头文件中，例如：

    template <typename T>
    class List
    {
    public:
        void Insert(T &val) {
            ...
        }
    };

# 命名约定

命名约定用于描述代码中各种符号的拼写规则。我们不使用匈牙利命名，同时也不会使用一连串的小写英文字母加下划线的极端方式。一个值得一提的原则是，当你往一个代码文件中添加或修改代码时，你加入的代码命名方式应该与已存在的代码风格高度吻合。

## 通用命名规则

对于变量、函数、类、命名空间、文件等都使用连续的单词组合，单词之间不使用下划线。整个项目中我们会将更多注意力放在具有接口性质的代码单元，例如头文件、类公共接口、模块对外的类和接口，因此，对这些代码单元的命名规则将显得最为重要。与此对应，对于接口实现中的规则我们会放宽限制。以下为一些符合约定的命名：

    LexParser.cpp
    class LexParser;
    int m_level;
    int m_objCount;
    namespace Analyzer;
    int g_analyzer;

我们允许使用单词缩写，但请尽量保证缩写和大家的认识一致，例如：

    contrl: ctrl
    manager: mgr
    list: lst
    count: cnt
    level: lvl
    variable: var

杜绝使用拼音命名，如果你实在找不到一个通俗的英文单词，查阅一个生僻的单词也比拼音好。

## 类型命名

类型命名每个单词以大写字母开头，不包含下划线：MyExcitingClass、MyExcitingEnum。所有类型命名——类、结构体、类型定义（typedef）、枚举——使用相同约定，例如：

    // classes and structs
    class UrlTable { ...
    class UrlTableTester { ...
    struct UrlTableProperties { ...
    // typedefs
    typedef hash_map<UrlTableProperties *, string> PropertiesMap;
    // enums
    enum UrlTableErrors { ...

**注意，因为我们不使用匈牙利命名，所以，不要出现CMyExictingClass、SUrlTableProperties这样的名字。**

## 变量命名

变量命名同样每个单词以大写字母开头，但第一个字母需要小写。并且，对于成员变量、全局变量、静态变量，需要使用特殊的前缀标明其性质，例如：

    int m_level; // 成员变量
    int g_objCount; // 全局变量
    int s_cnt; // 局部变量

## 函数命名

函数命名每个单词以大写字母开头。建议： 良好的函数命名必然是以动词开头；对于``Get/Is``一类的函数，其实现不要有**获取**之外的副作用。

## 常量、宏命名

常量和宏的命名均使用全大写，单词之间以下划线分割，例如：

    #define MAX_COUNT 256
    const int MAX_COUNT = 256;

对于使用方式类似函数的宏，其命名亦可遵循函数命名规则，例如：

    #define Max(a, b) ((a) > (b) ? (a) : (b))

## 枚举命名

枚举值应全部大写，单词间以下划线相连：MY_EXCITING_ENUM_VALUE。枚举名称属于类型，因此大小写混合：UrlTableErrors。

    enum UrlTableErrors {
        OK = 0,
        ERROR_OUT_OF_MEMORY,
        ERROR_MALFORMED_INPUT,
    };

## 命名规则例外

当命名与现有C/C++实体相似的对象时，可参考现有命名约定：

* bigopen()，函数名，参考open()
* uint，typedef 类型定义
* sparse_hash_map，STL 相似实体；参考STL 命名约定


# 格式

对于一种编程风格而言，除了名字命名外，代码格式是另一项重要的规范。

## 行长度

对于行长度，这里没有一个准确的数字，例如80行。一个重要的原则是，不要让编辑器出现横向滚动条，尽量让显示器可以显示完一行代码。如果你使用的是22寸的宽屏显示器，也不要为难你的脖子。行长度决定你显示器上代码的总体美观程度，所以不要出现某一行特别长，也最好不要出现整体上太窄的情况。

## 空格和制表符(TAB)

只使用空格，每次缩进4个空格。不要让代码中出现TAB字符，设定编辑器将TAB转为空格。

## 函数声明与定义

返回类型和函数名在同一行，合适的话，参数也放在同一行。 函数看上去像这样：

    ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) {
        DoSomething();
        ...
    }

如果同一行文本较多，容不下所有参数：

    ReturnType ClassName::ReallyLongFunctionName(Type par_name1,
        Type par_name2,
        Type par_name3) {
        DoSomething();
        ...
    }

甚至连第一个参数都放不下：

    ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
            Type par_name1, // 8 space indent
            Type par_name2,
            Type par_name3) {
        DoSomething(); // 4 space indent
        ...
    }

注意以下几点：

* 返回值总是和函数名在同一行；
* 左圆括号（open parenthesis）总是和函数名在同一行；
* 函数名和左圆括号间没有空格；
* 圆括号与参数间没有空格；
* 左大括号（open curly brace）总在最后一个参数同一行的末尾处；
* 右大括号（close curly brace）总是单独位于函数最后一行；
* 右圆括号（close parenthesis）和左大括号间总是有一个空格；
* 函数声明和实现处的所有形参名称必须保持一致；
* 所有形参应尽可能对齐；
* 缺省缩进为4个空格；
* 独立封装的参数保持8个空格的缩进。

如果函数为const 的，关键字const 应与最后一个参数位于同一行。

    // Everything in this function signature fits on a single line
    ReturnType FunctionName(Type par) const {
        ...
    }

    // This function signature requires multiple lines, but
    // the const keyword is on the line with the last parameter.
    ReturnType ReallyLongFunctionName(Type par1,
            Type par2) const {
        ...
    }

如果有些参数没有用到，在函数定义处将参数名注释起来

    // Always have named parameters in interfaces.
    class Shape {
    public:
        virtual void Rotate(double radians) = 0;
    }

    // Always have named parameters in the declaration.
    class Circle : public Shape {
    public:
        virtual void Rotate(double radians);
    }

    // Comment out unused named parameters in definitions.
    void Circle::Rotate(double /*radians*/) {}
    // Bad - if someone wants to implement later, it's not clear what the
    // variable means.
    void Circle::Rotate(double) {}

## 函数调用 

尽量放在同一行，否则，将实参封装在圆括号中。函数调用遵循如下形式：

    bool retval = DoSomething(argument1, argument2, argument3);

如果同一行放不下，可断为多行，后面每一行都和第一个实参对齐，左圆括号后和右圆括号前不要留空格：

    bool retval = DoSomething(averyveryveryverylongargument1,
                              argument2, argument3);

如果函数参数比较多，可以出于可读性的考虑每行只放一个参数：

    bool retval = DoSomething(argument1,
                              argument2,
                              argument3,
                              argument4);

如果函数名太长，以至于超过行最大长度，可以将所有参数独立成行：

    if (...) {
        ...
        ...
    if (...) {
        DoSomethingThatRequiresALongFunctionName(
            veryLongArgument1, // 4 space indent
            argument2,
            argument3,
            argument4);
    }

## 条件/循环语句

注意空格的使用，不多也不少。

    if (condition) { // 括号中没有空格
        ...
    } else { // else位于大括号同行
        ...
    }

对于只有一行代码的``if``，你可以将其放在大括号中，也可以不放，你甚至可以将这条语句和条件判定放在同一行。

对于循环：

    for (int i = 0; i < MAX; ++i) {
        ...
    }
    
    while (condition) {
        ... 
    }

## 指针和引用表达式

句点（.）或箭头（->）前后不要有空格，指针/地址操作符（*、&）后不要有空格。下面是指针和引用表达式的正确范例：

    x = *p;
    p = &x;
    x = r.y;
    x = r->y;

注意：

* 在访问成员时，句点或箭头前后没有空格；
* 指针操作符*或&后没有空格。

在声明指针变量或参数时，星号需要与变量名紧挨：

    char *c;
    const string &str;

 
## 预处理指令

预处理指令不要缩进，从行首开始。

## 类格式

声明属性依次序是public:、protected:、private。

    class MyClass : public OtherClass { // 同样注意空格
    public:
        MyClass();

        void Test();

    private:
        int m_data;
        int m_level;
    };

注意：

* 除第一个关键词（一般是public）外，其他关键词前空一行，如果类比较小的话也可以不空
* 这些关键词后不要空行
* public 放在最前面，然后是protected 和private
* 先放成员函数，然后是成员变量

## 留白

在代码中关于空格的使用必须严格，不能多也不能少，以下使用符合规范：

    int a; // 注释与分号之间，分号与代码之间需严格控制空格
    std::vector<std::list<std::string>> vec; // 目前的编译器已经可以识别这里的'>>'，不需要添加额外空格
    int a = 1; 
    for ( ; ptr != NULL; ++ptr) {
    }
    a++;
    if (a > 0 && b == 1 && !inSpace) {
    }

对于连续的变量定义，垂直对齐是没有必要的，以下为推荐的规范：

    int m_abc;
    std::string m_name;
    float m_precision;

对于函数块之间，无论是声明还是定义，需要留出一行空行：

    void func1() {
        ...
    }

    void func2() {
        ...
    }

# 注释 

在本规范中，有些注释是必须的，而有些注释则是可选的。注释可以使用中文也可以使用英文。注释内容通常放在代码行之上，或代码行尾，但不放在代码行之下。注释使用C风格的注释还是C++风格的注释没有规定，但最好统一。

## 头文件注释

头文件从某种意义上来说，担负着描述对接口的描述职责。头文件中的注释包含三部分：对文件的描述，对类的描述，对类关键接口的描述。例如对于头文件LexParser.h：

    //
    // file: LexParser.h
    // author: Kevin Lynx
    // brief: description for this file
    //
    #ifndef LEX_PARSER_H
    #define LEX_PARSRE_H
    
    // 
    // class documentation
    //
    class LexParser {
    public:
        //
        // function documentation
        //
        void Parse(const std::string &file);

        void Func2();
    };

    #endif // LEX_PARSER_H

对于文件描述和类描述来说，内容可能重复，你可以不对文件做描述，但通常为头文件保留一个注释头更美观（何况留下你的大名一方面可以增加自己的成就感，另一方面也方便代码阅读者能知道找谁答疑解惑）。对接口的使用注释是必要的。

对于这里的注释，并没有对格式有特定要求，例如按照doxygen的格式做描述以方便生成单独的文档，但如果能保持一种固定的格式，也是一个好习惯。

## 实现文件注释

对于实现文件（CPP）而言，也需要一个注释头，至少保留原始作者名。对于关键或出彩的代码段，推荐留下注释，注释可以放在代码块之上，也可以放在行尾。例如：

    //
    // file: LexParser.cpp
    // author: Kevin Lynx
    //
    #include "LexParser.h"
    
    void LexParser::Parse(const std::string &file) {
        // some other comments
        ...        
    }



