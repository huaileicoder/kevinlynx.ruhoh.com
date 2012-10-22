---
title: '触发器设计'
date: '2012-10-22'
categories: mmo
tags: [trigger]
analytics: false
comments: false
---

本文描述一种基于事件队列的触发器实现。

## 设计思想

本设计最主要的思想，是将游戏世界中触发器模块关注的事件放置到一个队列中。在之前的基于触发器触发实例的设计中，其主要动机是解决触发器递归触发的问题，但随之而来的触发器（包含触发实例）维护成本却增高。基于事件队列的设计，本质上也是解决触发器的递归触发，但在后面的描述中会看到，这样的设计几乎没有更多的类对象维护成本。从某种意义上说，它更轻量，并且让整个系统更简单。

## 主要结构

在描述代码结构之前，需要明确本设计中的一些关键概念。

### 事件

在现有触发器需求中，触发器包含属性变化、碰撞、碰撞离开、定时、移动、移动停止等种类。这里的事件指的就是这些分类，例如当某个对象的属性发生改变时，则对应地产生该事件，投递给触发器模块。

在原来的触发器实现中，定时器事件的处理与其他事件的处理是不同的。在新的设计中，我们将统一这种不同，即定时器响应函数中仅产生事件，放置到事件队列里。

触发器的创建、删除，也被处理为事件。在整个系统中，仅有一个队列。事件在队列中的位置，反应了每个事件发生的时间先后顺序。触发器的创建、删除，需要与普通事件建立一种时间先后顺序（后面会描述原因），因此被作为特殊事件，与普通事件放置在相同队列。

### 事件队列

在游戏的某一帧里，可能产生很多事件。在触发器的触发过程中（脚本里），也可能产生事件。我们将这些事件统一放置到一个队列中。将事件置于队列中，可以避免触发器的递归触发。在每一帧里，都会遍历整个队列，将所有事件处理完。新产生的事件置于下一帧处理。

如果将触发器的创建、删除都统一为事件，那么在触发器的触发过程中，脚本的执行对于触发器模块来说，除了能产生事件以外，不会再有任何相关操作。这一点非常重要，可以有效控制脚本与触发器模块的关联程度。

### 代码示例

这里将对以上关键概念做简要代码设计：

    struct Event {
        CGUID owner; // 事件宿主，亦为触发器宿主
        in type; // 事件类型
        Argument args; // 事件参数集合         
    };

    typedef std::list<Event> EventList;
    EventList Trigger::s_events; // 全局唯一事件队列

当然，除了事件队列这个数据结构外，还有维护触发器对象的结构，这些结构与现有实现基本一致。

最主要的触发器处理实现：


    EventList events = s_events;  // 在触发过程中也会产生事件，所以这里复制到临时队列
    s_events.clear();
    for (EventList::iterator it = events.begin(); it != events.end(); ++it) {
        Event &ent = *it;
        if (ent.type == CREATE) {
            // 创建触发器，直接将新触发器加入到对应的结构，新创建触发器时，可能立即产生事件，例如碰撞检测

        } else if (ent.type == DESTROY) {
            // 删除触发器，相对于基于触发实例的设计而言，删除触发器操作是轻量的，不包含更多的逻辑

        } else { // 普通事件
            TriggerList *triggerslist = FindList(ent.owner); // 查找到该宿主关联的所有触发器
            Triggers *triggers = triggerslist->Find(ent.type); // 进一步获取该事件对应的触发器列表
            if (triggers) { // 若宿主有关心该事件的触发器
                // 进一步检测该事件可能触发的触发器
                for (Triggers::iterator iit = triggers->begin(); iit != triggers->end(); ++iit) {
                    Trigger *trigger = *iit;
                    if (trigger->CheckDispatch(ent))  // 检查该事件是否满足该触发器触发条件
                        // 触发脚本，在触发完后，可能会因为触发次数到上限而主动删除该触发器，这个动作也被作为事件处理
                        trigger->Dispatch(ent);
                }
            }
        }
    }

如果将触发器的创建、删除置于单独的队列里处理，因为没有与普通事件建立时间关系，那么就可能导致某些事件被派发到不该派发的触发器，则可能导致产生了不该有的脚本触发。

基于以上结构，应用层的使用方式可为：

    Event event = CreateEvent(PROPERTY_CHANGE, owner, name, oldval, newval);
    Trigger::PushEvent(event);

    // 脚本接口：创建触发器
    int CreateTrigger(lua_State *L) {
        Request req = CreateRequest(CREATE, owner, inargs);
        Trigger::PushRequest(req);
    }

    // 脚本接口：删除触发器
    int DeleteTrigger(lua_State *L) {
        Request req = CreateRequest(DESTROY, owner);
        Trigger::PushRequest(req);
    }

    // 定时器
    void Trigger::OnTimeOut(arg) {
        if (arg->type == CYCLE || arg->type == DELAY) { // 普通定时器
            Event event = CreateEvent(TIMER, owner);
            Trigger::PushEvent(event);
        } else if (arg->type == LIFE) { // 触发器时限控制
            Request req = CreateRequest(DESTROY, owner);
            Trigger::PushRequest(req);
        }
    }

## 相关问题

该设计能否处理以往触发器实现中的典型问题，以及是否会面对新的复杂问题，在本节讨论。

* 在以上触发器处理循环中，如果触发器A触发过程中删除了触发器B，那么单纯地创建一个删除事件则是不够的。在真正地删除触发器B前，就不应该检测触发器B的触发。为此，简单地解决办法为在触发器上添加有效标志。

* 定时器触发器的处理与普通触发器的处理统一了，这将避免很多定时器类触发器的问题。

* 删除一个宿主而导致删除的触发器列表，也不再特殊。

* 强制运行某个触发器，也可抽象为事件，该事件指定触发器ID，强制执行之

