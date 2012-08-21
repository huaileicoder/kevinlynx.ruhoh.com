--
date: '2012-08-21'
categories: mmo
tags: [chat server]
--

# 聊天服务器设计

## 设计目标

本文旨在设计一个较为通用的聊天服务器。该聊天服务器将尽量独立于具体的游戏项目，不依赖游戏逻辑本身。它将作为一种游戏可使用的组件，通过一些适配代码而用于游戏中。以下讨论中将涉及到游戏客户端（GC）、游戏服务器（GS）以及聊天服务器（CS）等角色，均直接使用简称。

## 基础思想

CS作为一个服务器，它通过自己定义的协议，来完成聊天信息的转发服务。基于此，GC和GS都将充当CS客户端的角色。在各种聊天动作中，都有聊天个体和聊天组的概念。个体即为玩家，而组则是聊天个体集合，例如临时队伍、工会、甚至临时建立的多人聊天房间。当个体在组中发出聊天信息时，CS将广播此消息到该组的所有成员。组的建立由GS向CS发起请求完成，而GC一般没有建立组的权限。因此，我们看到GC和GS虽然都作为CS的网络客户端，但其所做动作却不相同。因此，我们将CS的客户端按权限分为不同的类型。

### 概念

按照以上描述，整个系统中包含以下概念：

* 实体(Entity)，CS的每一个网络客户端就是一个实体，实体有不同的类型，不同类型的实体在CS上可做的动作不同，实体的建立按照其类型不同也不同，例如一般而言，客户端可直接在CS上成为一个实体，但GS则可能需要验证IP才能成为一个实体。每一个实体也是一个聊天信息的发送、接收者。
* 组(Group)，组仅仅是逻辑上的概念，用于保存实体的容器。当建立一个队伍时，GS则请求CS创建一个组，并添加队伍成员实体列表到该组中。

### 网络架构

            CS
          /  |  \
         /   |   \
        /    |    \
       GC    GC   GS

如图中所示，CS只开启一个网络服务，GS和GC均作为CS的客户端。CS在逻辑上通过实体类型运行不同逻辑。       

### 实体间通信

在GC发送的聊天内容中，如果携带道具信息，而正确的道具信息又需要从GS上获取，基于该需求，需要一种方案来支持实体间的通信。该设计中处理实体间通信本质上还是依赖CS的中转。实体间通信要解决的问题实际上是一种聊天字符串的过滤处理，即将原始字符串翻译为玩家最终希望看到的字符串，这个翻译过程则可能涉及到从GS上获取游戏逻辑信息。

## 实现

### 消息协议/流程

* create entity
    * type
    * id
    * brief (i.e name)

    GC和GS都会向CS发送该消息。对于GS而言，CS会检查其IP合法性。对于GC而言，id即为玩家的GUID，该id即为实体的id，GS并不关心id具体值。建立实体时，就需要将该实体与对应的socket关联，以方便以后的消息交互。brief是一个简单的字符串标示，它用于简单地描述一个实体。这个简单描述主要用于CS转发聊天内容给接收者时，描述发送者信息。例如玩家B收到玩家A发的聊天信息时，将显示玩家A的名字。GC将使用玩家的名字作为brief，GS不关心brief。

* create group
    * entity-count
    * entity list 

    创建组，CS需要验证该实体是否有权限。组需要携带组成员列表，也就是实体列表，但这些实体一般都是GC。

* add entity to group
    * entity id
    * group id
* remove entity from group
    * entity id
    * group id

* c2s chat text
    * type (talk to entity or group)
    * group/entity id
    * text

    某实体发出聊天信息，该消息需要标示其接收者是组还是单个实体。如果是组，则CS会广播该聊天内容到指定的组中；如果是单个实体，则直接转发聊天内容。需要注意的是，CS需要先处理这里的聊天内容，例如屏蔽字过滤、物品信息填充等。

* s2c chat text
    * sender entity id
    * sender entity biref (i.e name) 
    * group id (null if this text is a peer to peer message)
    * text

    该消息由CS发送给实体。携带的`sender entity id`可用于客户端查询发送者详细信息等功能。`sender entity brief`可用于客户端显示发送者简要信息，例如名字。`group id`用于组内广播消息，如果是空，则表示该信息为私聊内容。

### 代码示例

参见<https://gist.github.com/3413212>

## 特殊问题处理

### 字符串预处理

可交由lua脚本处理，考虑到这个预处理过程可能会比较复杂，例如获取物品信息，所以需要绑定很多较为底层的接口到脚本中，例如向某个实体发送消息。

获取物品信息示例：

    c++:
    bool Entity::Send(const std::string &text) {
        if (callLua("preprocess_text", text) != "") {
            // lua return text directly
        } else {
            // lua will return text in future
            return false;
        }
    }

    lua:
    function preprocess_text(text)
        if exec_inner_script(text) then
            return nil
        end
        return filter_text(text)
    end

    -- "xxxx{exp}xxx" -> exp, i.e: check out my equipment {query_item(1, 2)}
    function exec_inner_script(text)
        for exp in string.gmatch(text, "[.]*{([.]+)}[.]*") do
            local b, r = pcall(loadstring(exp))
        end
    end 

    function query_item(con, pos)
    end

    -- called by c++
    function process_message(msg)
    end
    
