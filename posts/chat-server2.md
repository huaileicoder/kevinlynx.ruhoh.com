---
title: '聊天服务器设计(二)'
date: '2012-08-22'
categories: mmo
tags: [chat server]
analytics: false
comments: false
---

本文基于《聊天服务器设计(一)》做部分修改，主要将Entity和Group统一起来，使得CS对应用而言，聊天信息仅发送给Entity。Entity在实现上可能是一个玩家个体，一个GS，或者一个Group。

    enum EntityType {
        NIL, CLIENT, GAME_SERVER, GROUP
    };

    class Entity {
        public:
            Entity(int type);
        
            // send text to `this`
            virtual bool Send(Entity *sender, const std::string &text) const = 0;

            virtual bool Modify(ReadBuffer &buf) { }

            virtual std::string PreprocessText(const std::string &text) {
                return text;
            }

            // decode id and brief
            void Create(ReadBuffer &buf);

        protected: 
            int m_type;
            std::string m_brief;
            CGUID m_id;
    };

    // used by Client and Server, bind with socket
    class SockEntity : public Entity {
        public: 
            SockEntity(int type);

            virtual bool Send(const std::string &text) const {
                // find bind socket and send the text
            }
    };

    // created by game server, and be ready when client connected.
    class ClientEntity : public SockEntity {
        public:
            ClientEntity() : SockEntity(CLIENT) {
            }
            
            virtual bool Send(Entity *sender, const std::string &text) const {
                return m_connected ? SockEntity::Send(text) : false;
            }

            virtual bool Modify(ReadBuffer &buf) {
                m_connected = true;
                return true;
            }

            static Entity *Create(ReadBuffer &buf);

        private:
            CGUID m_gsEntity;
            bool m_connected;
    };

    class GSEntity : public SockEntity {
        public:
            GSEntity() : SockEntity(GAME_SERVER) {
            }

            static Entity *Create(ReadBuffer &buf);
    };

    class GroupEntity : public Entity {
        public:
            typedef std::map<CGUID, Entity*> EntityTable;
            enum { DEL, ADD };

            GroupEntity();

            void DecodeMembers(ReadBuffer &buf);

            // broadcast the text to all members
            virtual bool Send(Entity *sender, const std::string &text) const {
                for (EntityTable::const_iterator it = m_mems.begin(); it != m_mems.end(); ++it) {
                    it->second->Send(sender, text);
                }
            }

            virtual bool Modify(ReadBuffer &buf) {
                char t = buf.ReadNumber<char>();
                CGUID entityID = buf.ReadGUID();
                ...
            }

            static Entity *Create(ReadBuffer &buf) {
                Entity *entity = new GroupEntity();
                entity->Create(buf);
                entity->DecodeMembers(buf);
                return entity;
            }

        private:
            void RemoveMember(const CGUID &entityID) {
                m_mems.erase(entityID);
            }

        private:
            EntityTable m_mems;
    };

    class EntityFactory {
        public:
            typedef Entity* (*Creator) (ReadBuffer&);
            typedef std::map<int, Creator> CreatorTable;
            typedef std::map<CGUID, Entity*> EntityTable;

            void Bind() {
                m_creators[CLIENT] = &SockEntity::Create;
                m_creators[GAME_SERVER] = &SockEntity::Create;
                m_creators[GROUP] = &GroupEntity::Create;
            }

            // create entity from message, message format:
            //      type
            //      entity arguments
            bool Create(ReadBuffer &buf) {
                int type = buf.ReadNumber<int>();
                CreatorTable::iterator it = m_creators.find(type);
                if (it == m_creators.end()) return false;
                Entity *entity = (*it->second)(buf);
                m_entities[entity->ID()] = entity;
                return true;
            }

            bool Modify(ReadBuffer &buf) {
                CGUID id = buf.ReadGUID();
                EntityTable::iterator it = m_entities.find(id);
                if (it == m_entities.end()) return false;
                it->second->Modify(buf);
                return true;
            }

            bool SendText(ReadBuffer &buf) {
                CGUID id = buf.ReadGUID();
                std::string text = buf.ReadString();
                return SendText(id, text);
            }

            bool SendText(const CGUID &senderID, const CGUID &receiverID const std::string &text) {
                Entity *sender = Find(senderID);
                Entity *receiver = Find(receiverID);
                if (!sender || !receiver) return false;
                std::string postext = sender->PreprocessText(text);
                if (postext.lenght() == 0) return false;
                return receiver->Send(sender, postext);
            }

            Entity *Find(const CGUID &id) const;

        private:
            CreatorTable m_creators;
            EntityTable m_entities;
    };

因此，消息格式调整为：

* create entity
    * type
    * id
    * brief
    * arguments

    不同类型的entity在创建时，arguments都不同。格式分别为：  

        1. game server entity
            * 无参数
        2. client entity
            * game server entity id
        3. group entity
            * entity count
            * entity list

    *注：client entity的创建改由game server发起创建。client entity的创建必须使用服务器来创建，而client本身只向CS通告自己已连接。* client entity创建过程为：

        client -> game server -> chat server, create client entity 
        client -> chat server, after create entity, notify chat server, the entity created before is ready
* modify entity
    * id
    * arguments

    修改entity，不同entity的修改操作不一样。对于client entity表示其已连接CS，成为一个正常的entity；对于group entity主要是增删成员，其argument格式为：
        
        * type (del/add)
        * entity id
    
    对于game server entity而言目前无意义。
* c2s chat text
    * receiver entity id
    * text
* s2c chat text
    * sender entity id
    * sender entity biref (i.e name) 
    * receiver entity id
    * text

## 部分功能实现

* 客户端实体建立

    客户端实体的建立仅通过服务器来创建，如连接GS的客户端，当角色进入场景时，GS发送创建实体消息到CS。此时创建的实体处于未完成状态，当客户端连接CS并发起修改实体消息时，该实体才转换为完整状态。仅有完整状态的实体才能发送消息。

* 全GS Group的建立

    如有广播全GS的需求，则需要建立全GS group entity。GS初连上CS时，即创建一个group entity。此后每次玩家进入该GS，GS就通过modify entity告知CS添加组成员，从而完整维护全GS group。

* 全WS Group的建立

    基于`全GS Group的建立`实现，WS也会连上CS，并添加各个GS Group entity作为这个group的成员。

* 从Group中移除entity

    因为group是由类似GS这类角色创建的，所以GS本身也维护着类似group的逻辑结构（例如队伍），那么当client entity掉线时，也应该由GS来负责从group中移除。如果移除后的group为空，需要递归移除该group entity所在的group，也由GS来负责。

* 屏蔽某entity发言

    对client entity，修改其状态(modify entity)即可（目前状态有完整、未完整，可添加不可发言状态）。


