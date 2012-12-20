���ĵ���������һ��C++������淶����̷��û�кû�֮�֡�����㱾����ѭ�Լ���һ�׷�񣬲��Ҿ�ȷ���Կո�����ã���ô����Ժܿ����Ⲣ��ѭ���淶�����淶�ο�/ȡ�Բ���(��Google C++��̹淶��)(https://github.com/brantyoung/zh-google-styleguide/)

# �ļ�

�󲿷�ʱ��һ��ʵ���ļ�(.cpp/.c)�����Ӧһ��ͬ����ͷ�ļ�(.h)�������ų����⣬��������������(main)���ļ���ÿһ���������ļ��ڴ󲿷�ʱ��ֻ����һ����Ķ��弰��ӿ�ʵ�֡��ļ�����֯Ӧ���������ر��ģ��Ļ��֡�

�ļ�����������Ҫ���ش�СдԼ����������ʽʹ�������ĵ��ʲ�������ĸ��д�����磺

    LexParser.h
    LexParser.cpp

## ͷ�ļ�

ͷ�ļ�ʹ��``#define``�������ظ���������ʹ��``#pragma once``����������ʽ�󲿷������ʹ���ļ������ɣ������ڿ��ܴ��ڳ�ͻ���ļ������������ģ�����������ռ�����Ϊ��ǰ׺����������ļ�analyzer/Lex.h��

    #ifndef ANALYZER_LEX_H
    #define ANALYZER_LEX_H
    ...
    #endif // ANALYZER_LEX_H

## ͷ�ļ�����

Ӧ�������ܵؼ���ͷ�ļ�֮����������⼴Ӧ�����ܵ�ʹ��ǰ����������������û��Ҫ������°�����ͷ�ļ�����ʱ��Ϊ��ʵ����һ�㣬������Ҫ�ڴ�����ʹ��ָ������ã������Ƕ������磺

    class LexParser;

    void Parse(LexParser *parser);

## �����ļ��Ĵ���

���ﲢ���ϸ�Ҫ����ĸ��������ͷ�ļ������豣֤���Ȱ���ϵͳ����������⣩ͷ�ļ���Ȼ������Լ�д��ͷ�ļ������磺

    #include <stdio.h>
    #include <stdlib.h>
    #include "LexParser.h"

## ģ��ʵ��

����ģ�����ʵ�ֶ��ԣ����ｨ��ʹ��STL�ķ�ʽ������ģ����ӿڵ�ʵ�ִ���ֱ�ӷ������ඨ���У������ǵ���������������ȷʵ��Ҫ���룬Ҳ��ֱ�ӷ����ඨ��֮��ͬ��λ��ͷ�ļ��У����磺

    template <typename T>
    class List
    {
    public:
        void Insert(T &val) {
            ...
        }
    };

# ����Լ��

����Լ���������������и��ַ��ŵ�ƴд�������ǲ�ʹ��������������ͬʱҲ����ʹ��һ������СдӢ����ĸ���»��ߵļ��˷�ʽ��һ��ֵ��һ���ԭ���ǣ�������һ�������ļ�����ӻ��޸Ĵ���ʱ�������Ĵ���������ʽӦ�����Ѵ��ڵĴ�����߶��Ǻϡ�

## ͨ����������

���ڱ������������ࡢ�����ռ䡢�ļ��ȶ�ʹ�������ĵ�����ϣ�����֮�䲻ʹ���»��ߡ�������Ŀ�����ǻὫ����ע�������ھ��нӿ����ʵĴ��뵥Ԫ������ͷ�ļ����๫���ӿڡ�ģ��������ͽӿڣ���ˣ�����Щ���뵥Ԫ�����������Ե���Ϊ��Ҫ����˶�Ӧ�����ڽӿ�ʵ���еĹ������ǻ�ſ����ơ�����ΪһЩ����Լ����������

    LexParser.cpp
    class LexParser;
    int m_level;
    int m_objCount;
    namespace Analyzer;
    int g_analyzer;

��������ʹ�õ�����д�����뾡����֤��д�ʹ�ҵ���ʶһ�£����磺

    contrl: ctrl
    manager: mgr
    list: lst
    count: cnt
    level: lvl
    variable: var

�ž�ʹ��ƴ�������������ʵ���Ҳ���һ��ͨ�׵�Ӣ�ĵ��ʣ�����һ����Ƨ�ĵ���Ҳ��ƴ���á�

## ��������

��������ÿ�������Դ�д��ĸ��ͷ���������»��ߣ�MyExcitingClass��MyExcitingEnum�������������������ࡢ�ṹ�塢���Ͷ��壨typedef����ö�١���ʹ����ͬԼ�������磺

    // classes and structs
    class UrlTable { ...
    class UrlTableTester { ...
    struct UrlTableProperties { ...
    // typedefs
    typedef hash_map<UrlTableProperties *, string> PropertiesMap;
    // enums
    enum UrlTableErrors { ...

**ע�⣬��Ϊ���ǲ�ʹ�����������������ԣ���Ҫ����CMyExictingClass��SUrlTableProperties���������֡�**

## ��������

��������ͬ��ÿ�������Դ�д��ĸ��ͷ������һ����ĸ��ҪСд�����ң����ڳ�Ա������ȫ�ֱ�������̬��������Ҫʹ�������ǰ׺���������ʣ����磺

    int m_level; // ��Ա����
    int g_objCount; // ȫ�ֱ���
    int s_cnt; // �ֲ�����

## ��������

��������ÿ�������Դ�д��ĸ��ͷ�����飺 ���õĺ���������Ȼ���Զ��ʿ�ͷ������``Get/Is``һ��ĺ�������ʵ�ֲ�Ҫ��**��ȡ**֮��ĸ����á�

## ������������

�����ͺ��������ʹ��ȫ��д������֮�����»��߷ָ���磺

    #define MAX_COUNT 256
    const int MAX_COUNT = 256;

����ʹ�÷�ʽ���ƺ����ĺ꣬�����������ѭ���������������磺

    #define Max(a, b) ((a) > (b) ? (a) : (b))

## ö������

ö��ֵӦȫ����д�����ʼ����»���������MY_EXCITING_ENUM_VALUE��ö�������������ͣ���˴�Сд��ϣ�UrlTableErrors��

    enum UrlTableErrors {
        OK = 0,
        ERROR_OUT_OF_MEMORY,
        ERROR_MALFORMED_INPUT,
    };

## ������������

������������C/C++ʵ�����ƵĶ���ʱ���ɲο���������Լ����

* bigopen()�����������ο�open()
* uint��typedef ���Ͷ���
* sparse_hash_map��STL ����ʵ�壻�ο�STL ����Լ��


# ��ʽ

����һ�ֱ�̷����ԣ��������������⣬�����ʽ����һ����Ҫ�Ĺ淶��

## �г���

�����г��ȣ�����û��һ��׼ȷ�����֣�����80�С�һ����Ҫ��ԭ���ǣ���Ҫ�ñ༭�����ֺ������������������ʾ��������ʾ��һ�д��롣�����ʹ�õ���22��Ŀ�����ʾ����Ҳ��ҪΪ����Ĳ��ӡ��г��Ⱦ�������ʾ���ϴ�����������۳̶ȣ����Բ�Ҫ����ĳһ���ر𳤣�Ҳ��ò�Ҫ����������̫խ�������

## �ո���Ʊ��(TAB)

ֻʹ�ÿո�ÿ������4���ո񡣲�Ҫ�ô����г���TAB�ַ����趨�༭����TABתΪ�ո�

## ���������붨��

�������ͺͺ�������ͬһ�У����ʵĻ�������Ҳ����ͬһ�С� ��������ȥ��������

    ReturnType ClassName::FunctionName(Type par_name1, Type par_name2) {
        DoSomething();
        ...
    }

���ͬһ���ı��϶࣬�ݲ������в�����

    ReturnType ClassName::ReallyLongFunctionName(Type par_name1,
        Type par_name2,
        Type par_name3) {
        DoSomething();
        ...
    }

��������һ���������Ų��£�

    ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
            Type par_name1, // 8 space indent
            Type par_name2,
            Type par_name3) {
        DoSomething(); // 4 space indent
        ...
    }

ע�����¼��㣺

* ����ֵ���Ǻͺ�������ͬһ�У�
* ��Բ���ţ�open parenthesis�����Ǻͺ�������ͬһ�У�
* ����������Բ���ż�û�пո�
* Բ�����������û�пո�
* ������ţ�open curly brace���������һ������ͬһ�е�ĩβ����
* �Ҵ����ţ�close curly brace�����ǵ���λ�ں������һ�У�
* ��Բ���ţ�close parenthesis����������ż�������һ���ո�
* ����������ʵ�ִ��������β����Ʊ��뱣��һ�£�
* �����β�Ӧ�����ܶ��룻
* ȱʡ����Ϊ4���ո�
* ������װ�Ĳ�������8���ո��������

�������Ϊconst �ģ��ؼ���const Ӧ�����һ������λ��ͬһ�С�

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

�����Щ����û���õ����ں������崦��������ע������

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

## �������� 

��������ͬһ�У����򣬽�ʵ�η�װ��Բ�����С�����������ѭ������ʽ��

    bool retval = DoSomething(argument1, argument2, argument3);

���ͬһ�зŲ��£��ɶ�Ϊ���У�����ÿһ�ж��͵�һ��ʵ�ζ��룬��Բ���ź����Բ����ǰ��Ҫ���ո�

    bool retval = DoSomething(averyveryveryverylongargument1,
                              argument2, argument3);

������������Ƚ϶࣬���Գ��ڿɶ��ԵĿ���ÿ��ֻ��һ��������

    bool retval = DoSomething(argument1,
                              argument2,
                              argument3,
                              argument4);

���������̫���������ڳ�������󳤶ȣ����Խ����в����������У�

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

## ����/ѭ�����

ע��ո��ʹ�ã�����Ҳ���١�

    if (condition) { // ������û�пո�
        ...
    } else { // elseλ�ڴ�����ͬ��
        ...
    }

����ֻ��һ�д����``if``������Խ�����ڴ������У�Ҳ���Բ��ţ����������Խ��������������ж�����ͬһ�С�

����ѭ����

    for (int i = 0; i < MAX; ++i) {
        ...
    }
    
    while (condition) {
        ... 
    }

## ָ������ñ��ʽ

��㣨.�����ͷ��->��ǰ��Ҫ�пո�ָ��/��ַ��������*��&����Ҫ�пո�������ָ������ñ��ʽ����ȷ������

    x = *p;
    p = &x;
    x = r.y;
    x = r->y;

ע�⣺

* �ڷ��ʳ�Աʱ�������ͷǰ��û�пո�
* ָ�������*��&��û�пո�

������ָ����������ʱ���Ǻ���Ҫ�������������

    char *c;
    const string &str;

 
## Ԥ����ָ��

Ԥ����ָ�Ҫ�����������׿�ʼ��

## ���ʽ

����������������public:��protected:��private��

    class MyClass : public OtherClass { // ͬ��ע��ո�
    public:
        MyClass();

        void Test();

    private:
        int m_data;
        int m_level;
    };

ע�⣺

* ����һ���ؼ��ʣ�һ����public���⣬�����ؼ���ǰ��һ�У������Ƚ�С�Ļ�Ҳ���Բ���
* ��Щ�ؼ��ʺ�Ҫ����
* public ������ǰ�棬Ȼ����protected ��private
* �ȷų�Ա������Ȼ���ǳ�Ա����

## ����

�ڴ����й��ڿո��ʹ�ñ����ϸ񣬲��ܶ�Ҳ�����٣�����ʹ�÷��Ϲ淶��

    int a; // ע����ֺ�֮�䣬�ֺ������֮�����ϸ���ƿո�
    std::vector<std::list<std::string>> vec; // Ŀǰ�ı������Ѿ�����ʶ�������'>>'������Ҫ��Ӷ���ո�
    int a = 1; 
    for ( ; ptr != NULL; ++ptr) {
    }
    a++;
    if (a > 0 && b == 1 && !inSpace) {
    }

���������ı������壬��ֱ������û�б�Ҫ�ģ�����Ϊ�Ƽ��Ĺ淶��

    int m_abc;
    std::string m_name;
    float m_precision;

���ں�����֮�䣬�������������Ƕ��壬��Ҫ����һ�п��У�

    void func1() {
        ...
    }

    void func2() {
        ...
    }

# ע�� 

�ڱ��淶�У���Щע���Ǳ���ģ�����Щע�����ǿ�ѡ�ġ�ע�Ϳ���ʹ������Ҳ����ʹ��Ӣ�ġ�ע������ͨ�����ڴ�����֮�ϣ��������β���������ڴ�����֮�¡�ע��ʹ��C����ע�ͻ���C++����ע��û�й涨�������ͳһ��

## ͷ�ļ�ע��

ͷ�ļ���ĳ����������˵�������������Խӿڵ�����ְ��ͷ�ļ��е�ע�Ͱ��������֣����ļ������������������������ؼ��ӿڵ��������������ͷ�ļ�LexParser.h��

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

�����ļ���������������˵�����ݿ����ظ�������Բ����ļ�����������ͨ��Ϊͷ�ļ�����һ��ע��ͷ�����ۣ��ο�������Ĵ���һ������������Լ��ĳɾ͸У���һ����Ҳ��������Ķ�����֪����˭���ɽ�󣩡��Խӿڵ�ʹ��ע���Ǳ�Ҫ�ġ�

���������ע�ͣ���û�жԸ�ʽ���ض�Ҫ�����簴��doxygen�ĸ�ʽ�������Է������ɵ������ĵ���������ܱ���һ�̶ֹ��ĸ�ʽ��Ҳ��һ����ϰ�ߡ�

## ʵ���ļ�ע��

����ʵ���ļ���CPP�����ԣ�Ҳ��Ҫһ��ע��ͷ�����ٱ���ԭʼ�����������ڹؼ�����ʵĴ���Σ��Ƽ�����ע�ͣ�ע�Ϳ��Է��ڴ����֮�ϣ�Ҳ���Է�����β�����磺

    //
    // file: LexParser.cpp
    // author: Kevin Lynx
    //
    #include "LexParser.h"
    
    void LexParser::Parse(const std::string &file) {
        // some other comments
        ...        
    }



