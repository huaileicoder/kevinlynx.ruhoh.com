---
title: '触发器设计(草稿)'
date: '2012-10-19'
categories: mmo
tags: [trigger]
analytics: false
comments: false
---

# DRAFT

    struct Event {
        CGUID owner;
        int type;
        Arguments args;
    };

    struct Request {
        CGUID owner;
        int type;
        Arguments args;
    };

    list<Event> events;

    list<Request> requests;
    
    hash_map<owner-guid, TriggerList*> owner_triggers;

    hash_map<trigger-guid, Trigger*> triggers;

    Event event = CreateEvent(PROPERTY_CHANGE, owner, name, oldval, newval);
    Trigger::PushEvent(event);

    int CreateTrigger(lua_State *L) {
        Request req = CreateRequest(CREATE, owner, inargs);
        Trigger::PushRequest(req);
    }

    int DeleteTrigger(lua_State *L) {
        Request req = CreateRequest(DESTROY, owner);
        Trigger::PushRequest(req);
    }

    void Trigger::OnTimeOut(arg) {
        if (arg->type == CYCLE || arg->type == DELAY) {
            Event event = CreateEvent(TIMER, owner);
            Trigger::PushEvent(event);
        } else if (arg->type == LIFE) {
            Request req = CreateRequest(DESTROY, owner);
            Trigger::PushRequest(req);
        }
    }

    // called in main loop
    void Trigger::Process() {
        // first destroy/create triggers, to make sure the next dispatch loop will use 
        // correct trigger list
        for (RequestList::iterator it = m_requests.begin(); it != m_requests.end(); ++it) {
            Request &req = *it;
            if (req->type == CREATE) {

            } else if (req->type == DESTROY) {

            }
        }

        EventList events = m_events;  // cause the loop below will change the container
        for (EventList::iterator it = events.begin(); it != events.end(); ++it) {
            Event &ent = *it;
            TriggerList *triggerslist = FindList(ent.owner);
            Triggers *triggers = triggerslist->Find(ent.type); // get the specified type list
            if (triggers) {
                for (Triggers::iterator iit = triggers->begin(); iit != triggers->end(); ++iit) {
                    Trigger *trigger = *iit;
                    if (trigger->CheckDispatch(ent))
                        // maybe create/destroy triggers, or create new events
                        // if the trigger life end, what to do ? push destroy request ?
                        trigger->Dispatch(ent);
                }
            }
        }
    }

## problem

* 将创建/销毁与事件分开可能有问题。若某个触发器晚于某个事件的发生而创建，实际上这个触发器是不能享受该事件的，若将两者分开，可能会导致这个事件投递到错误的触发器。如果将事件和请求放在一个队列中，统称事件，那么处理将变为：

        EventList events = m_events;  // cause the loop below will change the container
        for (EventList::iterator it = events.begin(); it != events.end(); ++it) {
            Event &ent = *it;
            if (ent.type == CREATE) {
                // create new trigger 
            } else if (ent.type == DESTROY) {
                // destroy the trigger
            } else { // normal event
                TriggerList *triggerslist = FindList(ent.owner);
                Triggers *triggers = triggerslist->Find(ent.type); // get the specified type list
                if (triggers) {
                    for (Triggers::iterator iit = triggers->begin(); iit != triggers->end(); ++iit) {
                        Trigger *trigger = *iit;
                        if (trigger->CheckDispatch(ent))
                            // maybe create/destroy triggers, or create new events
                            // if the trigger life end, what to do ? push destroy request ?
                            trigger->Dispatch(ent);
                    }
                }
            }
        }

* 如果在触发器A触发脚本时，删除了触发器B，那么当遍历到触发器B时就应该忽略之。但如果将删除请求缓存起来，那么触发器B依然可能得到触发，逻辑不正确。需要通过给触发器添加状态标志来解决这个问题？对于删除事件（请求），需要找到对应的触发器，标记其为非法，然后在下一帧处理事件列表时将其真正删除。

* 这样的结构中，删除一个触发器就是单纯的删除，不会有其他连锁事件发生。删除事件发生时，仅仅标记触发器，不影响触发器相关的管理列表。

* 创建一个触发器时，可能会导致某些事件立即发生，例如碰撞触发器。

* 定时器触发器不再特殊，对于定时器触发器而言，只是其事件产生，是由定时器完成。

* 删除一个对象而导致删除的触发器列表，也不再特殊。

* 强制运行某个触发器，也可抽象为事件，该事件指定触发器ID，强制执行之


