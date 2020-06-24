### ES Mapping
* 의미
    * ES의 문서 스키마(6.x부터 1 index - single mapping만 가능)
        1. Dynamic Mapping - 스키마가 없을 때 인입되는 문서를 보고 자동으로 매핑을 만듦
        ```json
        PUT {인덱스 이름}/_doc/{인덱스 ID 번호} 
        { 
            "count": 5 
        }
        ```
        | Value Type | Mapping Field / Description in Dynamic mapping |
        |:-----------|:-----------------------------------------------|
        |null|No field is added|
        |true of false|boolean field|
        |floating point number|float field
        |integer|long field|
        |object|object field|
        |array|string or object field|
        |date string|double or long field|
        |text string|text field with a keyword sub-field|
        2. Static Mapping - 사용자가 정의한 스키마를 기준으로 매핑
* 특징
    * `mappings` 필드를 정의하면 인덱스 정의하듯이 입력해도 매핑으로 인식
    * 매핑 필드 추가는 가능하나 삭제나 변경은 불가능
* 응용
    * `template`을 이용하여 인덱스가 생성될 때 사용자 정의된 세팅이나 매핑을 자동으로 적용 가능
        1. 인덱스 패턴, 인덱스 세팅, 인덱스 매핑 관련 사항 정의
        2. 인덱스가 생성될 때 패턴이 매칭되는 인덱스는 해당 정의
        3. order가 높은 번호가 낮은 번호를 override하여 merging
        ```json
        PUT _template/{템플릿 이름}
        {
            "index_patterns": ["{인덱스 이름 패턴}", "{인덱스 이름 패턴}"], "order" : 0,
            "settings": 
            {
                "index.number_of_shards": {샤드 개수}
            }
        }
        ```
        4. 템플릿 삭제는 DELETE 메소드 이용
        `DELETE _template/{템플릿 이름}`


### 매핑과 필드
* 매핑에 쓰이는 주요 파라미터
    * analyzer: 색인과 검색 시 지정한 분석기로 형태소 분석을 수행하는 파라미터로 `text 타입`의 필드는 analyzer 파라미터를 필수적으로 사용해야함
    * boost: 필드에 가중치를 부여하는 파라미터로 `유사도 점수(_score)`에 영향을 크게 끼침
    * copy_to: 매핑 파라미터를 추가한 필드의 값을 지정한 필드로 복사
    * doc_vlaue: ES에서 기본으로 사용하는 캐시로 text 타입를 제외한 모든 타입이 사용
    * dynamic: 매핑에 필드를 추가할 때 동적으로 추가가 가능하게 할지 안할지를 결정하는 파라미터
    * format: 날짜/시간 등에서 미리 구성된 포멧을 이용할 때 설정하는 파라미터
    * ignore_above: 필드에 저장되는 문자열이 지정한 크기를 넘어서면 빈값으로 색인하게하는 파라미터
    * fields: `다중 필드`를 설정할 수 있는 옵션으로 필드 안에 또 다른 필드의 정보 추가가 가능
    * properties: `오브젝트`나 `중첩` 타입의 스키마를 정의할 때 반드시 사용하는 옵션으로 필드의 타입을 매핑
* 메타 필드
    * _index: 해당 문서의 인덱스 이름을 가진 필드
    * _type: 해당 문서가 속한 매핑의 타입에 대한 정보를 가진 필드
    * _id: 문서를 식별하는 필드
    * _source: `문서의 원본 데이터`로 JSON 형태의 문서를 검색 결과로 표시할 때 사용하는 필드
    * _all: 색인에 사용된 모든 필드의 정보를 가진 필드로 전체 필드에 대한 검색 시에 사용했지만 6.x이상에서는 폐기된 필드
    * _routing: 특정 문서를 `특정 샤드`에 저장하기 위해 사용자가 지정하는 필드
*  필드의 데이터 타입
    * keyword: `분석기를 거치지 않고` 원문 그대로 색인하는 특성을 가진 정형화된 내용을 담는 타입 - 집계나 정렬이 가능
    * text: `분석기를 거치는` 데이터로 전문 검색이 가능하게하는 필드 - 집계나 정렬이 불가능
    * long: 최댓값과 최솟값을 갖는 64비트 정수
    * integer: 최댓값과 최솟값을 갖는 32비트 정수
    * short: 최댓값과 최솟값을 갖는 16비트 정수
    * byte: 최댓값과 최솟값을 갖는 8비트 정수
    * double: 64비트 부동 소수점을 갖는 수
    * float: 32비트 부동 소수점을 갖는 수
    * range: 범위가 이는 데이터를 저장할 때 사용하는 타입
    * date: JSON 포맷에서 문자열로 처리되는 타입으로 `format`을 이용해 날짜를 표현하는 타입
    * object: JSON 포맷에서 문서는 내부 객체를 포함할 수 있는데 문서를 데이터로 가지는 필드
    * nested: object 객체 배열을 독립적으로 색인하고 질의하는 형태의 데이터 타입