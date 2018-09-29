# Babel Protocol: C++ Module (Strasbourg 2018)

Ceci est la spécification du protocole Babel (Module C++, 2018).

This specification defines the protocol used in the Babel project.

The protocol is to be used within a single connection.  
The server 
Le protocole est valide à l’emploi pour une communication client-client ou client-server.

**WARNING: The protocol uses big-endian numbers.**  
**WARNING: Don't forget to use `#pragma pack()` directives on each struct that you intend to directly read into or write from it.**  

## Message structure

|\_\_ **`size`** (4 bytes) \_\_|\_\_ **`type`** (1 byte) \_\_|\_\_ **`body`** (**`size`** bytes) \_\_ ... \_\_|

**`size`** is the size of the message body in bytes.  
**`type`** is the message type.  
**`body`** is the body/data of the message.  

# Message Size

This is the size of the message body, in bytes as a 4 bytes unsigned integer (`uint32_t`)

## Message Type

This is the message type, as one byte (`uint8_t`).
List of message types:

**Client-Server**
   - **`00000000`** (**0**) : **`Command::Ping`**
   - **`10000000`** (**128**) : **`Command::Pong`**
   - **`00000001`** (**1**) : **`Command::Login`** (request)
   - **`10000001`** (**129**) : **`Command::Login`** (response)
   - **`00000010`** (**2**) : **`Command::Logout`** (request)
   - **`10000010`** (**130**) : **`Command::Logout`** (response)
   - **`00000011`** (**3**) : **`Command::Register`** (request)
   - **`10000011`** (**131**) : **`Command::Register`** (response)
   - **`00000100`** (**4**) : **`Command::GetMessages`** (request)
   - **`10000100`** (**132**) : **`Command::GetMessages`** (response)
   - **`00000101`** (**5**) : **`Command::GetUsers`** (request)
   - **`10000101`** (**133**) : **`Command::GetUsers`** (response)
   - **`00000110`** (**6**) : **`Command::SendMessage`** (request)
   - **`10000110`** (**134**) : **`Command::SendMessage`** (response)
   - **`00000111`** (**7**)   : **`Command::CallUser`** (request)
   - **`10000111`** (**135**) : **`Command::CallUser`** (response)
   - **`00001101`** (**13**) : **`Command::AddFriend`** (request)
   - **`10001101`** (**141**) : **`Command::AddFriend`** (response)
   - **`00001110`** (**14**) : **`Command::DelFriend`** (request)
   - **`10001110`** (**142**) : **`Command::DelFriend`** (response)

**Server-Client**
   - **`00001000`** (**8**) : **`Event::IncomingCall`** (event)
   - **`10001000`** (**136**) : **`Event::IncomingCall`** (response)
   - **`00001001`** (**9**) : **`Event::CallStatus`** (event)
   - **`10001001`** (**137**) : **`Event::CallStatus`** (response)
   - **`00001010`** (**10**) : **`Event::StatusUpdate`** (event)
   - **`10001010`** (**138**) : **`Event::StatusUpdate`** (response)
   - **`00001011`** (**11**) : **`Event::GetMessage`** (event)
   - **`10001011`** (**139**) : **`Event::GetMessage`** (response)

**Client-Client**
   - **`00001100`** (**12**) : **`Event::AudioFrame`** (event)


## Prelude

These are types and defines that will be used throughout the specification.  
It is strongly encouraged you use the exact same definitions of these types.

```c++
using u8 = uint8_t;
using u16 = uint16_t;
using u32 = uint32_t;
using i8 = int8_t;
using i16 = int16_t;
using i32 = int32_t;

#define STD_SIZE (256)
#define MSG_SIZE (1024)

/*
** This define is optional.
** It just allows to pack a struct
** by annotating it like this:
** 
**   PACK(struct Test {
**       bool status;
**       u32 id;
**   });
*/
#ifdef _MSC_VER
# define PACK(d) __pragma(pack(push, 1)) d __pragma(pack(pop))
#else
# define PACK(d) d __attribute__((packed))
#endif

enum class Status : u8 {
    Disconnected = 0,
    Available,
    Away,
    Busy,
    Hidden
};

enum class Request : u8 {
    Ping = 0,
    Login,	
    Logout,
    Register,
    GetMessages,
    GetUsers,
    SendMessage,
    CallUser,
    IncomingCall,
    CallStatus,
    StatusUpdate,
    GetMessage,
    AudioFrame,
    AddFriend,
    DelFriend
};

enum class Response : u8 {
    Pong = 128,
    Login,	
    Logout,
    Register,
    GetMessages,
    GetUsers,
    SendMessage,
    CallUser,
    IncomingCall,
    CallStatus,
    StatusUpdate,
    GetMessage,
    AudioFrame,
    AddFriend,
    DelFriend
};
    
struct Msg {
    u32 sender;
    char msg[STD_SIZE];
};

namespace Command {
    // All commands go in this namespace
};

namespace Event {
    // All events go in this namespace
};
```

## Command::Login
### Request
```c++
struct LoginRequest {
    char login[STD_SIZE];
    char passwd[STD_SIZE];
    Status status;
};
```

### Response
```c++
struct LoginResponse {
    bool status;
    u32 id;
    char nickname[STD_SIZE];
};
```

## Command::Logout
### Request
Zero-sized request.
```c++
struct LogoutRequest {};
```
### Response
```c++
struct LogoutResponse {
    bool status;
};
```

## Command::Register
### Request
```c++
struct RegisterRequest {
    char login[STD_SIZE];
    char nickname[STD_SIZE];
    char passwd[STD_SIZE];
};
```

### Response
```c++
struct RegisterResponse {
    bool status;
};
```
## Command::GetMessages
### Request
```c++
struct GetMsgRequest {
    u32 dest_id;
};
```
### Response
```c++
//! Can't be read directly
// messages.size() == body_size / sizeof(Msg)
struct GetMsgResponse {
    std::vector<Msg> messages;
};
```

## Command::GetUsers
```c++
enum class UserQuery : u8 {
    All = 0,
    Contact,
    Friend,
};
```
### Request
```c++
struct GetUsersRequest {
    UserQuery type;
};
```
### Response
```c++
enum class UserType : u8 {
    Contact = 0,
    Friend,
};

struct User {
    UserType type;
    u32 id;
    char name[STD_SIZE];
    Status status;
};

//! Can't be read directly
// users.size() == body_size / sizeof(User)
struct GetUsersResponse {
    std::vector<User> users;
};
```
## Command::SendMessage
### Request
```c++
struct SendMessageRequest {
    Msg msg;
};
```
### Response
```c++
struct SendMessageResponse {
    bool status;
};
```

## Command::CallUser
### Request
```c++
struct CallUserRequest {
    u32 dest_id;
    u16 port;
    u32 sampleRate;
    u32 frameSize;
};
```
### Response
```c++
struct CallUserResponse {
    bool status;
};
```

## Command::AddFriend
### Request
```c++
struct AddFriendRequest {
    u32 friend_id;
};
```
### Response
```c++
struct AddFriendResponse {
    bool status;
};
```

## Command::DelFriend
### Request
```c++
struct DelFriendRequest {
    u32 friend_id;
};
```
### Response
```c++
struct DelFriendResponse {
    bool status;
};
```

## Event::IncomingCall
### Event
```c++
struct IncomingCallRequest {
    u32 dest_id;
    u8 ip[4];
    u16 port;
};
```
### Response
```c++
struct IncomingCallResponse {
    bool status;
};
```

## Event::GetMessage
### Event
```c++
struct GetMessageRequest {
    Msg msg;
};
```
### Response
```c++
struct GetMessageResponse {
    bool status;
};
```

## Event::CallStatus
### Event
```c++
struct CallStatusRequest {
    bool accepted;
};
```
### Response
```c++
struct CallStatusResponse {
    bool accepted;
};
```
## Event::StatusUpdate
### Event
```c++
struct StatusUpdateRequest {
    u32 id;
    Status newStatus;
};
```
### Response
```c++
struct StatusUpdateResponse {
    bool accepted;
};
```
## Event::AudioFrame
### Event
```c++
//! This can't be read directly
struct AudioFrameRequest {
    u32 size;
    std::vector<unsigned char> packet;
};
```

