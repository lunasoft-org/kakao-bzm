# 카카오 비즈메시지 센터 API v1.0.0

1. [공통](#1-공통)
2. [발신프로필 관리](#2-발신프로필-관리)
3. [템플릿 관리](#3-템플릿-관리)
4. [발신 프로필 그룹 관리](#4-발신-프로필-그룹-관리)
5. [(개발 서버) 테스트 사용자 인증](#5-개발-서버-테스트-사용자-인증)


## 참고 : 변경사항

| 일시       | 변경 내역                                                                                                                                                                                                                |
| ---------- | -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2022.10.    | 카카오 비즈메시지 센터 API 초안 작성                 


## 1. 공통
- 비즈메시지 센터 API는 https 프로토콜을 사용하며, GET, POST method를 사용합니다. response는 JSON형태로 전달됩니다.
- 비즈메시지 센터 API는 루나소프트에 등록된 고객사에 대해 허용된 IP로 접근할 수 있는 제한적으로 오픈된 서비스를 제공한다.
- 테스트용 이라고 적힌 항목은 개발을 위한 API로 개발서버에서만 사용가능 합니다.
- 루나소프트 운영 담당자를 통해 요청 시 필요한 `AgentKey`를 발급 받아야 한다. (요청 시 header에 추가)


### 1.1. Host
> ####**[개발서버]** `https://test-kakao-bizcenter.lunasoft.co.kr/`

> ####**[운영서버]**  `https://kakao-bizcenter.lunasoft.co.kr/`

### 1.2. RESPONSE SAMPLE
##### [성공]
```
{
    "code": "200",
    "data": {
        ... // 요청한 데이터
    }
}
```

##### [잘못된 요청 또는 에러]
```
{
    "code": "500",
    "message": "상세 내용"
}
```

### 1.3. 코드 정의
| code    | 설명 |
| ------- | --- |
| 200     | 요청 성공 |
| 403     | 권한 없음 |
| 405     | 파라미터 오류 |
| 504     | 템플릿 코드 중복 |
| 505     | 템플릿 이름 중복 |
| 506     | 템플릿 내용이 1000자 초과 |
| 507     | 유효하지 않은 발신 프로필 |
| 508     | 요청한 데이터가 없음, 삭제 상태의 데이터 요청 시 응답 |
| 509     | 요청을 처리할수 있는 상태가 아님 (ex: 템플릿 검수 요청이 가능한 상태가 아닙니다.) |
| 510     | 템플릿의 버튼/바로연결 형식이 유효하지 않음 |
| 525     | 템플릿의 카테고리가 유효하지 않음 |
| 512     | 허브파트너는 발신프로필 추가 및 그룹내 발신프로필 추가가 제한된 상태 |
| 513     | 메시지 결과 수신 채널이 올바르지 않음 |
| 514     | 비즈니스 인증이  필요한 카카오톡 채널 |
| 530     | 유효하지 않은 플러그인 |
| 600     | 이미지 업로드 실패 |
| 610     | 파일 업로드 실패 |
| 611     | 첨부파일의 크기가 50MB를 초과 |
| 612     | 첨부파일 형식이 유효하지 않음 |
| 613     | 첨부파일의 개수가 10개를 초과 |
| 614     | 첨부파일이 존재 하지 않음 |
| 801~805 | 발신프로필 등록이 차단된 상태 |
| 811     | 발신프로필 등록이 차단된 허브파트너 |


## 2. 발신프로필 관리
### 2.1. 카카오톡 채널 인증 토큰 요청
- 발신프로필 등록을 위한 카카오톡 채널 인증 토큰을 요청하는 API 입니다.
- 입력한 전화번호에 연결된 카카오톡으로 인증 토큰이 발송되며, 발송된 토큰은 발신프로필 등록시 사용됩니다.
- 인증받은 토큰은 7일동안 비즈메시지 센터 서버에 보관됩니다.

##### [Request]
**GET** /api/v1/sender/token

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| yellowId    | text | Y   | 카카오톡 채널 (예: @에릭커피) |
| phoneNumber | text | Y   | 카카오톡 채널 알림받는 관리자 핸드폰 번호 |

##### [Response]
| 키   | 타입  | 설명 |
| ---- | -----| --- |
| code | text | 결과 코드 |

### 2.2. 발신프로필 카테고리 전체 조회
발신프로필 등록시 사용할 카테고리 목록 전체를 조회합니다.

##### [Request]
**GET** /api/v1/category/all

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Response]
| 키   | -             | 타입    | 설명 |
| ---- | ------------- | ------- | --- |
| code |               | text    | 결과 코드 |
| data |               | list    |          |
|      | code          | text(11)| 카테고리 코드 |
|      | name          | text    | 카테고리 이름 |


### 2.3. 발신프로필 카테고리 조회
카테고리 코드에 해당하는 특정 카테고리를 조회합니다.

##### [Request]
**GET** /api/v1/category

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키          | 타입   | 필수  | 설명 |
| ----------- | ----- | --- | --- |
| categoryCode | text(11) | Y   | 카테고리 코드 |

##### [Response]
| 키   | -             | 타입    | 설명 |
| ---- | ------------- | ------- | --- |
| code |               | text    | 결과 코드 |
| data |               | object    |          |
|      | code          | text(11)| 카테고리 코드 |
|      | name          | text    | 카테고리 이름 |

## 2.4. 등록
- 발신프로필을 신규 등록합니다.
- 사전에 카카오톡 채널이 생성되어 있어야 하며, [카카오톡 채널 관리자센터](https://center-pf.kakao.com/)에서 비즈니스 인증을 받아야 합니다.
- 카카오톡 채널은 프로필 activated 상태이고 운영자에 의해 차단되지 않은 정상 상태여야 등록 가능합니다.
- 카카오톡 채널로 관리자 인증 토큰([2.1. 카카오톡 채널 인증 토큰 요청](#21-카카오톡-채널-인증-토큰-요청)) 을 등록전 받아야 하며, 토큰을 요청한 전화번호로만 발신프로필로 등록 가능합니다.
- [2.2. 카테고리 전체 목록 조회 API](#22-발신프로필-카테고리-전체-조회)를 사용하여 입력할 카테고리 코드 목록을 조회할 수 있으며, 카테고리 정보를 함께 등록할 수 있습니다.

##### [Request]
**POST** /api/v3/sender/create

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |
| token       | text | Y   | 2.1에서 카카오톡으로 받은 토큰값 |
| phoneNumber | text | Y   | 2.1에서 토큰 요청시 입력한 핸드폰 번호 |

##### [Parameter]
| 키           | 타입  | 필수  | 설명 |
| ------------ | ---- | --- | --- |
| yellowId     | text | Y   | 등록할 카카오톡 채널 (예: @에릭커피) |
| categoryCode | text(11) | Y   | 카테고리 코드 (11자리 숫자) |
| channelKey   | text(20) | N   | 메시지 전송 결과 수신 채널 |


##### [Example]
```
curl -X POST \
  -H "Content-type:application/json" \
  -H 'agentKey: {사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey}' \
  -H 'phoneNumber: {센터에 등록된 운영자 카카오 계정의 인증 전화번호}' \
  -H 'token: {카카오톡 채널 인증 토큰}' \
  -d '{"yellowId":"{카카오톡 채널 UUID}", "categoryCode":"{카테고리 코드}"}' \
  https://test-bizcenter.lunasoft.co.kr/api/v3/sender/create
```

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키   | -             | 타입     | 설명 |
| ---- | ------------- | ------- | --- |
| code |               | text    | 결과 코드 |
| data | senderKey     | text(40)| 발신프로필키 |
|      | uuid          | text(20)| 카카오톡 채널 |
|      | name          | text(20)| 카카오톡 채널 프로필명 |
|      | status        | text(1) | 상태(A:정상) |
|      | block         | boolean | 발신프로필 차단 여부 |
|      | dormant       | boolean | 발신프로필 휴면 여부 |
|      | profileStatus | text(1) | 카카오톡 채널 상태 (A:activated, C:deactivated, B:block, E:deleting, D:deleted) |
|      | createdAt     | text(19)| 등록일 |
|      | modifiedAt    | text(19)| 최종수정일 |
|      | categoryCode  | text(11)| 카테고리 코드
|      | alimtalk      | boolean | 알림톡 사용 여부 |
|      | bizchat       | boolean | 상담톡 사용 여부 |
|      | brandtalk     | boolean | 브랜드톡 사용 여부 |
|      | commitalCompanyName | text(100) | 위탁사 이름 (상담톡 관련) |
|      | channelKey    | text(20)| 메시지 전송 결과 수신 채널키 |

##### [Example]
```
{
    "code": "200",
    "data": {
        "senderKey": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
        "uuid": "@dkfflaxhrxptmxm",
        "name": "bzmtest",
        "status": "A",
        "block": false,
        "dormant":false,
        "profileStatus": "A",
        "createdAt": "2015-08-21 17:58:24",
        "modifiedAt": "",
        "categoryCode": "99999999999",
        "channelKey": "test"
    }
}
```

## 2.5. 조회
발신프로필 정보를 조회합니다.

##### [Request]
**GET** /api/v3/sender

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키        | 타입 | 필수  | 설명 |
| --------- | --- | --- | --- |
| senderKey | text(40) | Y  | 등록된 발신프로필의 키 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키   | -             | 타입    | 설명 |
| ---- | ------------- | ------- | --- |
| code |               | text    | 결과 코드 |
| data | senderKey     | text(40)| 발신프로필키 |
|      | uuid          | text(20)| 카카오톡 채널 |
|      | name          | text(20)| 카카오톡 채널 프로필명 |
|      | status        | text(1) | 상태(A:정상) |
|      | block         | boolean | 발신프로필 차단 여부 |
|      | dormant       | boolean | 발신프로필 휴면 여부 |
|      | profileStatus | text(1) | 카카오톡 채널 상태 (A:activated, C:deactivated, B:block, E:deleting, D:deleted) |
|      | createdAt     | text(19)| 등록일 |
|      | modifiedAt    | text(19)| 최종수정일 |
|      | categoryCode  | text| 카테고리 코드
|      | alimtalk      | boolean | 알림톡 사용 여부 |
|      | bizchat       | boolean | 상담톡 사용 여부 |
|      | brandtalk     | boolean | 브랜드톡 사용 여부 |
|      | commitalCompanyName | text(100) | 위탁사 이름 (상담톡 관련) |
|      | channelKey    | text(20)| 메시지 전송 결과 수신 채널키 |
|      | businessProfile | boolean | 카카오톡 채널 비즈니스 인증 여부 |
|      | businessType  | text(50)| 카카오톡 채널 비즈니스 인증 타입 |

##### [Example]
```
{
    "code": "200",
    "data": {
        "senderKey": "2662e99eb7a1f21abb3955278e9955f5a9a99b62",
        "uuid": "@dkfflaxhrxptmxm",
        "name": "bzmtest",
        "status": "A",
        "block": false,
        "dormant":false,
        "profileStatus": "A",
        "createdAt": "2015-08-21 17:58:24",
        "modifiedAt": "",
        "categoryCode": "99999999999",
        "channelKey": "test"
    }
}
```

### 2.6. 삭제
발신프로필 삭제는 루나소프트 운영 담당자를 통해 수동으로 삭제 가능합니다.
발신프로필 삭제 시 발신프로필 하위에 등록된 모든 템플릿도 삭제되며, 삭제 후에는 복구가 불가능하니 꼭 필요한 경우에만 사용해주세요.


### 2.7. 미사용 프로필 휴면 해제
장기 미사용으로 인해 휴면 상태인 발신 프로필을 차단 해제합니다.

##### [Request]
**POST** /api/v1/sender/recover

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키   | 타입 | 필수 | 설명 |
| --------- | ---- | --- | --- |
| senderKey | text(40) | Y   | 등록된 발신프로필의 키 |

##### [Response]
| 키| 타입 | 설명 |
| ---- | ---- | --- |
| code | text | 결과 코드 |


## 3. 템플릿 관리
### 3.1. 템플릿 등록
#### 3.1.1  템플릿 등록

- 템플릿을 신규 등록합니다. 사전에 발신프로필 또는 발신 프로필 그룹이 등록 되어있어야 합니다.
- [3.9.1. 템플릿 카테고리 전체 조회 API](#391-템플릿-카테고리-전체-조회)를 사용하여 입력할 카테고리 코드 목록을 조회할 수 있으며, 카테고리 정보를 함께 등록할 수 있습니다.

> 템플릿 검수 가이드 : https://kakaobusiness.gitbook.io/main/ad/bizmessage/notice-friend/audit

##### [Request]
**POST** /api/v2/alimtalk/template/create

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키              | 타입        | 필수 | 설명 |
| --------------- | ---------- | --- | --- |
| senderKey       | text(40)   | Y   | 발신 프로필 키(senderKeyType이 G인경우발신 프로필그룹키) |
| senderKeyType   | text(1)    | N   | 발신 프로필 키 타입 (G:그룹 , S:발신프로필(default)) |
| templateCode    | text(30)   | Y   | 템플릿 코드 |
| templateName    | text(200)  | Y   | 템플릿 이름 |
| templateMessageType | text   | Y   | 템플릿 메시지 유형<br/>(BA: 기본형, EX: 부가 정보형, AD: 채널 추가형, MI: 복합형)<br/>- EX : templateExtra 필드 필수<br>- AD : 그룹 템플릿 사용 불가<br>- MI : templateExtra 필드 필수, 그룹 템플릿 사용 불가 |
| templateEmphasizeType | text | Y   | 템플릿 강조 유형<br/>(NONE: 선택안함, TEXT: 강조표기형, IMAGE: 이미지형, ITEM_LIST: 아이템리스트형)<br/>- TEXT: templateTitle, templateSubtitle 필드 필수<br/>- ITEM_LIST: templateItem.list, (templateImage(Name, Url), templateHeader, templateItemHighlight 필드 중 1개 이상 필수, templateItem.summary 필드는 templateItem.list  사용시에만 사용 가능 (2022-08-30 이후이며 그 전까지 list 필수)<br>-IMAGE: templateImageName, templateImageUrl 필드 필수 |
| templateContent | text       | Y   | 템플릿 내용 |
| templateExtra   | text       | N   | 부가 정보(템플릿 검수 가이드 참고) |
| templateImageName | text     | N   | 템플릿 이미지 파일명 (템플릿 검수 가이드 참고, [업로드 API](https://github.com/lunasoft-org/kakao-bzm/blob/main/upload-api.md#22-알림톡-템플릿-등록용-이미지-업로드-요청) 참고) |
| templateImageUrl | text      | N   | 템플릿 이미지 링크 (템플릿 검수 가이드 참고, [업로드 API](https://github.com/lunasoft-org/kakao-bzm/blob/main/upload-api.md#22-알림톡-템플릿-등록용-이미지-업로드-요청)) 참고) |
| templateTitle   | text       | N   | 템플릿 내용 중 강조 표기할 핵심 정보 (템플릿 검수 가이드 참고) |
| templateSubtitle | text      | N   | 강조 표기 보조 문구 (템플릿 검수 가이드 참고) |
| templateHeader  | text(16))  | N   | 헤더 (템플릿 검수 가이드 참고) |
| templateItemHighlight | object | N | 아이템 하이라이트 (템플릿 검수 가이드 참고, [상세](#3111-templateItemHighlight-상세-요청-파라미터)) |
| templateItem    | object     | N   | 아이템 리스트 (템플릿 검수 가이드 참고, [상세](#3112-templateItem-상세-요청-파라미터)) |
| categoryCode    | text   | N   | 템플릿 카테고리 코드 ([3.9.1. 템플릿 카테고리 전체 조회 API](#391-템플릿-카테고리-전체-조회) 참고) |
| securityFlag    | boolean    | N   | 보안 템플릿 여부,  OTP등 보안 메시지 일 경우 설정<br/>발신 당시의 메인 디바이스를 제외한 모든 디바이스에 메시지 텍스트 미노출<br/> (디폴트: 미설정)|
| buttons         | list       | N   | 버튼 정보 (최대 5개 등록 가능 (단, 바로연결과 함께 사용 시 2개로 제한됨), [상세](#3113-buttons-상세-요청-파라미터))|
| quickReplies    | list       | N   | 바로연결 정보 (최대 10개 등록 가능, [상세](#3114-quickReplies-상세-요청-파라미터))|
+ 템플릿 메시지 타입이 채널 추가형 (AD), 복합형 (MI)인 경우 기존의 광고 문구 영역이 "채널 추가하고 이 채널의 광고와 마케팅 메시지를 카카오톡으로 받기" 고정되며 수정 불가

##### 3.1.1.1 templateItemHighlight 상세 요청 파라미터
| 키                 | -   | 타입    | 필수 | 설명 |
| ------------------ | --- | ------ | --- | --- |
| templateItemHighlight |  | object    |     | 아이템 하이라이트 (템플릿 검수 가이드 참고) |
|            | title       | text(30)  | Y   | 타이틀 (썸네일 이미지가 있을 땐 21자) |
|            | description | text(19)  | Y   | 디스크립션 (썸네일 이미지가 있을 땐 13자) |
|            | imageUrl    | text(500) | N   | 썸네일 이미지 주소 ([업로드 API](https://github.com/lunasoft-org/kakao-bzm/blob/main/upload-api.md#23-알림톡-이미지-업로드-요청) 참고) |

##### 3.1.1.2 templateItem 상세 요청 파라미터
| 키      | -   | -   | 타입       | 필수 | 설명 |
| ------- | --- | --- | -------- | --- | --- |
| templateItem |  | | object   |     | 아이템 리스트 (템플릿 검수 가이드 참고) |
|  | list         | | array    | N   | 아이템 리스트 (최소 2개, 최대 10개) (2022-08-30 부터이며 그 이전에는 Y) |
|  |  | title       | text(6)  | Y   | 타이틀 |
|  |  | description | text(23) | Y   | 디스크립션 |
|  | summary      | | object   | N   | 아이템 요약 정보 |
|  |  | title       | text(6)  | Y   | 타이틀 |
|  |  | description | text(14) | Y   | 디스크립션 (변수 및 화폐 단위, 숫자, 쉼표, 마침표만 사용 가능) |

##### 3.1.1.3 buttons 상세 요청 파라미터
| 키    | -   | 타입    | 필수 | 설명 |
| ----- | --- | ------ | --- | --- |
| buttons  |  | array     |   | 버튼 목록 |
|  | name     | text(14)  | Y | 버튼명 (단, AC인 경우 제외) |
|  | linkType | text(2)   | Y | 버튼 타입 아래 **button 타입별 속성**에서 확인 가능 |
|  | ordering | number    | N | 버튼 노출 순서
|  | linkMo  | text(500) | - | mobile 환경에서 버튼 클릭 시 이동할 url |
|  | linkPc  | text(500) | - | pc 환경에서 버튼 클릭 시 이동할 url |
|  | linkAnd   | text(500) | - | mobile android 환경에서 버튼 클릭 시 실행할<br/>application custom scheme |
|  | linkIos   | text(500) | - | mobile ios 환경에서 버튼 클릭 시 실행할<br/>application custom scheme |
|  | pluginId | text(24)  | - | 플러그인 ID |
|  | bizFormId | number  | - | 비즈니스폼 ID |
+ 템플릿 메시지 타입이 채널 추가형 (AD), 복합형 (MI)인 경우 채널 추가(AC) 버튼이 필수이며 첫번째로 위치해야 한다.
+ 채널 추가(AC) 버튼의 경우 버튼명이 "채널 추가"로 고정되며 name 파라미터를 안 보내도 된다. (name을 같이 넘겨도 되지만 사용하지 않음)
+ 비즈니스폼(BF) 버튼의 버튼명은 비즈니스폼 유형에 따라 "톡에서 설문하기", "톡에서 응모하기", "톡에서 예약하기" 중 한가지만 사용가능



##### button 타입별 속성

+ 필수 파라메터를 모두 입력하셔야 정상적인 발송이 가능합니다.

| 버튼타입 | 속성  | 타입 | 필수 | 설명 |
| --- | ------- | ---- | --- | --- |
| AC | -        | -    | - | 채널 추가 **(템플릿 메시지 유형이 AD, MI인 경우에만 사용 가능)** |
| WL | linkMo   | text | Y | 버튼 클릭 시 이동할 pc/mobile환경별 web url |
|    | linkPc   | text | N | |
| AL | linkAnd  | text | - | **linkIos, linkAnd, linkMo 중 2가지 필수 입력**<br/>mobile android 환경에서 버튼 클릭 시 실행할 application custom scheme |
|    | linkIos  | text | - | mobile ios 환경에서 버튼 클릭 시 실행할 application custom scheme |
|    | linkMo   | text | - | mobile 환경에서 버튼 클릭 시 이동할 url |
|    | linkPc   | text | N | pc 환경에서 버튼 클릭 시 이동할 url |
| BK | -        | -    | - | 해당 버튼 텍스트 전송 |
| MD | -        | -    | - | 해당 버튼 텍스트 + 메시지 본문 전송 |
| BC | -        | -    | - | 상담톡 전환 |
| BT | -        | -    | - | 봇 전환 |
| DS | -        | -    | - | 배송 조회 |
| P1 | pluginId | text | Y | 이미지 보안 전송 플러그인 ID |
| P2 | pluginId | text | Y | 개인정보이용 플러그인 ID |
| P3 | pluginId | text | Y | 원클릭 결제 플러그인 ID |
| BF | bizFormId | number | Y | 비즈니스폼 ID |

##### 3.1.1.4 quickReplies 상세 요청 파라미터
| 키           | -    | 타입   | 필수 | 설명 |
| ------------ | --- | ------ | --- | --- |
| quickReplies |  | array     | - | 바로연결 목록 |
|  | name         | text(14)  | Y | 바로연결명 |
|  | linkType     | text(2)   | Y | 바로연결 타입 아래 **quickReplies 타입별 속성**에서 확인 가능 |
|  | linkMo       | text(500) | - | mobile android 환경에서 버튼 클릭 시 실행할<br/>application custom scheme |
|  | linkPc       | text(500) | - | mobile ios 환경에서 버튼 클릭 시 실행할<br/>application custom scheme |
|  | linkAnd      | text(500) | - | pc 환경에서 버튼 클릭 시 이동할 url |
|  | linkIos      | text(500) | - | mobile 환경에서 버튼 클릭 시 이동할 url |
|  | bizFormId    | number    | - | 비즈니스폼 ID |



+ 비즈니스폼(BF) 바로연결의 바로연결명은 비즈니스폼 유형에 따라 "톡에서 설문하기", "톡에서 응모하기", "톡에서 예약하기" 중 한가지만 사용가능



##### quickReplies 타입별 속성

+ 필수 파라메터를 모두 입력하셔야 정상적인 발송이 가능합니다.

| 바로연결타입 | 속성 | 타입 | 필수 | 설명 |
| --- | ------ | ---- | --- | --- |
| WL | linkMo  | text | Y | 바로연결 클릭 시 이동할 pc/mobile환경별 web url |
|    | linkPc  | text | N | |
| AL | linkAnd | text | - | **linkIos, linkAnd, linkMo 중 2가지 필수 입력**<br/>mobile android 환경에서 바로연결 클릭 시 실행할 application custom scheme |
|    | linkIos | text | - | mobile ios 환경에서 바로연결 클릭 시 실행할 application custom scheme |
|    | linkMo  | text | - | mobile 환경에서 바로연결 클릭 시 이동할 url |
|    | linkPc  | text | N | pc 환경에서 바로연결 클릭 시 이동할 url |
| BK | -       | -    | - | 해당 바로연결 텍스트 전송 |
| BC | -       | -    | - | 상담톡 전환 |
| BT | -       | -    | - | 봇 전환 |
| BF | bizFormId | number | Y | 비즈니스폼 ID |


##### [Example]
```
curl -v -X POST
    -d 'senderKey={senderKey}'
    -d 'senderKeyType=S'
    -d 'templateCode=center_api_test_02'
    -d 'templateName=상품발송안내02'
    -d 'templateMessageType=BA'
    -d 'templateEmphasizeType=NONE'
    -d 'templateContent=#{홍길동}님 주문하신 #{가습기} 금 일 배송 출발 하였습니다.'
    -d 'buttons[0].ordering=1'
    -d 'buttons[0].linkType=AC'
    -d 'buttons[0].name=채널추가'
    -d 'buttons[1].ordering=1'
    -d 'buttons[1].linkType=DS'
    -d 'buttons[1].name=택배조회'
    -d 'buttons[2].ordering=3'
    -d 'buttons[2].linkType=WL'
    -d 'buttons[2].name=바로가기'
    -d 'buttons[2].linkMo=http://daum.net'
    -d 'quickReplies[0].linkType=BK'
    -d 'quickReplies[0].name=봇키워드'
    -d 'quickReplies[1].linkType=WL'
    -d 'quickReplies[1].name=바로가기'
    -d 'quickReplies[1].linkMo=http://daum.net'
https://test-bizcenter.lunasoft.co.kr/api/v2/alimtalk/template/create
```

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키   | 타입     | 설명 |
| ---- | ------ | --- |
| code | text   | 결과 코드 |
| data | object | [템플릿 상세](#321-템플릿-상세-결과) 참고 |


### 3.2. 조회
템플릿을 조회합니다.

##### [Request]
**GET** /api/v2/alimtalk/template

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키 | 타입 | 필수 | 설명 |
| ------------- | ---- | --- | --- |
| senderKey     | text(40) | Y  | 발신 프로필 키 |
| senderKeyType | text(1) | N  | 발신프로필타입 (G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y  | 템플릿 코드 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키   | 타입     | 설명 |
| ---- | ------ | --- |
| code | text   | 결과 코드 |
| data | object | [템플릿 상세](#321-템플릿-상세-결과) 참고 |

##### [Example]
```
{
    "code": "200",
    "data": {
        "senderKey": "{senderKey}",
        "senderKeyType": "S",
        "templateCode": "center_api_test_02",
        "templateName": "상품발송안내02",
        "templateMessageType": "BA",
        "templateEmphasizeType": "IMAGE",
        "templateContent": "#{홍길동}님 주문하신 #{가습기} 금 일 배송 출발 하였습니다.",
        "templateImageName": "image.jpg",
        "templateImageUrl": "http://kage.kakao.com/dn/xxxxx/xxxxxxxxx/xxxxxxxxxxxxxx/xxxxx.jpg",
        "inspectionStatus": "REG",
        "createdAt": "2017-01-25 11:13:40",
        "modifiedAt": "",
        "status": "R",
        "block": false,
        "dormant":false,
        "categoryCode": "002001",
        "securityFlag": true,
        "comments": [
            {
                "id": 1,
                "content": "승인되었습니다.",
                "userName": "홍길동",
                "createdAt": "2017-01-26 11:46:14",
                "status": "APR",
                "attachment": [
                    {
                        "originalFileName": "sample.jpg",
                        "filePath": "http://kage.kakao.com/dn/xxxxx/xxxxxxxxx/xxxxxxxxxxxxxx/xxxxx.jpg"
                    }
                ]
            }
        ],
        "buttons": [ {
            "ordering":1,
	    "name":"채널 추가",
	    "linkType":"AC",
            "linkTypeName":"채널 추가",
            "linkMo": null,
            "linkPc": null,
            "linkIos": null,
            "linkAnd": null
         }, {
            "ordering": 2,
            "name": "택배조회",
            "linkType": "DS",
            "linkTypeName": "배송조회",
            "linkMo": null,
            "linkPc": null,
            "linkIos": null,
            "linkAnd": null
        }, {
            "ordering": 3,
            "name": "바로가기",
            "linkType": "WL",
            "linkTypeName": "웹링크",
            "linkMo": "http://daum.net",
            "linkPc": null,
            "linkIos": null,
            "linkAnd": null
        }]
    }
}
```


#### 3.2.1 템플릿 상세 결과
| 키    | -               | 타입 | 설명 |
| ---- | ---------------- | ------- | --- |
| code |                  | text    | 결과 코드 |
| data | senderKey        | text(40)    | 발신프로필키 |
|      | senderKeyType    | text(1)    | 발신프로필타입(S:기본, G:발신프로필그룹) |
|      | templateCode     | text(30)    | 템플릿코드 |
|      | templateName     | text(200)    | 템플릿이름 |
|      | templateMessageType | text | 템플릿 메시지 유형<br/>(BA: 기본형, EX: 부가 정보형, AD: 채널 추가형, MI: 복합형) |
|      | templateEmphasizeType | text | 템플릿 강조표기 유형<br/>(NONE: 선택안함, TEXT: 강조표기형, IMAGE: 이미지형, ITEM_LIST: 아이템리스트형) |
|      | templateContent  | text    | 템플릿내용 |
|      | templateExtra    | text    | 부가 정보(템플릿 검수 가이드 참고, optional) |
|      | templateImageName | text   | 템플릿 이미지 파일명 (템플릿 검수 가이드 참고, optional) |
|      | templateImageUrl | text    | 템플릿 이미지 링크 (템플릿 검수 가이드 참고, optional) |
|      | templateTitle    | text    | 템플릿 내용 중 강조 표기할 핵심 정보 (템플릿 검수 가이드 참고, optional) |
|      | templateSubtitle | text    | 강조 표기 보조 문구 (템플릿 검수 가이드 참고, optional) |
|      | templateHeader   | text    | 헤더 (템플릿 검수 가이드 참고, optional) |
|      | templateItemHighlight | object | 아이템 하이라이트 (템플릿 검수 가이드 참고, optional, [상세](#3211-templateItemHighlight-상세-결과)) |
|      | templateItem     | object  | 아이템 리스트 (템플릿 검수 가이드 참고, optional, [상세](#3212-templateItem-상세-결과)) |
|      | inspectionStatus | text    | 검수 상태(REG:등록,  REQ:심사요청, APR:승인, REJ: 반려) |
|      | createdAt        | text(19)    | 등록일 |
|      | modifiedAt       | text(19)    | 수정일 |
|      | status           | text(1)    | 템플릿 상태(S:중지, A:정상, R:대기(발송전)) |
|      | block            | boolean | 템플릿 차단 여부(true:차단, false:해제) |
|      | dormant          | boolean | 템플릿 휴면 여부 |
|      | categoryCode     | text    | 템플릿 카테고리코드 |
|      | securityFlag     | boolean | 보안 템플릿 여부 (true:설정, false:미설정) |
|      | comments         | list    | 검수결과 댓글리스트 (아래 comments 상세표 참고, [상세](#3213-comments-상세-결과)) |
|      | buttons          | list    | 버튼 정보 ([상세](#3214-buttons-상세-결과)) |
|      | quickReplies     | list    | 바로연결 정보 ([상세](#3215-quickReplies-상세-결과)) |

##### 3.2.1.1 templateItemHighlight 상세 결과
| 키          | 타입 | 설명 |
| ----------- | ---- | --- |
| title       | text(30) | 타이틀 |
| description | text(19) | 디스크립션 |
| imageUrl    | text(500) | 썸네일 이미지 주소 |

##### 3.2.1.2 templateItem 상세 결과
| 키       | -   | 타입 | 설명 |
| ------- | ---- | ------ | --- |
| list    |      | array  | 아이템 리스트 |
|  | title       | text(6)   | 타이틀 |
|  | description | text(23)   | 디스크립션 |
| summary |      | object | 아이템 요약 정보 |
|  | title       | text(6)   | 타이틀 |
|  | description | text(14)   | 디스크립션 |

##### 3.2.1.3 comments 상세 결과
| 키                 | -   | 타입 | 설명 |
| ----------------- | --- | ---- | --- |
| id                |  | Int  | 댓글 아이디 |
| content           |  | text | 댓글 내용 |
| userName          |  | text | 작성자 |
| createdAt         |  | text | 등록일 |
| status            |  | text | 댓글 상태(INQ:문의, APR:승인, REJ:반려, REP:답변) |
| attachment        |  | list | 첨부파일 |
|   | originalFileName | text | 업로드 당시 기존 파일명 |
|   | filePath         | text | 파일 다운로드 경로 |

##### 3.2.1.4 buttons 상세 결과
| 키 | 타입 | 설명 |
| -------- | ---- | --- |
| ordering | number | 버튼 노출 순서 |
| name     | text   | 버튼명 |
| linkType | text   | 버튼의 링크타입 (DS: 배송조회, WL: 웹링크, AL: 앱링크, BK: 봇키워드, MD: 메시지전달, BC: 상담톡전환, BT: 봇전환, AC: 채널 추가, P1: 이미지 보안 전송 플러그인, P2:개인정보이용 플러그인, P3: 원클릭 결제 플러그인, BF: 비즈니스폼) |
| linkTypeName | text | 버튼의 링크타입이름 (배송조회, 웹링크, 앱링크, 봇키워드, 메시지전달, 상담톡전환, 봇전환, 채널 추가, 이미지 보안 전송 플러그인, 개인정보이용 플러그인, 원클릭 결제 플러그인, 비즈니스폼) |
| linkMo   | text   | 모바일 웹링크주소 |
| linkPc   | text   | PC 웹링크주소 |
| linkIos  | text   | IOS 앱링크주소 |
| linkAnd  | text   | Android 앱링크주소 |
| pluginId | text   | 플러그인 ID |
| bizFormId | number   | 비즈니스폼 ID |


##### 3.2.1.5 quickReplies 상세 결과
| 키 | 타입 | 설명 |
| -------- | ---- | --- |
| name     | text | 바로연결명 |
| linkType | text | 바로연결의 링크타입 (WL: 웹링크, AL: 앱링크, BK: 봇키워드, MD: 메시지전달, BC: 상담톡전환, BT: 봇전환, BF: 비즈니스폼) |
| linkTypeName | text | 버튼의 링크타입이름 (웹링크, 앱링크, 봇키워드, 메시지전달, 상담톡전환, 봇전환, 비즈니스폼) |
| linkMo   | text | 모바일 웹링크주소 |
| linkPc   | text | PC 웹링크주소 |
| linkIos  | text | IOS 앱링크주소 |
| linkAnd  | text | Android 앱링크주소 |
| bizFormId | number | 비즈니스폼 ID |


### 3.3. 검수 요청/취소 및 승인 취소
등록된 템플릿을 검수 요청/검수 요청 취소를 할수있고, 승인후 발송되지 않은 대기(R)상태의 템플릿을 재검수를 위해 승인취소 할수 있습니다.

#### 3.3.1. 검수요청
- 등록된 템플릿을 검수 요청 합니다. 템플릿상태가 대기(R)이고 템플릿 검수상태가 등록(REG)인 경우에만 요청 가능합니다.
- 검수요청시 의견 또는 문의를 선택적으로 입력할수 있습니다.

##### [Request]
**POST** /api/v2/alimtalk/template/request

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키 | 타입 | 필수 | 설명 |
| ------------- | ---- | --- | --- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입 (G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y   | 템플릿 코드 |
| comment  | text(500) | N   | 의견 또는 문의 사항 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키 | 타입 | 설명 |
| ---- | ---- | ------- |
| code | text | 결과 코드 |

#### 3.3.2. 검수요청(파일첨부) - 지원 예정
- 등록된 템플릿을 검수 요청 합니다. 템플릿상태가 대기(R)이고 템플릿 검수상태가 등록(REG)인 경우에만 요청 가능합니다.
- 파일와 함께 문의또는 의견을 남길수 있습니다.
- 파일 형식은 png, jpg, jpeg, gif, pdf, hwp, doc, docx만 가능하며 개당 50MB까지 첨부할 수 있습니다.

##### [Request]
**POST** /api/v2/alimtalk/template/request_with_file

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키 | 타입 | 필수 | 설명 |
| ------------- | ---- | --- | --- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입 (G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y   | 템플릿 코드 |
| comment  | text(500) | Y   | 의견 또는 문의 사항 |
| attachment    | binary | N | 업로드 할 파일의 절대 경로 (최대 10개 업로드 가능) |


##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키 | 타입 | 설명 |
| ---- | ---- | ------- |
| code | text | 결과 코드 |


#### 3.3.3. 검수요청 취소
- 등록된 템플릿을 검수 요청 합니다. 템플릿상태가 대기(R)이고 템플릿 검수상태가 검수요청(REQ)인 경우에만 요청 가능합니다.

##### [Request]
**POST** /api/v2/alimtalk/template/cancel_request

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키 | 타입 | 필수 | 설명 |
| ------------- | ---- | --- | --- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입 (G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y   | 템플릿 코드 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키 | 타입 | 설명 |
| ---- | ---- | ------- |
| code | text | 결과 코드 |

#### 3.3.4. 승인 취소
- 승인된 템플릿이 대기(R) 상태 일때 승인 취소할 수 있습니다. 승인취소시 등록(REG)상태로 변경되며 재 검수 요청 가능합니다.

##### [Request]
**POST** /api/v2/alimtalk/template/cancel_approval

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키 | 타입 | 필수 | 설명 |
| ------------- | ---- | --- | --- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입 (G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y   | 템플릿 코드 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키 | 타입 | 설명 |
| ---- | ---- | ------- |
| code | text | 결과 코드 |

### 3.4. 수정
등록된 템플릿을 수정합니다. 템플릿상태가 대기(R)이고 템플릿 검수상태가 등록(REG) 또는 반려(REJ)인 경우에만 수정 가능합니다.


#### 3.4.1 템플릿 수정
##### [Request]
**POST** /api/v2/alimtalk/template/update

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키            | 타입 | 필수 | 설명 |
| ------------------ | -------   | --- | --------------------------------------- |
| senderKey          | text(40)      | Y   | 발신 프로필 키 |
| senderKeyType      | text(1)      | N   | 발신프로필타입 (G:그룹, S:기본(default)) |
| templateCode       | text(30)  | Y   | 템플릿 코드 |
| newSenderKey       | text(40)      | Y   | 발신 프로필 키 (senderKeyType이 G인경우발신 프로필그룹키) |
| newSenderKeyType   | text(1)      | N   | 발신 프로필 키 타입 (G:그룹 , S:발신프로필(default)) |
| newTemplateCode    | text(30)  | Y   | 템플릿 코드 |
| newTemplateName    | text(200) | Y   | 템플릿 이름 |
| newTemplateMessageType | text  | Y   | 템플릿 메시지 유형<br/>(BA: 기본형, EX: 부가 정보형, AD: 채널 추가형, MI: 복합형)<br/>- EX : newTemplateExtra 필드 필수<br/>- AD : 그룹 템플릿 사용 불가<br>- MI : newTemplateExtra 필드 필수, 그룹 템플릿 사용 불가 |
| newTemplateEmphasizeType | text| Y   | 템플릿 강조 유형<br/>(NONE: 선택안함, TEXT: 강조표기형, IMAGE: 이미지형, ITEM_LIST: 아이템리스트형)<br/>- TEXT: newTemplateTitle, newTemplateSubtitle 필드 필수<br/>- ITEM_LIST: newTemplateItem.list, (newTemplateImage(Name, Url), newTemplateHeader, newTemplateItemHighlight 필드 중 1개 이상 필수, newTemplateItem.summary 필드는 newTemplateItem.list  사용시에만 사용 가능 사용 가능 (2022-08-30 이후이며 그 전까지 list 필수)<br>-IMAGE: newTemplateImageName, newTemplateImageUrl 필드 필수 |
| newTemplateContent | text      | Y   | 템플릿 내용 |
| newTemplateExtra   | text      | N   | 부가 정보(템플릿 검수 가이드 참고) |
| newTemplateImageName | text    | N   | 템플릿 이미지 파일명 (템플릿 검수 가이드 참고, [업로드 API](https://github.com/lunasoft-org/kakao-bzm/blob/main/upload-api.md#22-알림톡-템플릿-등록용-이미지-업로드-요청) 참고) |
| newTemplateImageUrl | text     | N   | 템플릿 이미지 링크 (템플릿 검수 가이드 참고, [업로드 API](https://github.com/lunasoft-org/kakao-bzm/blob/main/upload-api.md#22-알림톡-템플릿-등록용-이미지-업로드-요청) 참고) |
| newTemplateTitle   | text      | N   | 템플릿 내용 중 강조 표기할 핵심 정보 (템플릿 검수 가이드 참고) |
| newTemplateSubtitle | text     | N   | 강조 표기 보조 문구 (템플릿 검수 가이드 참고) |
| newTemplateHeader  | text(16)  | N   | 헤더 (템플릿 검수 가이드 참고) |
| newTemplateItemHighlight | object | N | 아이템 하이라이트 (템플릿 검수 가이드 참고, [상세](#3111-templateItemHighlight-상세-요청-파라미터)) |
| newTemplateItem    | object    | N   | 아이템 리스트 (템플릿 검수 가이드 참고, [상세](#3112-templateItem-상세-요청-파라미터))
| newCategoryCode    | text      | N   | 템플릿 카테고리 코드  ([3.9.1. 템플릿 카테고리 전체 조회 API](#391-템플릿-카테고리-전체-조회) 참고) |
| securityFlag       | boolean   | N   | 보안 템플릿 여부,  OTP등 보안 메시지 일 경우 설정<br/>발신 당시의 메인 디바이스를 제외한 모든 디바이스<br/> (디폴트: 미설정)|
| buttons            | list      | N   | 버튼 정보 (최대 5개 등록 가능 (단, 바로연결과 함께 사용 시 2개로 제한됨), [상세](#3113-buttons-상세-요청-파라미터)) |
| quickReplies       | list      | N   | 바로연결 정보([상세](#3114-quickReplies-상세-요청-파라미터)) |

##### [Example]
```
curl -v -X POST -d 'senderKey={senderKey}' -d 'senderKeyType=S' -d 'templateCode=center_api_test_02' -d 'newSenderKey={senderKey}' -d 'newSenderKeyType=S' -d 'newTemplateCode=center_api_test_02' -d 'newTemplateName=상품발송안내02' -d 'newTemplateMessageType=BA' -d 'newTemplateEmphasizeType=NONE' -d 'newTemplateContent=#{홍길동}님 주문하신 #{가습기} 금 일 배송 출발 하였습니다.' -d 'buttons[0].ordering=1' -d 'buttons[0].linkType=DS' -d 'buttons[0].name=택배조회' -d 'buttons[1].ordering=2' -d 'buttons[1].linkType=WL' -d 'buttons[1].name=바로가기' -d 'buttons[1].linkMo=http://daum.net'  -d 'quickReplies[0].linkType=BK' -d 'quickReplies[0].name=봇키워드' -d 'quickReplies[1].linkType=WL' -d 'quickReplies[1].name=바로가기' -d 'quickReplies[1].linkMo=http://daum.net'  https://test-bizcenter.lunasoft.co.kr/api/v2/alimtalk/template/update
```

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키   | 타입     | 설명 |
| ---- | ------ | --- |
| code | text   | 결과 코드 |
| data | object | [템플릿 상세](#321-템플릿-상세-결과) 참고 |


### 3.5. 템플릿 사용 중지/해제
#### 3.5.1. 중지
등록된 템플릿을 중지 상태로 변경합니다. 템플릿상태가 대기(R) 또는 정상(A)이고 템플릿 검수상태가 승인(APR)인 경우에만 요청 가능합니다.

##### [Request]
**POST** /api/v2/alimtalk/template/stop

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키 | 타입 | 필수 | 설명 |
| ------------- | ---- | --- | --- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입 (G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y   | 템플릿 코드 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키 | 타입 | 설명 |
| ---- | ---- | ------- |
| code | text | 결과 코드 |


#### 3.5.2. 중지해제
등록된 템플릿을 정상 상태로 되돌립니다. 템플릿상태가 중지(S)이고 템플릿 검수상태가 승인(APR)인 경우에만 요청 가능합니다.

##### [Request]
**POST** /api/v2/alimtalk/template/reuse

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키 | 타입 | 필수 | 설명 |
| ------------- | ---- | --- | --- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입 (G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y   | 템플릿 코드 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키 | 타입 | 설명 |
| ---- | ---- | ------- |
| code | text | 결과 코드 |


### 3.6. 삭제
- 등록된 템플릿을 삭제 합니다.
- 템플릿상태가 대기(R)이고 템플릿 검수상태가 등록(REG) 또는 반려(REJ)인 경우에만 삭제 가능합니다.

##### [Request]
**POST** /api/v2/alimtalk/template/delete

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키 | 타입 | 필수 | 설명 |
| ------------- | ---- | --- | --- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입 (G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y   | 템플릿 코드 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키 | 타입 | 설명 |
| ---- | ---- | ------- |
| code | text | 결과 코드 |

### 3.7. 최근 변경 템플릿 조회
등록한 템플릿 중 변경된 건을 조회합니다. 변경된 템플릿은 [템플릿 조회](#32-조회) 를 통해 확인 가능합니다.

##### [Request]
**GET** /api/v3/alimtalk/template/last_modified

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키            | 타입     | 필수 | 설명 |
| ------------- | ------- | --- | --- |
| senderKey     | text(40)| N   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입 (G:그룹, S:기본(default)) |
| since         | text(14)| N   | 기준시간 yyyyMMddHHmmss (default는 요청한 시간부터 1시간전) |
| page          | number  | N   | 페이지 (default : 1) |
| count         | number  | N   | 한 페이지에 조회할 건수(default: 1000) |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키   | -    | 타입     | 설명 |
| ---- | ------- | ------ | --- |
| code |         | text   | 결과 코드 |
| hasNext |         | boolean   |  다음 페이지 여부 |
| list |         | array   | 변경된 템플릿 정보 |
|   |    senderKey     | text(40)   |  발신 프로필 키 |
|   |    senderKeyType     | text(1)   | 발신프로필타입 (G:그룹, S:기본(default)) |
|   |    templateCode     | text(30)   | 템플릿코드 |



### 3.8. 승인/반려 (테스트용)
#### 3.8.1. 승인 (테스트용)
등록된 템플릿을 승인 합니다.

##### [Request]
**POST** /api/v2/alimtalk/template/test_approve

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키            | 타입  | 필수 | 설명 |
| ------------- | ---- | --- | --- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입(G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y   | 템플릿 코드 |
| comment       | text | N   | 댓글 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키   | 타입   | 설명 |
| ---- | ---- | --- |
| code | text | 결과 코드 |

#### 3.8.2. 반려 (테스트용)
등록된 템플릿을 반려합니다.

##### [Request]
**POST** /api/v2/alimtalk/template/test_reject

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키            | 타입  | 필수 | 설명 |
| ------------- | ---- | --- | --- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입(G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y   | 템플릿 코드 |
| comment       | text | N   | 댓글 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키   | 타입   | 설명 |
| ---- | ---- | --- |
| code | text | 결과 코드 |


### 3.9. 템플릿 카테고리
#### 3.9.1. 템플릿 카테고리 전체 조회
템플릿 등록시 사용할 카테고리 목록 전체를 조회합니다.

##### [Request]
**GET** /api/v2/alimtalk/template/category/all

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Response]
| 키   |   | 타입  | 설명 |
| ---- | ---- | ---- | ------- |
| code |      | text | 결과 코드 |
| data |      | array | |
|      | code | text | 카테고리 코드 |
|      | name | text(15) | 카테고리 이름 |
|      | groupName | text(15) | 카테고리 그룹 이름 |
|      | inclusion | text(500) | 카테고리 적용 대상 템플릿 설명 |
|      | exclusion | text(500) | 카테고리 제외 대상 템플릿 설명 |

##### [Example]
```
{
    "code": "200",
    "data": [{
        "code": "001001",
        "name": "예시 카테고리",
        "groupName": "예시 카테고리 그룹",
        "inclusion": "이런 템플릿을 이 카테고리에 적용하세요",
        "exclusion": "이런 템플릿은 이 카테고리에 적용하지 마세요"
    },
    ...]
}
```


#### 3.9.2. 템플릿 카테고리 조회
카테고리 코드에 해당하는 특정 템플릿 카테고리를 조회합니다.

##### [Request]
**GET** /api/v2/alimtalk/template/category

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키           | 타입  | 필수 | 설명 |
| ------------ | ---- | --- | --- |
| categoryCode | text | Y   | 카테고리 코드 |

##### [Response]
| 키   |      | 타입  | 설명 |
| ---- | ---- | ---- | ------- |
| code |      | text | 결과 코드 |
| data |      | object | |
|      | code | text | 카테고리 코드 |
|      | name | text(15) | 카테고리 이름 |
|      | groupName | text(15) | 카테고리 그룹 이름 |
|      | inclusion | text(500) | 카테고리 적용 대상 템플릿 설명 |
|      | exclusion | text(500) | 카테고리 제외 대상 템플릿 설명 |

##### [Example]
```
{
    "code": "200",
    "data": {
        "code": "001001",
        "name": "예시 카테고리",
        "groupName": "예시 카테고리 그룹",
        "inclusion": "이런 템플릿을 이 카테고리에 적용하세요",
        "exclusion": "이런 템플릿은 이 카테고리에 적용하지 마세요"
    }
}
```

### 3.10. 템플릿 휴면 해제
장기간 미사용으로 휴면된 템플릿을 해제합니다. 해제후 30일간 사용하지 않는경우 재 휴면처리 됩니다.
##### [Request]
**POST** /api/v2/alimtalk/template/dormant/release

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키            | 타입  | 필수 | 설명 |
| ------------- | ---- | --- | ---------- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| senderKeyType | text(1) | N   | 발신프로필타입 (G:그룹, S:기본(default)) |
| templateCode  | text(30) | Y   | 템플릿 코드 |

##### [Response]
| 키   | 타입   | 설명 |
| ---- | ---- | --- |
| code | text | 결과 코드 |


### 3.11. 기등록된 템플릿 (타입 : BA, EX)을 "채널 추가 버튼" 및 "채널 추가 안내 문구"가 포함된 템플릿으로 전환
- 템플릿 전환이 되면 BA는 AD로, EX는 MI 타입으로 변경됨
- "채널 추가 버튼" 무조건 맨 처음으로 추가되며, 템플릿에 버튼이 이미 5개인 경우는 전환 실패
- 채널 추가 안내 문구 (36자)가 포함되므로 기존 템플릿이 이미 964를 초과하는 경우 전환 실패
- 휴면 등 비정상적인 템플릿에 대해서는 전환 실패

##### [Request]
**POST** /api/v2/alimtalk/template/convertAddCh

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키            | 타입  | 필수 | 설명 |
| ------------- | ---- | --- | ---------- |
| senderKey     | text(40) | Y   | 발신 프로필 키 |
| templateCode  | text(30) | Y   | 템플릿 코드 |

##### [Response]
| 키   | 타입   | 설명 |
| ---- | ---- | --- |
| code | text | 결과 코드 |


## 4. 발신 프로필 그룹 관리
- 발신프로필 여러개를 그룹으로 관리 가능합니다. 동일 그룹에 속한 발신프로필은 발신 프로필 그룹으로 등록된 템플릿을 공유하여 사용 할 수 있습니다.
- 발신 프로필 그룹 신규 등록은 루나소프트 운영 담당자에게 요청하여 처리 가능합니다. (별도의 API를 지원하지 않습니다.)

### 4.1.조회
발신 프로필 그룹 리스트를 조회합니다.

##### [Request]
**GET** /api/v1/group

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요. data 하위의 값은 list로 제공됩니다.

| 키   | -         | 타입  | 설명 |
| ---- | --------- | ---- | --- |
| code |           | text | 결과 코드 |
| data | groupKey  | text(100) | 발신 프로필 그룹키 |
|      | name      | text(40) | 이름 |
|      | createdAt | text(19) | 등록일 |

##### [Example]
```
{
    "code": "200",
    "data": [{
        "groupKey": "be92e7eaeb00e15957467c5f5c94274b7747611x",
        "name": "카카오 서비스 그룹",
        "createdAt": "2016-01-13 15:58:20"
    },
             …생략…
            {
        "groupKey": "d1616ab2e6e3cb44fc398cb25c22476e8f2393e1",
        "name": "비즈메시지 그룹",
        "createdAt": "2016-01-13 15:57:42"
    }]
}
```


### 4.2. 그룹에 포함된 발신 프로필 조회
발신 프로필 그룹에 포함된 발신 프로필 목록을 조회합니다.

##### [Request]
**GET** /api/v3/group/sender

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키       | 타입   | 필수  | 설명 |
| -------- | ---- | ---- | --- |
| groupKey | text(100) | Y    | 발신 프로필 그룹키 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요. data 하위의 값은 list로 제공됩니다.

| 키   | -             | 타입      | 설명 |
| ---- | ------------- | ------- | --- |
| code |               | text    | 결과 코드 |
| data | senderKey     | text(40)    | 발신프로필키 |
|      | uuid          | text(20)    | 카카오톡 채널 |
|      | name          | text(20)    | 카카오톡 채널 프로필명 |
|      | status        | text(1)    | 상태 (A:정상) |
|      | block         | boolean | 발신프로필 차단 여부 (true: 차단, false: 해제) |
|      | dormant       | boolean | 발신프로필 휴면 여부 |
|      | profileStatus | text(1)   | 카카오톡 채널 상태 (A:activated, C:deactivated, B:block, E:deleting, D:deleted) |
|      | createdAt     | text(19)    | 등록일 |
|      | modifiedAt    | text(19)    | 최종수정일 |
|      | categoryCode  | text(11)    | 카테고리 코드 |


### 4.3. 그룹에 발신 프로필 추가
발신 프로필 그룹에 발신 프로필을 추가 합니다.

##### [Request]
**POST** /api/v1/group/sender/add

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키        | 타입  | 필수  | 설명 |
| --------- | ---- | --- | --- |
| groupKey  | text(100) | Y   | 발신 프로필 그룹키 |
| senderKey | text(40) | Y   | 발신 프로필 키 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키   | 타입   | 설명 |
| ---- | ---- | --- |
| code | text | 결과 코드 |


### 4.4. 그룹에 발신 프로필 삭제
발신 프로필 그룹에 발신 프로필을 삭제 합니다.

##### [Request]
**POST** /api/v1/group/sender/remove

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키        | 타입  | 필수  | 설명 |
| --------- | ---- | --- | --- |
| groupKey  | text(100) | Y   | 발신 프로필 그룹키 |
| senderKey | text(40) | Y   | 발신 프로필 키 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키   | 타입   | 설명 |
| ---- | ---- | --- |
| code | text | 결과 코드 |



## 5. (개발 서버) 테스트 사용자 인증
- 개발 서버에서 발송 테스트를 하기 위해 일회성 사용자 인증을 받을 수 있습니다.
- 인증된 사용자에 대해서 그날 하루동안 (23:59:59 까지) 테스트 발송을 할 수 있습니다.

### 5.1. Host
> ####**[개발서버]**  https://test-kakao-bizcenter.lunasoft.co.kr/

### 5.2 인증 토큰 요청
일회성 사용자 등록을 위한 토큰을 요청합니다.

##### [Request]
**GET** /api/v1/testUser/token

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키          | 타입  | 필수 | 설명 |
| ----------- | ---- | --- | --- |
| phoneNumber | text | Y   | 전화번호 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키         | 타입  | 설명 |
| ---------- | ---- | --- |
| code       | text | 결과 코드 |
| expired_at | text | 토큰 만료 시간 (yyyy-MM-dd HH:mm:ss, KST기준)|
| message    | text | 에러 메시지 |

### 5.3 테스트 사용자 인증
- 카톡으로 발송한 토큰을 이용해 일회성 사용자 인증을 시도합니다.

##### [Request]
**POST** /api/v1/testUser/certify

##### [Header]
| 키          | 타입  | 필수  | 설명 |
| ----------- | ---- | --- | --- |
| agentKey    | text | Y   | `사전에 루나소프트 운영 담당자를 통해 전달 받은 AgentKey` |

##### [Parameter]
| 키          | 타입  | 필수 | 설명 |
| ----------- | ---- | --- | --- |
| phoneNumber | text | Y   | 전화번호 |
| token       | text | Y   | 카톡으로 수신한 인증 번호 |

##### [Response]
응답 바디는 JSON객체로 아래 값을 참고 해주세요.

| 키         | 타입  | 설명 |
| ---------- | ---- | --- |
| code       | text | 결과 코드 |
| expired_at | text | 토큰 만료 시간 (yyyy-MM-dd HH:mm:ss, KST기준)|
| message    | text | 에러 메시지 |
