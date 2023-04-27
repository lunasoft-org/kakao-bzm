# 카카오 비즈메시지 알림톡 API v1.0.0

1. [개요](#1-개요)
2. [용어 정의](#2-용어-정의)
3. [선결 조건](#3-선결-조건)
4. [API 스펙](#4-API-스펙)
5. [발송 타입 별 Example](#5-발송-타입-별-Example)
6. [코드 정의](#6-코드-정의)
7. [테스트-방법](#7-테스트-방법)

## 참고 : 변경사항

| 일시       | 변경 내역                                                                                                                                                                                                                |
| ---------- | -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2022.10.    | 카카오 비즈메시지 업로드 API 초안 작성        

## 1. 개요

+ 본 문서는 https 프로토콜을 통해 데이터를 json 형식으로 가공하여 GET, POST method를 사용하여 전송하고, json형식의 결과 데이터를 parsing하여 처리할 수 있음을 전제로 작성되었다.
+ 비즈메시지 알림톡 API는 루나소프트에 등록된 고객사에 대해 허용된 IP로 접근할 수 있는 제한적으로 오픈된 서비스를 제공한다.

## 2. 용어 정의

### 2.1 허브 파트너(Hub Partner)
+ 제휴한 카카오톡 채널 통해 전화번호로 특정되는 카카오톡 사용자에게 메시지 전송을 대행하는 사업자

### 2.2 카카오톡 채널(Kakao Talk Channel)
+ 카카오톡 계정을 기반으로 한 비즈니스용 카카오톡 아이디
+ 카카오톡 채널 홈페이지([https://center-pf.kakao.com](https://center-pf.kakao.com))를 통해 개설

## 3. 선결 조건
> 메시지 API를 사용하여 메시지 전송을 대행하기 위해 아래의 조건이 선결되어야 한다.

+ 카카오 비즈메시지 센터 API를 통해 **메시지 발송 주체인 카카오톡 채널을 등록**하고, 발신 프로필 키(Sender Key)를 조회할 수 있다.
+ 알림톡의 경우 메시지 발송을 위해 비즈 메시지 센터 API를 통해 등록한 사용자 정보로 **전송할 메시지 유형의 템플릿과 템플릿 코드를 등록**해야 한다.
+ 메시지 전송 시스템의 IP 정보를 루나소프트 운영 담당자에게 전달하여 메시지 API 서버에 접근할 수 있도록 **접근 허용 요청**을 해야 한다.

+ 루나소프트 운영 담당자를 통해 발송에 필요한 `AgentKey`를 발급 받아야 한다. (발송 시 header에 추가)
+ 비 실시간 발송 시 루나소프트 운영 담당자를 통해 `WebHook URL`을 사전에 등록해야 한다.

## 4. API 스펙

### 4.1 Host
> ####**[개발서버]** `https://test-kakao-bizmessage.lunasoft.co.kr/`

> ####**[운영서버]** `https://kakao-bizmessage.lunasoft.co.kr/`

### 4.2 메시지 전송 요청

#### 4.2.1 실시간(Realtime)
> 전송 요청한 메시지를 즉시 발송 한 후, 발송 결과에 대한 상태 값(성공, 실패)을 바로 받을 수 있다.<br/>
> 단, 최대 발송량을 100 tps로 권장.<br/>
> (최대 tps 초과 시 timeout으로 인해 발송이 실패 될 수 있기 때문에 대량 발송 시 **비 실시간(non-Realtime)** 발송을 권장)

##### [Request]

+ path : /api/v3/alimtalk/send
+ method : POST
+ header
  + Content-type: application/json
  + agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`
+ client timeout
  + 100초 (아주 적은 빈도로 API 응답이 비정상적으로 지연되는 경우가 있어서 100초를 권장합니다)
+ parameter (json)

| 키 | 타입 | 필수 | 설명 | 예제 |
| ------------------ | ------------------ | ------------------ | ------------------ | ------------------ |
| message_type | text(2) | Y | 메시지 타입<br/>(AT: 알림톡, AI: 이미지 알림톡) | "message_type":"AT" |
| message_key | text(30) | Y | 메시지일련번호 (메시지에 대한 고유값) | "message_key":"605498276" |
| sender_key | text(40) | Y | 발신 프로필 키<br/> | "sender_key":"2662e99eb7a1f21abb3955278e9955f5a9a99b62" |
| country_code | text(6) | Y | 국가코드 | "country_code":"82" |
| recipient_number | text(20) | N | 사용자 전화번호<br/>**recipient_number 혹은 app_user_id 둘 중 하나는 반드시 있어야 한다.** | "recipient_number":"01012345678" |
| app_user_id | text(20) | N | 앱유저아이디<br/>**recipient_number 혹은 app_user_id 둘 중 하나는 반드시 있어야 한다.** <br/> recipient_number와 app_user_id의 정보가 동시에 요청된 경우 recipient_number로만 발송합니다.| "app_user_id":"12345" |
| template_code | text(30) | Y | 템플릿코드<br/>(실제 발송할 메시지 유형으로 등록된 템플릿의 코드) | "template_code":"A001_01" |
| message | text(1000) | Y | 사용자에게 전달될 메시지<br/>(공백 포함 1000자로 제한) | "message":"고객님의 택배가 금일 (18~20)시에 배달 예정입니다." |
| title | text(50) | N | 템플릿 내용 중 강조 표기할 핵심 정보 ([템플릿 검수 가이드 참고](https://kakaobusiness.gitbook.io/main/ad/bizmessage/notice-friend/audit))<br/> | "title":"20분 내 도착 예정" |
| header | text(16) | N | 메시지 상단에 표기할 제목 | "header":"포인트 적립 안내" |
| attachment | json | N | 메시지에 첨부할 내용<br/>(링크 버튼 / "target":"out" 속성 추가시 아웃링크) | ```"attachment":{"button":[{"name":"버튼명","type":"WL","url_pc":"https://lunasoft.co.kr/home/main/page/main/index", "url_mobile":"http://daum.net","target":"out"}]}```<br/> **[<4.3 Attachment 에서 확인 가능>](#43-Attachment)** |
| supplement | json | N | 메시지에 첨부할 바로연결<br/>(링크 버튼 / "target":"out" 속성 추가시 아웃링크) | ```"supplement":{"quick_reply":[{"name":"버튼명","type":"WL","url_pc":"https://lunasoft.co.kr/home/main/page/main/index", "url_mobile":"http://daum.net","target":"out"}]}```<br/> **[<4.4 Supplement 에서 확인 가능>](#44-Supplement)**|
| price | number | N | 모먼트 광고 전환 최적화 전용<br/>메시지 내 포함된 가격/금액/결제금액 | "price":39900 |
| currency_type | text(3) | N | 모먼트 광고 전환 최적화 전용<br/>메시지 내 포함된 가격/금액/결제금액의 통화단위 <br>KRW, USD, EUR 등 국제 통화 코드 사용 | "currency_type":"KRW" |
+ **모먼트 광고 전환 최적화 전용 필드는 메시지 본문에 반영되지 않습니다.**

##### [Response]

| 키 | 타입 | 필수 | 설명 | 예제 |
| ------ | ------ | ------ | ------ | ------ |
| code | text(6) | Y | 처리 결과 코드<br/>(0000은 정상 / 나머지는 오류) | "code ":"0000" **[<6.1 오류코드 에서 확인 가능>](#61-오류-코드)**|
| message_key | text(30) | N | 메시지일련번호 (요청시 설정한 메시지일련번호) | "message_key":"605498276" |
| message | text | N | 오류 메시지<br/>(오류시 존재하는 값) | "message ":"AckTimeoutException(1)" |


##### [Example] (※ 테스트 하기 전에 7.1 테스트 선결 조건을 먼저 확인하세요.)

```
$ curl  -H "Accept: application/json" -H "Content-type: application/json" -H "agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`" -X POST -d '{message_type":"AT","message_key":"2017010100004","sender_key":"2662e99eb7a1f21abb3955278e9955f5a9a99b62","country_code":"82","recipient_number":"01012345678","template_code":"A001_01","message":"임꺽정님이 보낸 등기 1234567_89123456를 홍길동(회사동료)님께 배달 완료 1588-1300"}' https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send

{"code":"0000","message_key":"2017010100004"}
```

#### 4.2.2 비 실시간(non-Realtime)
> 전송 요청한 메시지를 Queue에 저장 한 후 순차적으로 발송 한다.<br/>실시간 발송에 비해 tps에 대한 부담이 적으나 발송 결과를 즉시 받지 못하며, 발송 결과를 받을 수 있는 별도의 Webhook 설정이 필요하다.

##### [Request]

+ path : /api/v3/alimtalk/send/async<br/>**나머지 항목들은 위 실시간 발송과 동일함.**

##### [Response]

| 키 | 타입 | 필수 | 설명 | 예제 |
| ------ | ------ | ------ | ------ | ------ |
| code | text(6) | Y | 처리 결과 코드<br/>(LS0000은 발송 요청 성공 / 나머지는 오류) | "code ":"LS0000" **[<6.1 오류코드 에서 확인 가능>](#61-오류-코드)**|
| message_key | text(30) | N | 메시지일련번호 (요청시 설정한 메시지일련번호) | "message_key":"605498276" |
| message | text | N | 오류 메시지<br/>(오류시 존재하는 값) | "message ":"AckTimeoutException(1)" |

##### 4.2.3 Webhook
> 루나소프트 운영 담당자를 통해 `WebHook URL`을 사전에 등록해야 함.
- path : `사전에 등록된 WebHook URL`
- method : `POST`
- header :
  - Content-Type : application/json
- body :

```json
{
  "code": "text",  
  "message_key": "text",
  "message_type": "text",
  "response_dt": "text",
  "message": "text"
}
```

| 키 | 타입 | 필수 | 설명 | 예제 |
| ------ | ------ | ------ | ------ | ------ |
| code | text(6) | Y | 처리 결과 코드<br/>(0000은 정상 / 나머지는 오류) | "code ":"0000" **[<6.1 오류코드 에서 확인 가능>](#61-오류-코드)**|
| message_key | text(30) | N | 메시지일련번호 (요청시 설정한 메시지일련번호) | "message_key":"605498276" |
| message_type | text(2) | Y | 메시지 타입<br/>(AT: 알림톡, AI: 이미지 알림톡) | "message_type":"AT" |
| response_dt | datetime | Y | 메시지 발송 응답 시간 | 2023-05-01 09:05:30 |
| message | text | N | 오류 메시지<br/>(오류시 존재하는 값) | "message ":"AckTimeoutException(1)" |

- 발송 성공 Sample
```json
{
  "code": "0000",  
  "message_key": "605498276",
  "message_type": "AT",
  "response_dt": "2023-05-01 09:05:30"
}
```

- 발송 실패 Sample
```json
{
  "code": "3016",  
  "message_key": "605498276",
  "message_type": "AT",
  "response_dt": "2023-05-01 09:05:30",
  "message": "NoMatchedTemplateException"
}
```

### 4.2 알림톡 발송 방식

> Push 방식

대체 발송에 따른 메시지 중복 수신 문제를 해결한 방식이다.<br/>
미리 정의한 조건에 만족하는 활성 사용자에게 메시지를 전송하고 성공 처리한다.<br/>
활성 사용자 조건은 아래와 같다.<br/>

1. 서버와 연결되어 있는 카카오톡 사용자
2. 발송 당일 가입한 사용자를 제외한 최근 7일(168시간) 내에 카카오톡을 사용한 사용자

조건에 만족하지 않는 경우 NoSendAvailableException 오류를 결과로 응답한다.<br/>


### 4.3 Attachment
알림톡은 **[<4.2 메시지 전송 요청>](#42-메시지-전송-요청)**의 요청 필드 중 attachment 값에 링크 버튼을 첨부하여 발송할 수 있다.<br/>
**버튼은 목록으로(Array) 최대 5개까지** 템플릿에 등록하여 발송할 수 있다.<br/>

##### [Parameter]

| 키 | - |  - |타입 | 필수 | 설명 |
| ------ | ------ | ------ | ------ | ------ | ------ |
| button |  |  | array | - | 버튼 목록 |
|  | name |  | text(14) | Y | 버튼 제목 (AC 버튼의 경우 "채널 추가"로 요청) |
|  | type |  | text(2) | Y | 버튼 타입 **[<4.3.1 버튼 타입에서 확인 가능>](#431-버튼-타입별-속성)** |
|  | scheme_android |  | text | - | mobile android 환경에서 버튼 클릭 시 실행할<br/>application custom scheme |
|  | scheme_ios |  | text | - | mobile ios 환경에서 버튼 클릭 시 실행할<br/>application custom scheme |
|  | url_mobile |  | text | - | mobile 환경에서 버튼 클릭 시 이동할 url |
|  | url_pc |  | text | - | pc 환경에서 버튼 클릭 시 이동할 url |
|  | chat_extra |  | text(50) | - | 상담톡/봇 전환 시 전달할 메타정보 |
|  | chat_event |  | text(50) | - | 봇 전환 시 연결할 봇 이벤트명 |
|  | plugin_id |  | text(24) | - | 플러그인 ID |
|  | relay_id |  | text | - | 플러그인 실행시 X-Kakao-Plugin-Relay-Id 헤더를 통해 전달 받을 값<br>([카카오톡 비즈플러그인 안내 페이지](https://business.kakao.com/info/talkbizplugin) 하단 개발 가이드 참고  |) |
|  | oneclick_id |  | text | - | 원클릭 결제 플러그인에서 사용하는 결제 정보<br>([카카오톡 비즈플러그인 안내 페이지](https://business.kakao.com/info/talkbizplugin) 하단 개발 가이드 참고) |
|  | product_id |  | text | - | 원클릭 결제 플러그인에서 사용하는 결제 정보<br>([카카오톡 비즈플러그인 안내 페이지](https://business.kakao.com/info/talkbizplugin) 하단 개발 가이드 참고) |
| item_highlight |  |  | json | N | 아이템 하이라이트 |
|  | title |  | text(30) | Y | 타이틀 (이미지가 있는 경우 최대 21자) |
|  | description |  | text(19) | Y | 부가정보 (이미지가 있는 경우 최대 13자) |
| item |  |  | json | N |  아이템리스트와 아이템 요약정보 |
|  | list |  | array | N | 아이템리스트 (2022-08-30 부터이며 그 이전에는 Y) |
|  | | title | text(6) | Y | 타이틀 |
|  | | description | text(23) | Y | 부가정보 |
|  | summary |  | json | N | 아이템 요약정보 |
|  | | title | text(6) | Y | 타이틀 |
|  | | description | text(14) | Y | 가격정보<br>* 허용되는 문자: 통화기호(유니코드 통화기호, 元, 円, 원), 통화코드(ISO 4217), 숫자, 콤마, 소수점, 공백<br>* 소수점 2자리까지 허용 |


##### [Example]
```
"attachment":{"button":[{"name":"비즈메시지 소개","type":"WL","url_pc":"http://bizmessage.kakao.com/", "url_mobile":"http://bizmessage.kakao.com/"}]}

"attachment":{"button":[{"name":"비즈메시지 소개","type":"WL","url_pc":"http://bizmessage.kakao.com/", "url_mobile":"http://bizmessage.kakao.com/"},{"name":"커스텀스킴 테스트","type":"AL","scheme_ios":"scheme://xxx.xx", "scheme_android":"scheme://xxx.xx"}]}

"attachment":{"button":[{"name":"비즈메시지 소개","type":"WL","url_pc":"http://bizmessage.kakao.com/", "url_mobile":"http://bizmessage.kakao.com/"},{"name":"커스텀스킴 테스트","type":"AL","scheme_ios":"scheme://xxx.xx", "scheme_android":"scheme://xxx.xx"}], "item": { "list": [{"title": "가격", "description": "10,000원"}, {"title": "할인금액", "description": "-1원"}]}}

"attachment":{"button":[{"name":"비즈메시지 소개","type":"WL","url_pc":"http://bizmessage.kakao.com/", "url_mobile":"http://bizmessage.kakao.com/"},{"name":"커스텀스킴 테스트","type":"AL","scheme_ios":"scheme://xxx.xx", "scheme_android":"scheme://xxx.xx"}], "item_highlight": {"title": "비즈메시지", "description": "알림톡"}, "item": { "list": [{"title": "가격", "description": "10,000원"}, {"title": "할인금액", "description": "-1원"}], "summary": {"title": "총 금액", "description": "9,999원"}}}
```

#### 4.3.1 버튼 타입별 속성

+ 필수 파라메터를 모두 입력하셔야 정상적인 발송이 가능합니다.

| 버튼타입 | 속성 | 타입 | 필수 | 설명 |
| ------ | ------ | ------ | ------ | ------ |
| WL | url_mobile | text | Y | 버튼 클릭 시 이동할 pc/mobile환경별 web url |
| | url_pc | text | N | |
| AL | scheme_android | text | - | **scheme_ios, scheme_android, url_mobile 중 2가지 필수 입력**<br/>mobile android 환경에서 버튼 클릭 시 실행할 application custom scheme |
| | scheme_ios | text | - | mobile ios 환경에서 버튼 클릭 시 실행할 application custom scheme |
| | url_mobile | text | - | mobile 환경에서 버튼 클릭 시 이동할 url |
| | url_pc | text | N | pc 환경에서 버튼 클릭 시 이동할 url |
| DS | - | - | - | 버튼 클릭 시 배송조회 페이지로 이동 |
| BK | - | - | - | 해당 버튼 텍스트 전송 |
| MD | - | - | - | 해당 버튼 텍스트 + 메시지 본문 전송 |
| BC | - | - | - | 상담톡을 이용하는 카카오톡 채널만 이용가능 |
|  | chat_extra | text | N | 상담톡 전환 시 전달할 메타정보 |
| BT | - | - | - | 카카오 I 오픈빌더의 챗봇을 사용하는 카카오톡 채널만 이용가능 |
|  | chat_extra | text | N | 봇 전환 시 전달할 메타정보 |
|  | chat_event | text | N | 봇 전환 시 연결할 봇 이벤트명 |
| AC | - | - | - | 버튼 클릭 시 카카오톡 채널 추가 |
| P1 | - | - | - | 이미지 보안 전송 플러그인 |
| P2 | - | - | - | 개인정보이용 플러그인 |
| P3 | - | - | - | 원클릭 결제 플러그인 <br> (발송시 oneclick_id 또는 product_id를 필수로 전달해아 함) |
| BF | biz_form_id | number | Y | [카카오 비즈니스](https://business.kakao.com/)에서 생성한 비즈니스폼 ID |


### 4.4 Supplement
알림톡은 **[<4.2 메시지 전송 요청>](#42-메시지-전송-요청)**의 요청 필드 중 supplement 값에 링크 바로연결을 첨부하여 발송할 수 있다.<br/>
**바로연결은 목록으로(Array) 최대 10개까지** 템플릿에 등록하여 발송할 수 있다.<br/>
**단, 바로연결을 포함하여 발송 시, 버튼은 2개만 등록하여 발송할 수 있다.**<br/>

##### [Parameter]

| 키 | - | 타입 | 필수 | 설명 |
| ------ | ------ | ------ | ------ | ------ |
| quick_reply |  | array | - | 바로연결 목록 |
|  | name | text(14) | Y | 바로연결 제목 |
|  | type | text(2) | Y | 바로연결 타입 **[<4.4.1 바로연결 타입에서 확인 가능>](#441-바로연결-타입별-속성)** |
|  | scheme_android | text | - | mobile android 환경에서 바로연결 클릭 시 실행할<br/>application custom scheme |
|  | scheme_ios | text | - | mobile ios 환경에서 바로연결 클릭 시 실행할<br/>application custom scheme |
|  | url_mobile | text | - | mobile 환경에서 바로연결 클릭 시 이동할 url |
|  | url_pc | text | - | pc 환경에서 바로연결 클릭 시 이동할 url |
|  | chat_extra | text(50) | - | 상담톡/봇 전환 시 전달할 메타정보 |
|  | chat_event | text(50) | - | 봇 전환 시 연결할 봇 이벤트명 |


##### [Example]
```
"supplement":{"quick_reply":[{"name":"비즈메시지 소개","type":"WL","url_pc":"http://bizmessage.kakao.com/", "url_mobile":"http://bizmessage.kakao.com/"}]}

"supplement":{"quick_reply":[{"name":"비즈메시지 소개","type":"WL","url_pc":"http://bizmessage.kakao.com/", "url_mobile":"http://bizmessage.kakao.com/"},{"name":"커스텀스킴 테스트","type":"AL","scheme_ios":"scheme://xxx.xx", "scheme_android":"scheme://xxx.xx"}]}
```

#### 4.4.1 바로연결 타입별 속성

+ 필수 파라메터를 모두 입력하셔야 정상적인 발송이 가능합니다.

| 바로연결타입 | 속성 | 타입 | 필수 | 설명 |
| ------ | ------ | ------ | ------ | ------ |
| WL | url_mobile | text | Y | 바로연결 클릭 시 이동할 pc/mobile환경별 web url |
| | url_pc | text | N | |
| AL | scheme_android | text | - | **scheme_ios, scheme_android, url_mobile 중 2가지 필수 입력**<br/>mobile android 환경에서 바로연결 클릭 시 실행할 application custom scheme |
| | scheme_ios | text | - | mobile ios 환경에서 바로연결 클릭 시 실행할 application custom scheme |
| | url_mobile | text | - | mobile 환경에서 바로연결 클릭 시 이동할 url |
| | url_pc | text | N | pc 환경에서 바로연결 클릭 시 이동할 url |
| BK | - | - | - | 해당 바로연결 텍스트 전송 |
| BC | - | - | - | 상담톡을 이용하는 카카오톡 채널만 이용가능 |
|  | chat_extra | text | N | 상담톡 전환 시 전달할 메타정보 |
| BT | - | - | - | 카카오 I 오픈빌더의 챗봇을 사용하는 카카오톡 채널만 이용가능 |
|  | chat_extra | text | N | 봇 전환 시 전달할 메타정보 |
|  | chat_event | text | N | 봇 전환 시 연결할 봇 이벤트명 |
| BF | biz_form_id | number | Y | [카카오 비즈니스](https://business.kakao.com/)에서 생성한 비즈니스폼 ID |


## 5. 발송 타입 별 Example
### 5.1 기본 알림톡 발송 요청
<img src="/Images/Alimtalk/알림톡_기본.png" alt="기본 알림톡 발송" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000001",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "100016",
    "message": "본인확인 휴대폰 인증창에 입력하세요.\n[인증번호 : 12345678]"
}'
```

### 5.2 버튼 알림톡 발송 요청
+ 버튼은 최대 5개까지 등록하여 발송할 수 있습니다.

#### 5.2.1 알림톡 배송조회 버튼
+ 배송조회 버튼 타입 : DS
+ 배송조회 버튼은 클릭시 배송조회 페이지로 이동합니다.
+ 알림톡 메시지 파싱을 통해 배송조회 카카오검색 페이지 링크가 자동 생성 됩니다.
+ 배송조회 링크가 정상적으로 생성되는지 필히 사전 테스트 해보시기 바랍니다.<br>아래의 **지원 가능 택배사와 송장번호 패턴이 같을 경우만 배송조회 버튼이 표시**됩니다.

> 우체국택배, 로젠택배, 일양로지스, FedEx, 한진택배, 경동택배, 합동택배, 롯데택배, 농협택배,호남택배, 천일택배, 대신택배, 건영택배, CU편의점택배, CVSnet편의점택배, 한덱스, TNT Express, USPS, EMS, DHL, 굿투럭<br>**CJ택배사는 미지원**

<img src="/Images/Alimtalk/알림톡_배송조회_버튼.png" alt="알림톡 배송조회 버튼" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000003",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "121093",
    "message": "[루나소프트]\n안녕하세요.TEST님\n\n주문하신 상품이 발송 되었습니다.\n\n- 주문자명 : TEST\n- 수령자명 : RECEIVER\n- 날짜 : 2022-11-09 12:09\n- 상품명 : 테스트 상품\n- 택배사 : 한진택배\n- 운송장번호 : 531234567890\n\n공휴일 제외 1~2일 내에 배송될 예정입니다.\n\n\n▷ 루나소프트 바로가기\nlunasoft.co.kr\n고객센터\n(1644-4998)",
    "attachment": {
        "button": [
            {
              "type": "DS",
              "name": "배송 조회하기"
            },
            {
              "type": "WL",
              "name": "홈페이지 바로가기",
              "url_mobile": "https://lunasoft.co.kr",
              "url_pc": "https://lunasoft.co.kr"
            }
        ]
    }
}'
```

#### 5.2.2 알림톡 웹링크 버튼
+ 웹링크 버튼 타입 : WL
+ 웹링크 버튼은 클릭시 설정된 url 페이지로 이동합니다.
<img src="/Images/Alimtalk/알림톡_웹링크_버튼.png" alt="알림톡 웹링크 버튼" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000001",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "100017",
    "message": "[루나소프트] 회원가입 안내\nTEST님, 루나소프트 회원이 되신 것을 환영합니다.\n    \n▶신규 가입 회원 혜택\n신규 가입 회원 혜택 정보 - 1\n신규 가입 회원 혜택 정보 - 2",
    "attachment": {
        "button": [
            {
                "type": "WL",
                "name": "홈페이지 바로가기",
                "url_mobile": "http://lunasoft.co.kr",
                "url_pc": "http://lunasoft.co.kr"
            },
            {
                "type": "WL",
                "name": "웹링크 바로가기",
                "url_mobile": "http://lunasoft.co.kr",
                "url_pc": "http://lunasoft.co.kr"
            }
        ]
    }
}'
```

#### 5.2.3 알림톡 앱링크 버튼
+ 앱링크 버튼 타입 : AL
+ 앱링크 버튼은 클릭시 설정된 application custom scheme를 실행합니다.
+ **[ scheme_android, scheme_ios, url_mobile 중 2가지 필수 입력 ]**
<img src="/Images/Alimtalk/알림톡_앱링크_버튼.png" alt="알림톡 앱링크 버튼" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000005",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "150074",
    "message": "[루나소프트]\n안녕하세요. TEST님\n\n결제완료 되었습니다.\n감사합니다.\n\n- 날짜 : 2022-11-09 12:00\n- 주문번호 : 12345678-987654321\n\n고객센터\n(1644-4998)",
    "attachment": {
        "button": [
            {
                "type": "AL",
                "name": "쇼핑몰 APP",
                "scheme_android": "daumapps://open",
                "scheme_ios": "daumapps://open",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            }
        ]
    }
}'
```

#### 5.2.4 알림톡 봇키워드 버튼
+ 봇키워드 버튼 타입 : BK
+ 봇키워드 버튼은 클릭시 **해당 버튼의 텍스트를 전송**합니다.
<img src="/Images/Alimtalk/알림톡_봇키워드_버튼.png" alt="알림톡 봇키워드 버튼" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000006",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "126205",
    "message": "주문번호 : 12345678-987654321\n상품명 : 테스트 상품\n\n고객님!\n접수하신 #{문의}요청이 상담원 확인 중입니다.\n\n추가 확인이 필요한 경우, 상담원이 고객님께 연락드릴 수 있습니다.\n\n조금만 기다려주세요!",
    "attachment": {
        "button": [
            {
                "type": "BK",
                "name": "챗봇 시작하기"
            },
            {
                "type": "WL",
                "name": "쇼핑몰 둘러보기",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            }
        ]
    }
}'
```

#### 5.2.5 알림톡 메시지전달 버튼
+ 메시지 전달 버튼 타입 : MD
+ 메시지 전달 버튼은 클릭시 **해당 버튼의 텍스트 + 메시지 본문을 전송**합니다.
<img src="/Images/Alimtalk/알림톡_메시지전달_버튼.png" alt="알림톡 메시지전달 버튼" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000007",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "150179",
    "message": "[루나소프트]\n안녕하세요 TEST님,\n\n- 주문번호 : 12345678-987654321\n- 날짜 : 2022-11-09 12:00\n\n접수 처리 접수 되었습니다.\n\n▷ 루나소프트 바로가기\nlunasoft.co.kr\n고객센터\n(1644-4998)",
    "attachment": {
        "button": [
            {
                "type": "MD",
                "name": "메시지전달"
            },
            {
                "type": "WL",
                "name": "쇼핑몰 둘러보기",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            }
        ]
    }
}'
```

#### 5.2.6 알림톡 채널추가 버튼
+ 채널추가 버튼 타입 : AC
+ 채널추가 버튼은 클릭시 **카카오톡 채널을 추가**합니다.
+ 채널추가 버튼은 **첫 번째 버튼에만 사용**할 수 있습니다.
+ **채널추가가 안된 채널에서만 채널추가 버튼이 표시**됩니다.
<img src="/Images/Alimtalk/알림톡_채널추가_버튼.png" alt="알림톡 채널추가 버튼" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000009",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "151189",
    "message": "[루나소프트]\n안녕하세요.TEST님\n\n입금계좌 안내드립니다.\n\n아래 계좌로 입금 부탁드리며, 평일 오후3시 이전까지 입금해주신 고객님들께는 당일 출고 진행 도와드리고있습니다.\n\n- 금액 : 55,000\n- 은행 : 루나은행\n- 계좌번호 : 123-456-789654-321\n- 예금주 : 루나\n- 입금 후 1시간이내 입금 확인 알림톡을 수령받아보지 못하신 경우시라면\n고객센터 측으로 연락 진행 부탁드리겠습니다.\n\n▷ 루나소프트 바로가기\nlunasoft.co.kr\n고객센터\n(1644-4998)",
    "attachment": {
        "button": [
            {
                "type": "AC",
                "name": "채널 추가"
            },
            {
                "type": "WL",
                "name": "쇼핑몰 둘러보기",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            }
        ]
    }
}'
```

### 5.3 바로연결 알림톡 발송 요청
+ 바로연결은 목록으로(Array) 최대 10개까지 템플릿에 등록하여 발송할 수 있습니다.
+ **단, 바로연결을 포함하여 발송 시, 버튼은 2개까지만 등록하여 발송할 수 있습니다.**

#### 5.3.1 알림톡 바로연결 웹링크 버튼
+ 바로연결 웹링크 버튼 타입 : WL
+ 바로연결 웹링크 버튼은 클릭시 설정된 url 페이지로 이동합니다.
<img src="/Images/Alimtalk/알림톡_바로연결_웹링크_버튼.png" alt="알림톡 바로연결 웹링크 버튼" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000010",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "180001",
    "message": "[루나소프트] 회원가입 안내\nTEST님, 루나소프트 회원이 되신 것을 환영합니다.\n    \n▶신규 가입 회원 혜택\n신규 가입 회원 혜택 정보 - 1\n신규 가입 회원 혜택 정보 - 2",
    "supplement": {
        "quick_reply": [
            {
                "type": "WL",
                "name": "홈페이지 바로가기",
                "url_mobile": "http://lunasoft.co.kr",
                "url_pc": "http://lunasoft.co.kr"
            },
            {
                "type": "WL",
                "name": "웹링크 바로가기",
                "url_mobile": "http://lunasoft.co.kr",
                "url_pc": "http://lunasoft.co.kr"
            }
        ]
    }
}'
```

#### 5.3.2 알림톡 바로연결 앱링크 버튼
+ 바로연결 앱링크 버튼 타입 : AL
+ 바로연결 앱링크 버튼은 클릭시 설정된 application custom scheme를 실행합니다.
+ **[ scheme_android, scheme_ios, url_mobile 중 2가지 필수 입력 ]**
<img src="/Images/Alimtalk/알림톡_바로연결_앱링크_버튼.png" alt="알림톡 바로연결 앱링크 버튼" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000011",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "180002",
    "message": "[루나소프트]\n안녕하세요. TEST님\n\n결제완료 되었습니다.\n감사합니다.\n\n- 날짜 : 2022-11-09 12:00\n- 주문번호 : 12345678-987654321\n\n고객센터\n(1644-4998)",
    "supplement": {
        "quick_reply": [
            {
                "type": "AL",
                "name": "쇼핑몰 APP",
                "scheme_android": "daumapps://open",
                "scheme_ios": "daumapps://open",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            }
        ]
    }
}'
```

#### 5.3.3 알림톡 바로연결 봇키워드 버튼
+ 봇키워드 버튼 타입 : BK
+ 바로연결 봇키워드 버튼은 클릭시 해당 버튼의 텍스트를 전송합니다.
<img src="/Images/Alimtalk/알림톡_바로연결_봇키워드_버튼.png" alt="알림톡 바로연결 봇키워드 버튼" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000012",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "180003",
    "message": "주문번호 : 12345678-987654321\n상품명 : 테스트 상품\n\n고객님!\n접수하신 #{문의}요청이 상담원 확인 중입니다.\n\n추가 확인이 필요한 경우, 상담원이 고객님께 연락드릴 수 있습니다.\n\n조금만 기다려주세요!",
    "supplement": {
        "quick_reply": [
            {
                "type": "BK",
                "name": "챗봇 시작하기"
            },
            {
                "type": "WL",
                "name": "쇼핑몰 둘러보기",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            }
        ]
    }
}'
```

### 5.4 강조표기 알림톡 발송 요청
+ 알림톡 내용에서 강조가 필요한 내용을 말풍선 상단 영역에 강조하여 표현이 가능합니다.
+ 타이틀 : 본문의 내용 중, 고객에게 강조하여 표현할 내용
+ 보조문구 : title에 어떤 내용이 들어가는지에 대한 설명(발송시 입력 안함)
<img src="/Images/Alimtalk/강조표기_알림톡.png" alt="강조표기 알림톡 발송" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000015",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "151016",
    "message": "[루나소프트]\nTEST 고객님의 카드 부분취소 요청건 처리되었습니다.\n\n환불금액 : 35,000원\n주문번호 : 12345-9876543\n\n루나소프트 lunasoft.co.kr\n고객센터 1644-4998",
    "title": "35,000원",
    "attachment": {
        "button": [
            {
                "type": "WL",
                "name": "쇼핑몰 바로가기",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            }
        ]
    }
}'
```

### 5.5 부가정보 포함 알림톡 발송 요청
+ 고객에게 고정적인 안내가 지속적으로 필요한 경우에 사용합니다.
+ 알림톡 메시지의 **본문 하단에 노출되며 이용안내 등 보조적인 정보메시지** 안내할 수 있습니다.
+ 부가정보는 최대 500자로 변수 사용이 불가능합니다. url 포함 가능합니다.
+ 부가정보는 템플릿 등록 시 입력할 수 있습니다.
<img src="/Images/Alimtalk/부가정보_포함_알림톡.png" alt="부가정보 포함 알림톡 발송" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000017",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "170022",
    "message": "[루나소프트]\nTEST님, 안녕하세요. \n2022년 11월 10일로 예약하신 객실 정보를 안내드립니다.\n\n■ 예약번호: 123456789\n■ 객실명 : 스카이라운지 1234호\n■ 예약일자 : 2022년 11월 10일 ~ 2022년 11월 15일\n\n- 루나소프트",
    "attachment": {
        "button": [
            {
                "type": "WL",
                "name": "쇼핑몰 바로가기",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            }
        ]
    }
}'
```

### 5.6 이미지 알림톡 발송 요청
+ 이미지 알림톡은 메시지 내 포함할 수 있는 이미지입니다.
+ 이미지에도 광고성 내용은 포함 될 수 없습니다.
+ 강조표기형과 동시에 사용할 수 없습니다.
+ 이미지 알림톡은 템플릿 당 하나의 고정된 이미지만 사용 가능하며, 이미지 제작 가이드를 준수하여 템플릿을 등록해야 합니다.
<img src="/Images/Alimtalk/이미지_알림톡.png" alt="이미지 알림톡 발송" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AI",
    "message_key": "0000000017",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "151021",
    "message": "TEST님 카카오 도시가스 입니다.\nTEST님의 도시가스요금이 자동이체되어 납부가 완료되었습니다.\n\n□ 고객명 : TEST\n□ 고객번호 : 123456789\n□ 청구요금: 35,000 원\n□ 납부요금: 35,000 원\n\n문의사항은 아래 버튼을 눌러 자세한 내용을 확인해주세요.",
    "attachment": {
        "button": [
            {
                "type": "WL",
                "name": "결제내역 확인하기",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            }
        ]
    }
}'
```

### 5.7 아이템리스트 알림톡 발송 요청
+ 기존 텍스트 알림톡 기본형에 ( 이미지 / 헤더 / 아이템 하이라이트 / 아이템리스트 / 아이템 요약정보) 5가지 항목이 추가로 구성됩니다.
+ 이미지, 헤더, 아이템 하이라이트 영역을 필요에 따라 1개 이상 필수 선택하여 템플릿 등록
(예) 헤더+아이템리스트+내용 or 이미지+아이템리스트+내용
+ 템플릿 당 고정된 이미지만 사용 가능합니다.
+ 아이템리스트는 최소 2개 이상 최대 10개 항목으로 구성 가능합니다.
+ 아이템리스트 알림톡 사용을 위해서는 기본적으로 이미지 알림톡 발송이 가능해야 합니다.
<img src="/Images/Alimtalk/아이템리스트_알림톡_발송.png" alt="아이템리스트 알림톡 발송" style="zoom:50%;" />

```
$ curl -X POST https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send 
-H 'Accept: application/json'
-H 'Content-type: application/json'
-H 'agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`'
-d '{
    "message_type": "AT",
    "message_key": "0000000020",
    "sender_key": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
    "country_code": "82",
    "recipient_number": "01012345678",
    "template_code": "ITEM_LIST_007",
    "message": "안녕하세요. 어피치님\n카카오맵, 인공지능 탑재 기념 EVENT 당첨을 축하드립니다.",
    "header": "카카오맵 이벤트 당첨 안내",
    "attachment": {
        "button": [
            {
                "type": "WL",
                "name": "주소 입력하기",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            },
            {
                "type": "WL",
                "name": "고객센터 문의하기",
                "url_mobile": "https://lunasoft.co.kr",
                "url_pc": "https://lunasoft.co.kr"
            }
        ],      
        "item_highlight": {
            "title": "카카오 미니 C",
            "description": "당첨 경품"
        },
        "item": {
            "list": [
            {
                "title": "이벤트명",
                "description": "인공지능 탑재 기념 EVENT"
            },
            {
                "title": "주소입력기한",
                "description": "2023.01.31"
            },
            {
                "title": "경품발송일",
                "description": "2023.02.01 순차 발송"
            }
            ]
        }
    }
}'
```


## 6. 코드 정의
### 6.1 오류 코드

| code | message | 설명 |
| ------ | ------ | ------ |
| 0000 |  | 정상코드<br/>- 전송 성공 및 과금 처리 |
| 1001 | NoJsonBody | Request Body가 Json형식이 아님 |
| 1002 | InvalidHubPartnerKey | 허브 파트너 키가 유효하지 않음 |
| 1003 | InvalidSenderKey | 발신 프로필 키가 유효하지 않음 |
| 1004 | NoValueJsonElement | Request Body(Json)에서 name을 찾을 수 없음 |
| 1006 | DeletedSender | 삭제된 발신프로필 |
| 1007 | StoppedSender | 차단 상태의 발신프로필 |
| 1011 | ContractNotFound | 계약정보를 찾을 수 없음 |
| 1012 | InvalidUserKeyException | 잘못된 형식의 유저키 요청 |
| 1013 | InvalidAppLink | 유효하지 않은 app연결 |
| 1014 | InvalidBizNum | 유효하지 않은 사업자번호 |
| 1015 | TalkUserIdNotFonud | 유효하지 않은 app user id 요청 |
| 1016 | BizNumNotEqual | 사업자등록번호 불일치 |
| 1012 | InvalidUserKeyException | 잘못된 형식의 유저키 요청 |
| 1020 | InvalidReceiveUser | 전화번호 or app user id가 유효하지 않거나 미입력 요청 |
| 1021 | BlockedProfile | 차단 상태의 카카오톡 채널 (카카오톡 채널 운영툴에서 확인)  |
| 1022 | DeactivatedProfile | 닫힘 상태의 카카오톡 채널 (카카오톡 채널 운영툴에서 확인)  |
| 1023 | DeletedProfile | 삭제된 카카오톡 채널 (카카오톡 채널 운영툴에서 확인)  |
| 1024 | DeletingProfile | 삭제대기 상태의 카카오톡 채널 (카카오톡 채널 운영툴에서 확인)  |
| 1025 | SpammedProfile | 채널 제재 상태로 인한 메시지 전송 실패 (카카오톡 채널 운영툴에서 확인) |
| 1027 | MessageSpammedProfileException | 채널 메시지 제재 상태로 인한 메시지 전송 실패 (카카오톡 채널 운영툴에서 확인) |
| 1030 | InvalidParameterException | 잘못된 파라메터 요청 |
| 2003 | FailedToSendMessageByNoFriendshipException | (테스트 발송) 카카오톡 채널을 추가하지 않았음 |
| 2004 | FailedToMatchTemplateException | 템플릿 일치 확인시 오류 발생(내부 오류 발생) |
| 2006 | FailedToMatchSerialNumberPrefixPattern | **[<4.2 메시지 전송 요청>](#42-메시지-전송-요청)**에 명시된 시리얼넘버 형식 불일치 |
| 3000 | UnexpectedException | 예기치 않은 오류 발생 |
| 3005 | AckTimeoutException | 메시지를 발송했으나 수신확인 안됨 (성공불확실)<br/>- 서버에는 암호화 되어 보관되며 3일 이내 수신 가능 |
| 3006 | FailedToSendMessageException | 내부 시스템 오류로 메시지 전송 실패 |
| 3008 | InvalidPhoneNumberException | 전화번호 오류 |
| 3010 | JsonParseException | Json 파싱 오류 |
| 3011 | MessageNotFoundException | 메시지가 존재하지 않음 |
| 3012 | SerialNumberDuplicatedException | 메시지 일련번호가 중복됨<br/>- 메시지 일련번호는 CS처리를 위해 고유한 값이 부여되어야 함. |
| 3013 | MessageEmptyException | 메시지가 비어 있음 |
| 3014 | MessageLengthOverLimitException | 메시지 길이 제한 오류<br/>(템플릿별 제한 길이 또는 1000자 초과) |
| 3015 | TemplateNotFoundException | 템플릿을 찾을 수 없음 |
| 3016 | NoMatchedTemplateException | 메시지 내용이 템플릿과 일치하지 않음 |
| 3018 | NoSendAvailableException | 메시지를 전송할 수 없음 |
| 3025 | ExceedMaxVariableLengthException | 변수 글자수 제한 초과 |
| 3026 | Button chat_extra(event)-InvalidExtra(EventName)Exception '([A-Za-z0-9_]{1,50})' | 상담/봇 전환 버튼 extra, event 글자수 제한 초과 |
| 3027 | NoMatchedTemplateButtonException / NoMatchedTemplateQuickReplyException | 메시지 버튼/바로연결이 템플릿과 일치하지 않음 |
| 3028 | NoMatchedTemplateTitleException | 메시지 강조 표기 타이틀이 템플릿과 일치하지 않음 |
| 3029 | ExceedMaxTitleLengthException | 메시지 강조 표기 타이틀 길이 제한 초과 (50자) |
| 3030 | NoMatchedTemplateWithMessageTypeException | 메시지 타입과 템플릿 강조유형이 일치하지 않음 |
| 3031 | NoMatchedTemplateHeaderException | 헤더가 템플릿과 일치하지 않음 |
| 3032 | ExceedMaxHeaderLengthException | 헤더 길이 제한 초과(16자) |
| 3033 | NoMatchedTemplateItemHighlightException | 아이템 하이라이트가 템플릿과 일치하지 않음 |
| 3034 | ExceedMaxItemHighlightTitleLengthException | 아이템 하이라이트 타이틀 길이 제한 초과(이미지 없는 경우 30자, 이미지 있는 경우 21자) |
| 3035 | ExceedMaxItemHighlightDescriptionLengthException | 아이템 하이라이트 디스크립션 길이 제한 초과(이미지 없는 경우 19자, 이미지 있는 경우 13자) |
| 3036 | NoMatchedTemplateItemListException | 아이템 리스트가 템플릿과 일치하지 않음 |
| 3037 | ExceedMaxItemDescriptionLengthException | 아이템 리스트의 아이템의 디스크립션 길이 제한 초과(23자) |
| 3038 | NoMatchedTemplateItemSummaryException | 아이템 요약정보가 템플릿과 일치하지 않음 |
| 3039 | ExceedMaxItemSummaryDescriptionLengthException | 아이템 요약정보의 디스크립션 길이 제한 초과(14자) |
| 3040 | InvalidItemSummaryDescriptionException | 아이템 요약정보의 디스크립션에 허용되지 않은 문자 포함(통화기호/코드, 숫자, 콤마, 소수점, 공백을 제외한 문자 포함) |
| 4000 | ResponseHistoryNotFoundException | 메시지 전송 결과를 찾을 수 없음 |
| 4001 | UnknownMessageStatusError | 알 수 없는 메시지 상태 |
| 5000 | InvalidTestUser | (테스트 발송) 관리자 혹은 일회성 인증을 받은 사용자가 아님 |
| 5001 | DailyTestLimitExceeded | (테스트 발송) 일일 발송량 초과 |
| 9998 | 현재 서비스를 제공하고 있지 않습니다. | 시스템에 문제가 발생하여 담당자가 확인하고 있는 경우 |
| 9999 | 시스템에서 알 수 없는 문제가 발생하였습니다. <br/>담당자가 확인 중입니다. | 시스템에 문제가 발생하여 담당자가 확인하고 있는 경우 |
| LS0000 | 발송 요청 성공 | 발송 요청 성공 |
| LS0001 | 잘못된 요청 JSON 또는 Querystring | 잘못된 요청 JSON 또는 Querystring |
| LS0002 | 유효하지 않은 AgentKey | 유효하지 않은 AgentKey |
| LS0101 | 외부 서비스 API 타임 아웃 | 카카오톡 비즈메시지 알림톡 API 타임 아웃 |
| LS0102 | 외부 서비스 API Http Exception 발생 | 카카오톡 비즈메시지 알림톡 API Http Exception 발생 |
| LS9999 | 루나소프트 API 내부 시스템 에러 | 루나소프트 API 내부 시스템 에러 |

## 7. 테스트 방법

### 7.1 Host
**알림톡 테스트는 개발환경에서만 가능합니다.**
> ####**[개발서버]** `https://test-kakao-bizmessage.lunasoft.co.kr/`

### 7.2 테스트 선결 조건
+ 테스트할 서버 IP 정보를 루나소프트 운영 담당자에게 전달하여 테스트 서버 ACL(Access Control List)에 등록한 후 테스트 서버에 접속할 수 있습니다.
+ 테스트 카카오톡 채널은 비즈니스 인증이 완료된 채널을 개발서버에 직접 등록하여 발송 가능합니다.
+ 테스트 템플릿은 개발서버에 직접 등록하여 [직접 승인](https://github.com/lunasoft-org/kakao-bzm-center/blob/main/center-api.md#381-%EC%8A%B9%EC%9D%B8-%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%9A%A9) 후 발송 가능합니다.
+ 테스트 수신자는 허브파트너의 관리자에 포함되어 있거나, [일회성 인증](https://github.com/lunasoft-org/kakao-bzm-center/blob/main/center-api.md#5-%EA%B0%9C%EB%B0%9C-%EC%84%9C%EB%B2%84-%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%9D%B8%EC%A6%9D)을 받은 사용자만 가능합니다.
+ 테스트 수신자는 테스트를 위한 카카오톡 채널을 추가해야 합니다.
+ 하루에 발송 가능한 테스트 발송량은 허브파트너당 발송 성공 기준 알림톡 500건입니다.
+ 테스트 발송 시 메시지 내 텍스트 영역에 자동으로 [테스트발송] 말머리를 포함하여 발송되므로, 메시지 총 글자수가 1000자 이상일 경우 텍스트가 잘릴 수 있습니다. 최대 글자수에서 약 10글자 내외 여유를 두고 작성하시어 테스트 부탁 드립니다.

### 7.3 메시지 전송 요청

#### 7.3.1 메시지 전송 방식

활성 사용자에게 메시지 전송
```
$ curl  -H "Accept: application/json" -H "Content-type: application/json" -H "agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`" -X POST -d '{"message_type":"AT","message_key":"2017010100004","sender_key":"2662e99eb7a1f21abb3955278e9955f5a9a99b62","country_code":"82","recipient_number":"821012345678","template_code":"A001_01","message":"임꺽정님이 보낸 등기 1234567_89123456를 홍길동(회사동료)님께 배달 완료 1588-1300"}' https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send

{"code":"0000","message_key":"2017010100004"}
```

#### 7.3.2 메시지 전송 오류
1) 일련번호 중복
```
$ curl  -H "Accept: application/json" -H "Content-type: application/json" -H "agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`" -X POST -d '{"message_type":"AT","message_key":"2017010100004","sender_key":"2662e99eb7a1f21abb3955278e9955f5a9a99b62","country_code":"82","recipient_number":"821012345678","template_code":"A001_01","message":"임꺽정님이 보낸 등기 1234567_89123456를 홍길동(회사동료)님께 배달 완료 1588-1300"}' https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send

{"code":"3012","message_key":"2017010100004","message":"SerialNumberDuplicatedException(20170101-00001)"}
```
2) 템플릿 없음
```
$ curl  -H "Accept: application/json" -H "Content-type: application/json" -H "agentKey: `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey`" -X POST -d '{"message_type":"AT","message_key":"2017010100004","sender_key":"2662e99eb7a1f21abb3955278e9955f5a9a99b62","country_code":"82","recipient_number":"821012345678","template_code":"A001_01","message":"임꺽정님이 보낸 등기 1234567_89123456를 홍길동(회사동료)님께 배달 완료 1588-1300"}' https://test-bizmessage.lunasoft.co.kr/api/v3/alimtalk/send

{"code":"3015","message_key":"2017010100004","message":"TemplateNotFoundException(A001_02)"}
```
