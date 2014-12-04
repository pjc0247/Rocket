Rocket
======

간단한 예제
----
* 패킷의 스키마를 작성
```Ruby
class LoginRequest < RocketPacket
  required # required 명시된 패킷만 빌드에 참여함
  string :id
  string :password
  timestamp :time
end
```

* PGen 스크립트 실행
```
pgen -s scheme.rb -d .\src\packets.h
```

* Ruby에서의 사용 (테스트용)
서버 테스트를 위한 프레임워크를 Ruby로 제공
```Ruby
class LoginTest < RocketTestUnit
  def query
    qry = LoginRequest.new
    qry.id = "pjc0247"
    qry.pw = "test"
    return qry
  end
  
  def should
    resp = LoginResponse.new
    resp.result = true
    resp.nickname = SKIP # nickname 항목은 검사에서 제외
    return resp
  end
end
```
```Ruby
RocketTestPool.execute \
  [LoginTest]
```

* C/C++에서의 사용
__빌드된 패킷 구조체만 사용__
스키마로부터 빌드된 패킷들의 구조체만 사용합니다.<br>
각각의 패킷에 대한 handle/send 함수, 전반적인 서버 구조는 직접 작성해야 합니다.
```C++
/* on LoginRequest */
LoginResponse resp;
resp.result = true;
resp.nickname = "hello";

send(sock, &resp, sizeof(resp), 0);
```

__래핑 사용__
스키마를 기반으로 빌드된 handle/send 함수들을 사용합니다.<br>
전반적인 서버 구조는 직접 작성해야 합니다.
```C++
#include "Rocket.h"
#include "Packets.h"

/* 자동으로 생성되는 handleLoginRequest 함수 */
void handleLoginRequest(
  SOCKET client,
  const Rocket::String &id, const Rocket::String &pw,
  Rocket::Timestamp time){
  
  /* 자동으로 생성되는 sendLoginResponse 함수 */
  sendLoginResponse(
    client,
    true, "hello");
}

/* 직접 작성해야 하는 부분 */
void processPacket(){
  /* ... */
  if(packetID == LoginRequest::id)
    handleLoginRequest(...);
}
```

__Rocket 프레임워크 사용__
Rocket에서 제공하는 서버 프레임워크를 사용합니다.<br>
네트워크와 서버는 고려하지 않고 각 패킷에 대한 로직만 작성하면 됩니다.
```C++
#include "Rocket.h"
#include "Packets.h"

class LoginHandler{
public:
  void handleRequest(
    const Rocket::string &id, const Rocket::string &password,
    Rocket::Timestamp time){
    
    /* ... */
  }
};

int main(){
  return Rocket::run();
}
```

???
----
* 패킷 스키마 뿐만 아니라, 서버 전반적인 스키마 작성 & 빌드
```Ruby
class Accounts < RocketModel
  property :id, Identifier
  property :user_id, String
  property :user_pw, String
  property :nickname, String
end
class Inventory < RocketModel
  belongs_to :Accounts # One-to-Many관계 명시
  
  property :id, Identifier
  property :type, Int
  property :qty, Int
end
```
```Ruby
class LoginRequest < RocketPacket
  required
  string :id
  string :pw
  timestamp :time
  
  key :id, :Accounts, :user_id
  key :pw, :Accounts, :user_pw
end
```

```C++
result = Accounts::queryLoginRequest();
cout<< result.nickname;
```
