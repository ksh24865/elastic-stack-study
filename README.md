Elastic-stack-study
====

## 1. ELK Stack
 * 사용자가 서버나 데이터베이스로부터 원하는 데이터 및 로그를 실시간으로 수집하고 검색, 분석하여 시각화 시키는 오픈소스 서비스
 * Elastic Search + Logstash + Kibana + Filebeat
 

## 2. ELK Stack의 구성요소
### 2.1 Elastic search
* 기능
   * 확장성이 매우 좋은 오픈소스 검색엔진
   * Lucene 기반의 데이터 검색 및 분석 엔진이다.
   * 대량의 데이터를 신속하고 실시간으로 저장, 검색 및 분석할 수 있다.
   * 데이터를 중심부에 저장하여 예상되는 항목을 검색하고 예상치 못한 항목을 밝혀낼 수 있다.
   * 정형, 비정형, 위치정보, 메트릭 등 원하는 방법으로 다양한 유형의 검색을 수행하고 결합할 수 있다.
   * 표준 RESTful API와 JSON을 사용한다.
   
* 개념정리
1. full text 검색    
    - 관계형 DB에서는 Text 열을 한 줄씩 찾아 내려가면서 키워드가 있으면 가져오고 없으면 넘어감. 
        - row 안의 내용을 모두 읽어야 하기 때문에 기본적으로 속도가 느림.
        
    - 역 인덱스(Inverted Index)라는 구조로 데이터를 저장한다. (해당 키워드가 어떤 doc에 있다고 저장한다.)
        - 책의 맨 뒤 주요 키워드에 대한 내용이 몇 페이지에 있는지 볼 수 있는 찾아보기 페이지와 유사.
        - 추출된 각 키워드를 텀(term) 이라고 부르며, 이렇게 역 인덱스가 있으면 term을 포함하고 있는 도큐먼트들의 id를 바로 얻어올 수 있음.
        - 데이터 추가 시 읽어야 할 행이 늘어나는 것이 아니라 역 인덱스가 가리키는 id의 배열값이 추가되는 것 뿐이기 때문에 큰 속도의 저하 없이 빠른 속도로 검색이 가능
         <img width="540" alt="image (7)" src="https://user-images.githubusercontent.com/55729930/102207801-eee65b00-3f11-11eb-8e27-52ce311cd488.png">
         
    - 텍스트 분석(Text Analysis)
        - 문자열 필드가 저장될 때 데이터에서 검색어 토큰을 저장하기 위한 여러 단계의 처리 과정
    * 애널라이저(Analyzer) 
        - 텍스트분석을 처리하는 기능
        - 0-3개의 캐릭터 필터(Character Filter)와 1개의 토크나이저(Tokenizer), 그리고 0-n개의 토큰 필터(Token Filter)로 이루어짐.        
        * _analyze API
            * 분석된 문장을 _analyze API를 이용해서 확인
            * 토크나이저는 `tokenizer`, 토큰 필터는 `filter` 항목의 값으로 입력
            * 토크나이저는 하나만 적용되기 때문에 바로 입력하고, 토큰필터는 여러개를 적용할 수 있기 때문에 [ ] 안에 배열 형식으로 입력
        * Term 쿼리
            * `match` 쿼리와 문법은 유사하지만 `term` 쿼리는 입력한 검색어는 애널라이저를 적용하지 않고 입력된 검색어 그대로 일치하는 텀을 찾음.
        * 사용자 정의 애널라이저 - Custom Analyzer
            * _analyze API로 애널라이저, 토크나이저, 토큰필터들의 테스트가 가능하지만, 실제로 인덱스에 저장되는 데이터의 처리에 대한 설정은 애널라이저만 적용할 수 있다.
            * 인덱스 매핑에 애널라이저를 적용 할 때 토크나이저, 토큰필터 등을 조합하여 만든 사용자 정의 애널라이저(Custom Analyzer)를 주로 사용
            * 인덱스 settings 의 `"index" : { "analysis" :`  부분에 정의
            * `GET` 또는 `POST <인덱스명>/_analyze` 명령으로 사용이 가능
            * PUT my_index3 { "settings": { "index": { "analysis": { "analyzer": { "my_custom_analyzer": { "type": "custom", "tokenizer": "whitespace", "filter": [ "lowercase", "stop", "snowball" ] } } } } } }` ( my_index3 안에 whitespace 토큰크나이저 그리고 lowercase, stop, snowball 토큰필터를 사용하는 my_custom_analyzer 라는 이름의 애널라이저를 추가 )
        * 텀 벡터 - _termvectors API
            * 색인된 도큐먼트의 역 인덱스의 내용을 확인할 때는 도큐먼트 별로 _termvectors API를이용해서 확인
            * `GET <인덱스>/_termvectors/<도큐먼트id>?fields=<필드명>` 형식으로 사용
            * EX) `GET my_index3/_termvectors/1?fields=message` ( my_index3/_doc/1 도큐먼트의 message 필드의 termvectors 확인 )
    * 캐릭터 필터 - Character Filter
        * 텍스트 데이터가 입력되면 가장 먼저 필요에 따라 전체 문장에서 특정 문자를 대치하거나 제거하는데, 이 과정을 담당하는 기능
        * 캐릭터 필터는 텍스트 분석 중 가장 먼저 처리되는 과정으로 색인된 텍스트가 토크나이저에 의해 텀으로 분리되기 전에 전체 문장에 대해 적용되는 일종의 전처리 도구
        * HTML Strip
            * 입력된 텍스트가 HTML 인 경우 HTML 태그들을 제거하여 일반 텍스트로 만든다.
            * EX) `POST _analyze { "tokenizer": "keyword", "char_filter": [ "html_strip" ], "text": "<p>I&apos;m so <b>happy</b>!</p>" }` ( `<p>I&apos;m so <b>happy</b>!</p>`를 `I'm so happy!`로 변경
        * Mapping
            * 지정한 단어를 다른 단어로 치환
            * 특수문자 등을 포함하는 검색 기능을 구현하려는 경우 반드시 적용
            * `+` 기호는 _plus_ 로, `-` 기호는 _minus_ 로 치환
            ```
            PUT coding
            {
              "settings": {
                "analysis": {
                  "analyzer": {
                    "coding_analyzer": {
                      "char_filter": [
                        "cpp_char_filter"
                      ],
                      "tokenizer": "whitespace",
                      "filter": [ "lowercase", "stop", "snowball" ]
                    }
                  },
                  "char_filter": {
                    "cpp_char_filter": {
                      "type": "mapping",
                      "mappings": [ "+ => _plus_", "- => _minus_" ]
                    }
                  }
                }
              },
              "mappings": {
                "properties": {
                  "language": {
                    "type": "text",
                    "analyzer": "coding_analyzer"
                  }
                }
              }
            }
            ```
        * Pattern Replace
            * 정규식(Regular Expression)을 이용해서 좀더 복잡한 패턴들을 치환할 수 있는 캐릭터 필터
            * 카멜 표기법(camelCase)으로 된 단어를 대문자가 시작하는 단위 마다 공백을 삽입하여 세부 단어별로 토크나이징 될 수 있도록 camel 인덱스에 camel_analyzer 라는 애널라이저를 생성
            ```
            PUT camel
            {
              "settings": {
                "analysis": {
                  "analyzer": {
                    "camel_analyzer": {
                      "char_filter": [
                        "camel_filter"
                      ],
                      "tokenizer": "standard",
                      "filter": [
                        "lowercase"
                      ]
                    }
                  },
                  "char_filter": {
                    "camel_filter": {
                      "type": "pattern_replace",
                      "pattern": "(?<=\\p{Lower})(?=\\p{Upper})",
                      "replacement": " "
                    }
                  }
                }
              }
            }
            ```
            * camel_analyzer 애널라이저로 문장 분석
            ```
            GET camel/_analyze
            {
              "analyzer": "camel_analyzer",
              "text": [
                "public void FooBazBar()"
              ]
            }
            ```
        
    - 토크나이저
        - 문장에 속한 단어들을 텀 단위로 하나씩 분리 해 내는 처리 과정을 거치는데, 이 과정을 담당하는 기능
        - 토크나이저는 반드시 1개만 적용이 가능.
    - 토큰 필터
        - 분리된 텀 들을 하나씩 가공하는 과정을 담당
        - ex) lowercase : 토큰 필터를 이용해서 대문자를 모두 소문자로 바꿔 대소문자 구별 없이 검색이 가능
    - 불용어(stopword)
        - 검색어로서의 가치가 없는 단어 ( a, an, are, at, i )
    - snowball 토큰 필터
        - 문법상 변형된 단어를 일반적으로 검색에 쓰이는 기본 형태로 변환하여 검색이 가능하게 함
        - ex) jumps와 jumping은 모두 jump로 변경 후 텀 병합.
    - synonym 토큰 필터
        - 동의어를 하나의 단어로 텀 병합.
        - ex) quick 텀에 동의어로 fast를 지정하면 fast 로 검색했을 때도 같은 의미인 quick 을 포함하는 도큐먼트가 검색
    * 한글 형태소 분석기
        * 한글은 형태의 변형이 매우 복잡한 언어이다. 특히 복합어, 합성어 등이 많아 하나의 단어도 여러 어간으로 분리해야 하는 경우가 많아 한글을 형태소 분석을 하려면 반드시 한글 형태소 사전이 필요하다.
        * 노리 (nori), 아리랑 (arirang), 은전한닢 (seunjeon)등이 있다
    
2. Near Realtime
   * 거의 실시간 검색 플랫폼이라는 특징을 가지고 있다.
   
3. Cluster
   * 전체 데이터를 함께 보유하고 모든 노드에서 연합 인덱싱 및 검색 기능을 제공하는 하나 이상의 노드모음
   - 유일한 이름(unique name)으로 판별(identified) (기본값: 'elasticsearch')
   - 사용자는 클러스터를 대상으로 데이터를 저장하거나 검색 요청
   - Elasticsearch는 대용량 데이터의 증가에 따른 스케일 아웃과 데이터 무결성을 유지하기 위한 클러스터링을 지원, 클러스터를 기본으로 동작을 하며 1개의 노드만 있어도 클러스터로 구성
   - 여러 노드가 하나의 클러스터로 묶이기 위해서는 클러스터명 cluster.name 설정이 묶여질 노드들 모두 동일해야 한다.
   - 클러스터 구성
        - 여러 서버에 하나의 클러스터로 실행
            - 클라이언트와의 통신을 위한 http 포트, 노드 간의 데이터 교환을 위한 tcp 포트 존재.
            - 일반적으로 1개의 물리 서버마다 하나의 노드를 실행.
            - ex) 3개의 서버에서 각 1개의 노드를 실행
            
                <img width="320" alt="image (1)" src="https://user-images.githubusercontent.com/55729930/102191738-28609b80-3efd-11eb-9c9d-f8d55e634639.png">
                
            - ex) 한 서버에서 두개의 노드 실행, 또 다른 서버에서 한개 노드 실행
            
                <img width="320" alt="image (2)" src="https://user-images.githubusercontent.com/55729930/102191989-82f9f780-3efd-11eb-8bca-fadec5aba00a.png">
        - 하나의 서버에서 여러 클러스터 실행 
            - ex) 하나의 물리 서버에서 서로 다른 두 개의 클러스터 실행
                
                <img width="320" alt="image (3)" src="https://user-images.githubusercontent.com/55729930/102192754-893ca380-3efe-11eb-80f2-de3d0e2c4394.png">
                
                - node-1과 node-2는 하나의 클러스터로 묶여있기 때문에 데이터 교환이 가능. node-3 은 클러스터가 다르므로 나머지 노드들과 데이터 교환이 불가.
                - node-2 는 node-1이 마스터로 있는 클러스터 es-cluster-2에 묶인 것
                - node-3 스스로 es-cluster-2 클러스터의 마스터 노드
        - 디스커버리 (Discovery)
            - 노드가 처음 실행 될 때 같은 서버, 또는 `discovery.seed_hosts: [ ]` 에 설정된 네트워크 상의 다른 노드들을 찾아 하나의 클러스터로 바인딩 하는 과정
            - discovery.seed_hosts 설정에 있는 주소 순서대로 노드가 있는지 여부를 확인
                - 노드가 존재하는 경우 > cluster.name 확인
                    - 일치하는 경우 > 같은 클러스터로 바인딩 > 종료
                    - 일치하지 않는 경우 > 1로 돌아가서 다음 주소 확인 반복
            - 주소가 끝날 때 까지 노드를 찾지 못한 경우
                    - 스스로 새로운 클러스터 시작
            - 디스커버리 과정
            
                <img width="320" alt="image (4)" src="https://user-images.githubusercontent.com/55729930/102193919-04528980-3f00-11eb-87e1-03eb0deffed2.png">
4. Node
   - 클러스터의 일부이며 데이터를 저장하고 클러스터의 인덱싱 및 검색 기능에 참여하는 단일 서버
   - 노드에 할당되는 임의 UUID인 이름으로 식별
   - 특정 클러스터를 클러스터 이름으로 결합하도록 노드를 구성 할 수 있다.
   - 역할에 따라 마스터노드와 데이터, ingest, client 노드로 사용
   - 노드 관련 속성
        - node.master : 마스터 기능 활성화 여부
        - node.data : 데이터 기능 활성화 여부
        - node.ingest : Ingest 기능 활성화 여부
        - search.remote.connect : 외부 클러스터 접속 가능 여부
    - 노드 모드
        - 위의 속성들을 적절히 조합해서 특정 노드 모드로 운영가능
            - elasticsearch.yml 파일에 노드 관련 속성이 제공
        - Single Node mode
            - node.master: true / node.data: true / node.ingest: true / search.remote.connect: true
            - 모든 기능을 수행하는 모드. 기본 설정으로 지정돼 있기 때문에 elasticsearch.yml 파일에 아무런 설정을 하지 않는다면 기본적으로 싱글모드로 동작
        - Master Node mode
            - node.master: true / node.data: true / node.ingest: true / search.remote.connect: true
            - 클러스터의 제어를 담당하는 모드. 마스터 노드는 클러스터 전체를 관장하는 마스터 역할을 수행
                - 인덱스 생성/변경/삭제 등의 역할을 담당
                - 분산코디네이터 역할을 담당하여 클러스터를 구성하는 노드의 상태를 주기적으로 점검하여 장애를 대비
                - 마스터 노드는 클러스터에 다수(홀수개) 존재
                - 장애가 발생할 경우에도 후보 마스터 노드가 역할을 위임받아 안정적으로 클러스터 운영 유지
        - Data Node mode
            - node.master: false / node.data: true / node.ingest: false / search.remote.connect: false
            - 클러스터의 데이터를 보관하고 데이터의 CRUD, 검색, 집계 등 데이터 관련 작업을 담당하는 모드    *CRUD = Create(생성), Read(읽기), Update(갱신), Delete(삭제)
            - 노드가 데이터 모드로 동작하면 내부에 색인된 데이터가 저장. 마스터 노드와는 달리 대용량의 저장소를 필수적으로 갖춰야한다.
            - CRUD 작업과 검색, 집계와 같은 리소스를 제법 잡아먹는 역할도 수행하기 때문에 디스크만이 아닌 전체적인 스펙을 갖춘 서버로 운영
            - 대용량 클러스터 환경에서 마스터 노드와 데이터 노드의 분리는 필수.
        - Ingest Node mode
            - node.master: false / node.data: false / node.ingest: true / search.remote.connect: false
            - 다양한 형태의 데이터를 색인할 때 데이터의 전처리를 담당하는 모드
            - 엘라스틱서치에서 데이터를 색인하려면 인덱스라는(RDB Schema) 틀을 생성해야한다. 비정형 데이터를 다루는 저장소로 볼 수 있지만 일정한 형태의 인덱스를 생성해주어야한다.
            - 인덱스에는 여러 포맷의 데이터 타입 필드가 존재
            - 데이터를 색인할때 간단한 포맷 변경이나 유효성 검증 같은 전처리가 필요할 때 해당 모드를 이용
        - Coordination Node mode
            - node.master: false / node.data: false / node.ingest: false / search.remote.connect: false
            - 사용자 요청을 받아 처리하는 코디네이터 모드
            - 엘라스틱서치의 모든 노드는 기본적으로 코디네이션 모드 노드이다. 즉, 모든 노드가 사용자의 요청을 받아 처리할 수 있다는 뜻.
            - 다른 노드들에게 요청을 분산하고 결과값을 취합하는 코디네이션 노드를 별도로 구축한다면 안정적인 클러스터 운영이 가능
        
        <img width="320" alt="image (3)" src="https://user-images.githubusercontent.com/55729930/102196611-7b3d5180-3f03-11eb-91c9-b709445ee78d.png">
        
    - Split Brain
        - 네트워크 단절로 마스터 후보 노드가 분리되면 각자가 서로 다른 클러스터로 구성되어 계속 동작, 네트워크가 복구 되고 하나의 클러스터로 다시 합쳐졌을 때 데이터 정합성에 문제가 생기고 데이터 무결성 손실되는 현상.
        - 마스터 후보 노드를 하나만 놓게 되면 그 마스터 노드가 유실되었을 때 클러스터 전체가 작동을 정지 할 위험
        - 마스터 후보 노드들은 3개 이상의 홀수 개를 놓는 것을 권장
        <img width="500" alt="image (3)" src="https://user-images.githubusercontent.com/55729930/102223064-91f59f80-3f27-11eb-96d6-a1df8eafb709.png">
        
5. Index
   - 다소 유사한 특성을 갖는 Document들의 집합
   - 단일 클러스터에서 원하는만큼의 인덱스를 정의 할 수 있다.
   - RDB의 데이터베이스와 비슷한 개념
   - Index 내 다수의 Document저장
   - Indexing
        - 데이터가 검색될 수 있는 구조로 변경하기 위해 원본 문서를 검색어 토큰들으로 변환하여 저장하는 일련의 과정
    - Search
        - 인덱스에 들어있는 검색어 토큰들을 포함하고 있는 문서를 찾아가는 과정
    - Query
        - 사용자가 원하는 문서를 찾거나 집계 결과를 출력하기 위해 검색 시 입력하는 검색어 또는 검색 조건
    
    <img width="480" alt="image (4)" src="https://user-images.githubusercontent.com/55729930/102187425-52af5a80-3ef7-11eb-9578-2f17a8d99397.png">
    
6. Type
   - Index 내에서 하나 이상의 Type을 정의 할 수 있다.
   - ElasticSearch 7.x 버전부터는 _ doc로 고정
      
7. Document
   - JSON(Java Script Object Notatoin) 형태의 실제 의미있는 데이터를 가진 Elasticsearch 기본저장단위
   - Index를 생성
   - 고유한 문서 ID 보유 (문서 ID는 랜덤&수동 할당되며 문서 데이터를 찾아갈 때 사용)
   
8. Shards
   - Index는 잠재적으로 단일 노드의 하드웨어 제한을 초과 할 수 있는 많은 양의 데이터를 저장 할 수 있다. 하지만 단일 노드의 디스크가 맞지 않거나 단일 노드의 검색 요청만 처리하기에는 너무 느릴 수 있기 때문에 shards를 이용하여 Index를 여러 조각으로 나눌 수 있다. 
   - 노드에 분산되어 저장
   - 수평적으로 콘텐츠 볼륨을 split/scale 가능
   - 여러 노드에서 잠재적으로 분산을 통해 작업을 분산 및 병렬 처리를 할 수 있으므로 성능/처리량이 향상
   - 원본샤드(Primary Shard)와 복제본샤드(Replica Shard)로나뉨
   
    <img width="320" alt="image (3)" src="https://user-images.githubusercontent.com/55729930/102196126-d15dc500-3f02-11eb-8c0a-327677ae3929.png">
        
    - 단일 노드의 경우 프라이머리 샤드만 존재. Elasticsearch 는 아무리 작은 클러스터라도 데이터 가용성과 무결성을 위해 최소 3개의 노드로 구성 할 것을 권장.
    - 클러스터가 Node에 있던 프라이머리 샤드들을 유실하더라도 다른 노드들에 레플리카 샤드가 남아있으면 전체 데이터는 유실이 없이 사용이 가능.
    - 클러스터는 먼저 유실된 노드의 복구 대기. 타임아웃이 지나 유실된 노드가 복구되지 않는다고 판단되면 레플리카 샤드들의 복제를 시작.
        - 프라이머리 샤드, 레플리카 샤드의 개수는 유지됨.
    - 샤드의 개수는 인덱스를 처음 생성시 지정가능. 프라이머리 샤드 수는 인덱스를 처음 생성할 때 지정, 인덱스를 재색인 하지 않는 이상 변경불가. 복제본의 개수는 추후 변경이 가능.
       
8. Replication
   - 장애가 발생할 경우 복제본 샤드를 원본 샤드로 승격시켜 사용하기 때문에 복제본 샤드는 복사된 원본/기본 샤드와 동일한 노드에 할당되지 않는다.
   - 모든 복제본에서 검색을 병렬로 실행할 수 있기 때문에 검색 볼륨/처리량을 수평 확장 할 수 있다.
   - 기본적으로 각 인덱스는 4개의 기본 샤드와 1개의 복제본이 할당
9. Es와 RDB 비교

|RDB|ES|
|------|-----|
|Database|Index|
|Table|Type|
|Row|Document|
|Column|Field|
|index|Analyze|
|Primary ket|_id|
|Schema|Mapping|
|Physical partition|Shard|
|Logical partition|Route|
|Relational|Parent/Child,Nested|
|SQL|Query DSL|

10. RESTFul API
    - Elasticsearch는 Rest API를 기본으로 지원하며 모든 데이터 조회, 입력, 삭제를 http 프로토콜을 통해 Rest API로 처리한다.
    * Elasticsearch에서는 단일 도큐먼트별로 고유한 URL을 갖는다.
        * `http://<호스트>:<포트>/<인덱스>/_doc/<도큐먼트 id>`
    * CRUD API
        * 입력 (PUT)
            * EX) `PUT my_index/_doc/1 {"name":"Seongho Kim", "message":"Hi Elasticsearch"}`
            * 처음 도큐먼트를 입력시 "result" : "created" 로 표시, 동일한 URL에 다른 내용의 도큐먼트를 다시 입력시 덮어씌운 후 "result" : "updated" 표시
            * 실수로 도큐먼트가 덮어씌워지는 것을 방지하기 위해 _doc 대신 _create 를 사용
        * 조회 (GET)
            * EX) `GET my_index/_doc/1`
            * 해당 URL의 도큐먼트의 내용을 가져온다.
            * 문서의 내용은 _source 항목에 나타남
        * 삭제 (DELETE)
            * EX) `DELETE my_index/_doc/1`
        * 수정 (POST)
            * 데이터 입력 및 수정 시 사용
            * EX) `POST my_index/_doc {"name":"Seongho Kim", "message":"Hi Elasticsearch"}`
            * `POST <인덱스>/_doc` 까지만 입력하게 되면 자동으로 임의의 도큐먼트id 가 생성, PUT으로는 자동생성 불가능
            * `POST <인덱스>/_update/<도큐먼트 id>` 명령을 이용해 원하는 필드의 내용만 업데이트 가능
                * EX) `POST my_index/_update/1 {"doc": { "message":"Bye Elasticsearch" } }`
                * 수정 후 해당 도큐먼트를 GET 해보면 `"_version" : 2`로 변경
    * 벌크 API
        * 여러 명령을 배치로 수행하기 위해서 _bulk API의 사용
        * index, create, update, delete의 동작이 가능하며 delete를 제외하고는 명령문과 데이터문을 한 줄씩 순서대로 입력
        * EX) 
        <img width="500" alt="image (9)" src="https://user-images.githubusercontent.com/55729930/102512640-9bb90780-40cd-11eb-8d01-92d0c499a3da.png">
        
        * 모든 명령이 동일한 인덱스에서 수행되는 경우에는 아래와 같이 `<인덱스명>/_bulk` 형식으로도 사용이 가능
        * 따로따로 수행하는 것 보다 속도가 훨씬 빠름. 특히 대량의 데이터를 입력 할 때는 반드시 _bulk API를 사용해야 불필요한 오버헤드가 없음.
        * 벌크 명령을 json 파일로 저장 후 curl명령으로 실행 가능
            * _bulk API를 bulk.json파일로 저장 후 `curl -XPOST "http://localhost:9200/_bulk" -H 'Content-Type: application/json' --data-binary @bulk.json`를 통해 실행
    * 검색 API ( _search API )
        * 검색은 인덱스 단위로 이루어짐.
        * URI 검색
            * 요청 주소에 _search 뒤에 q 파라메터를 사용해서 검색어를 입력
                * EX) `GET test/_search?q=value` ( URI 검색으로 검색어 "value" 검색 )
            * URI 쿼리에서는 AND, OR, NOT 의 사용이 가능하며 반드시 모두 대문자로 입력
                * EX) `GET test/_search?q=value AND three` ( URI 검색으로 검색어 "value AND three" 검색 )
            * 검색어를 필드에서 찾고 싶으면 다음과 같이 <필드명>:<검색어> 형태로 입력
                * EX) `GET test/_search?q=field:value` ( URI 검색으로 "field" 필드에서 검색어 "value" 검색 )
        * 데이터 본문 (Data Body) 검색
            * 검색 쿼리를 데이터 본문으로 입력하는 방식
            * 쿼리 입력은 항상 query 지정자로 시작, 그 다음 레벨에서 쿼리의 종류를 지정
            * match 쿼리 
                * EX) `GET test/_search {"query": { "match": {"field": "value"  }} }`
            * 쿼리 별로 문법이 상이할 수 있는데 match 쿼리는 <필드명>:<검색어> 방식으로 입력
        * 멀티테넌시 (Multitenancy)
            * 여러 개의 인덱스를 한꺼번에 묶어서 검색
            * `ogs-2018-01`, `logs-2018-02` … 와 같이 날짜별로 저장된 인덱스들이 있다면 이 인덱스들을 모두 `logs-*/_search` 명령으로 한꺼번에 검색이 가능
            * 시간순으로 따라 쌓이는 로그 데이터를 다룰 때는 인덱스를 일단위 등으로 구분하는것이 용이
                * `GET logs-2018-*/_search` ( 와일드카드 * 를 이용해서 여러 인덱스 검색 )
        * 인덱스명 대신 _all 지정자를 사용하여 GET _all/_search 와 같이 실행하면 클러스터에 있는 모든 인덱스를 대상으로 검색이 가능.
            * _all은 시스템 사용을 위한 인덱스 등의 데이터까지 접근, 불필요한 작업 부하를 초래,  _all 은 되도록 지양
    * 검색과 쿼리 ( Query DSL )
        * 검색 (Search)
            * 수많은 대상 데이터 중에서 조건에 부합하는 데이터로 범위를 축소하는 행위
        * 풀 텍스트 쿼리 - Full Text Query
            * match_all
                * 별다른 조건 없이 해당 인덱스의 모든 도큐먼트를 검색하는 쿼리
                * 검색 시 쿼리를 넣지 않으면 elasticsearch는 자동으로 match_all을 적용해서 해당 인덱스의 모든 도큐먼트를 검색
            * match
                * 풀 텍스트 검색에 사용되는 가장 일반적인 쿼리
                * 특정 인덱스의 특정 필드에 특정 키워드가 포함되어 있는 모든 문서를 검색
                    * EX) `GET my_index/_search {"query": {"match": {"message": "dog"}}}` ( match 쿼리로 message 필드에서 dog 검색 )
                * match 검색에 여러 개의 검색어를 집어넣게 되면 디폴트로 OR 조건으로 검색이 되어 입력된 검색어 별로 하나라도 포함된 모든 문서를 모두 검색
                    * EX) `GET my_index/_search { "query": { "match": { "message": "quick dog" } } }` ( match 쿼리로 message 필드에서 quick OR dog 검색 )
                * 검색어가 여럿일 때 검색 조건을 OR 가 아닌 AND 로 바꾸려면 operator 옵션을 사용
                    * EX) `GET my_index/_search { "query": { "match": { "message": { "query": "quick dog", "operator": "and" } } } }` ( match 쿼리 quick AND dog 검색 )
            * match_phrase
                * 공백을 포함해 정확히 일치하는 내용을 검색
                    * EX) `GET my_index/_search { "query": { "match_phrase": { "message": "lazy dog" } } }` ( match_phrase 쿼리로 "lazy dog" 구문 검색 )
                * slop 이라는 옵션을 이용하여 slop에 지정된 값 만큼 단어 사이에 다른 검색어가 끼어드는 것을 허용
                    * EX) `GET my_index/_search { "query": { "match_phrase": { "message": { "query": "lazy dog", "slop": 1 } } } }` ( match_phrase 쿼리에 slop:1 로 "lazy dog" 구문 검색)
                    * "Lazy jumping dog"처럼 사이에 한 단어가 입력된 값도 검색됨.
            * query_string
                * URL검색에 사용하는 루씬의 검색 문법을 본문 검색에 이용하고 싶을 때 query_string 쿼리를 사용
                    * GET my_index/_search { "query": { "query_string": { "default_field": "message", "query": "(jumping AND lazy) OR \"quick dog\"" } } }` ( message 필드에서 lazy와 jumping을 모두 포함하거나 또는 "quick dog" 구문을 포함하는 도큐먼트를 검색하는 쿼리, match_phrase 처럼 구문 검색을 할 때는 검색할 구문을 쌍따옴표 \" 안에 넣음 )
    * Bool 복합 쿼리 - Bool Query
        * 본문 검색에서 여러 쿼리를 조합하기 위해서는 상위에 bool 쿼리를 사용하고 그 안에 다른 쿼리들을 넣는 식으로 사용이 가능
        * Bool 쿼리 인자
            * must : 쿼리가 참인 도큐먼트들을 검색
            * must_not : 쿼리가 거짓인 도큐먼트들을 검색
            * should : 검색 결과 중 이 쿼리에 해당하는 도큐먼트의 점수 증가
            * filter : 쿼리가 참인 도큐먼트를 검색하지만 스코어를 계산안함. must 보다 검색 속도가 빠르고 캐싱이 가능.
        * 사용방법
            * `GET <인덱스명>/_search { "query": { "bool": { "must": [ { <쿼리> }, … ], "must_not": [ { <쿼리> }, … ], "should": [ { <쿼리> }, … ], "filter": [ { <쿼리> }, … ] } } }`
            * EX) `GET my_index/_search { "query": { "bool": { "must": [ { "match": { "message": "quick" } }, { "match_phrase": { "message": "lazy dog" } } ] } } }` (  "quick"과 구문 "lazy dog"가 포함된 모든 문서를 검색하는 쿼리 )
        * must는 SQL의 AND 연산자와 유사하게 동작하지만 bool 쿼리에는 표준 SQL의 OR 와 정확히 일치하게 동작한다고 할 수 있는 연산자는 없음
        * 표준 SQL 과 Elasticsearch Bool 쿼리 비교
         <img width="500" alt="image (10)" src="https://user-images.githubusercontent.com/55729930/102520924-98c31480-40d7-11eb-9f0c-a756725e9c56.png">
         <img width="500" alt="image (11)" src="https://user-images.githubusercontent.com/55729930/102520929-9a8cd800-40d7-11eb-8c89-9069ce4bb539.png">
    * 정확도 - Relevancy
        * Elasticsearch 와 같은 풀 텍스트 검색엔진은 검색 결과가 입력된 검색 조건과 얼마나 정확하게 일치하는 지를 계산하는 알고리즘을 가지고 있어 이 정확도를 기반으로 사용자가 가장 원하는 결과를 먼저 보여줄 수 있다.
        * 스코어 (score) 점수
            * Elasticsearch의 검색 결과에는 스코어 점수가 표시가 됨. 이 점수는 검색된 결과가 얼마나 검색 조건과 일치하는지를 나타내며 점수가 높은 순으로 결과를 보여줌.
            * `GET my_index/_search { "query": { "match": { "message": "quick dog" } } }` 검색 시 각 검색 결과의 _score 항목에 스코어 점수가 표시되고 이 점수가 높은 결과부터 나타남. 상단의 max_score에는 전체 결과 중에서 가장 높은 점수가 표시. 
            * Elasticsearch 는 이 점수를 계산하기 위해 BM25 라는 알고리즘을 이용
                * 도큐먼트 내에 검색된 텀(term)이 더 많을수록 점수가 높아짐.
                * 흔한 단어 보다는 희소한 단어가 검색에 더 중요한 텀일 가능성이 높음. 검색된 텀이 흔할 수록 점수가 감소.
                * 긴 텍스트 보다는 길이가 짧은 텍스트에 검색어를 포함하고 있는 텍스트가 더 점수가 높음.
        * Bool : Should
            * 검색 점수를 조정하기 위해 사용
                * EX) `GET my_index/_search { "query": { "bool": { "must": [ { "match": { "message": "fox" } } ], "should": [ { "match": { "message": "lazy" } } ] } } }` 
                ( fox 검색 결과 중 lazy 를 포함한 결과에 가중치 부여)
                * EX) `GET my_index/_search { "query": { "bool": { "must": [ { "match": { "message": { "query": "lazy dog" } } } ], "should": [ { "match_phrase": { "message": "lazy dog" } } ] } } }`
                ( lazy 또는 dog 를 검색하면서 "lazy dog" 구문을 포함한 결과에 가중치 부여 )
    * 정확값 쿼리 - Exact Value Query
        * Elasticsearch는 정확도를 고려하는 풀 텍스트 외에도 검색 조건의 참 / 거짓 여부만 판별해서 결과를 가져오는 것이 가능
        * bool : filter
            * bool쿼리의 filter 안에 하위 쿼리를 사용하면 스코어에 영향을 주지 않음
                * EX) must 로 fox 및 filter 으로 quick 검색 시 match 쿼리로 fox 를 검색했을 때와 같은 점수
                * `GET my_index/_search { "query": { "bool": { "must": [ { "match": { "message": "fox" } } ], "filter": [ { "match": { "message": "quick" } } ] } } }`
        * keyword
            * 문자열 데이터는 keyword 형식으로 저장하여 정확값 검색이 가능
                * EX) `GET my_index/_search { "query": { "bool": { "filter": [ { "match": { "message.keyword": "Brown fox brown dog" } } ] } } }`
                    * keyword 타입으로 저장된 필드는 스코어를 계산하지 않고 정확값의 일치 여부만을 따짐
    * 범위 쿼리 - Range Query
        * 숫자, 날짜 형식은 range 쿼리를 이용해서 검색
        * range 쿼리는 `range : { <필드명>: { <파라메터>:<값> } }` 으로 입력
        * range 쿼리 파라미터
            * gte (Greater-than or equal to) - 이상 (같거나 큼)
            * gt (Greater-than) – 초과 (큼)  
            * lte (Less-than or equal to) - 이하 (같거나 작음)
            * lt (Less-than) - 미만 (작음)
            * EX) `GET phones/_search { "query": { "range": { "price": { "gte": 700, "lt": 900 } } } }` ( price 값이 700 이상 900 미만인 데이터 검색 )
        * 날짜검색
            * 날짜 값은 2016-01-01 또는 2016-01-01T10:15:30 과 같이 JSON 에서 일반적으로 사용되는 ISO8601 형식을 사용
                * EX) `GET phones/_search { "query": { "range": { "date": { "gt": "2016-01-01" } } } }` ( date 값이 2016-01-01 이후인 데이터 검색 )
            * 쿼리의 날짜 포맷을 다르게 하고 싶으면 `format` 옵션의 사용이 가능, `||` 을 사용해서 여러 값의 입력이 가능
                * EX) `GET phones/_search { "query": { "range": { "date": { "gt": "31/12/2015", "lt": "2018", "format": "dd/MM/yyyy||yyyy" } } } }` ( date 값이 2016-01-01 ~ 2018-01-01 사이의 데이터 검색 )
            * 날짜를 검색 할 때 예약어 사용가능
                * `now`(현재시간), `y`(년), `M`(월), `d`(일), `h`(시), `m`(분), `s`(초), `w`(주) 
            * range는 true / false 여부만을 판단하여 쿼리의 스코어는 모두 "_score" : 1.0로 동일
11. 인덱스 설정과 매핑    
    * `GET <인덱스명>/_settings`,`GET <인덱스명>/_mappings`로 조회하면 설정(settings) 그리고 매핑(mappings) 정보를 확인할 수 있다.
    * 설정 - Settings
        * number_of_shards, number_of_replicas
            * 프라이머리 샤드 수와 레플리카 수 설정
            ```
            PUT my_index
            {
              "settings": {
                "index": {
                  "number_of_shards": 3,
                  "number_of_replicas": 1
                }
              }
            }
            ```
        * refresh_interval
            * 세그먼트가 만들어지는 리프레시 타임을 설정하는 값, 기본은 1초
            * refresh_interval 을 30초로 my_index 생성
            ```
            PUT my_index
            {
              "settings": {
                "refresh_interval": "30s"
              }
            }
            ```
        * analyzer, tokenizer, filter
            *  애널라이저, 토크나이저, 토큰 필터 역시 setting 내부에 정의
            ```
            PUT my_index
            {
              "settings": {
                "analysis": {
                  "analyzer": {
                    "my_analyzer": {
                      "type": "custom",
                      "char_flter": [ "...", "..." ... ]
                      "tokenizer": "...",
                      "filter": [ "...", "..." ... ]
                    }
                  },
                  "char_filter":{
                    "my_char_filter":{
                      "type": "…"
                      ... 
                    }
                  }
                  "tokenizer": {
                    "my_tokenizer":{
                      "type": "…"
                      ...
                    }
                  },
                  "filter": {
                    "my_token_filter": {
                      "type": "…"
                      ...
                    }
                  }
                }
              }
            }
            ```
    * 매핑 - Mappings
        * 매핑
            * 데이터가 입력되어 자동으로 매핑이 생성되기 전에 미리 먼저 인덱스의 매핑을 정의 해 놓으면 정의 해 놓은 매핑에 맞추어 데이터가 입력됨
            * 인덱스의 매핑 정의
            ```
            PUT <인덱스명>
            {
              "mappings": {
                "properties": {
                  "<필드명>":{
                    "type": "<필드 타입>"
                    … <필드 설정>
                  }
                  …
                }
              }
            }
            ```
            * 이미 만들어진 매핑에 필드를 추가하는것은 가능하지만 이미 만들어진 필드를 삭제하거나 필드의 타입 및 설정값을 변경하는 것은 불가능하다. 
            * 기존 매핑에 필드 추가
            ```
            PUT <인덱스명>/_mapping
            {
              "properties": {
                "<추가할 필드명>": { 
                  "type": "<필드 타입>"
                  … <필드 설정>
                }
              }
            }
            ```
            * 필드의 변경이 필요한 경우 인덱스를 새로 정의하고 기존 인덱스의 값을 새 인덱스에 모두 재색인한다.
        * 동적(Dynamic) 매핑
            * 미리 정의하지 않아도 인덱스에 도큐먼트를 새로 추가하면 자동으로 매핑이 생성
        * 문자열 - text, keyword
            * text
                * 입력된 문자열을 텀 단위로 쪼개어 역 색인 (inverted index) 구조
                * 설정 가능한 옵션
                    * `"analyzer" : "<애널라이저명>"` 색인에 사용할 애널라이저를 입력
                    * `"search_analyzer" : "<애널라이저명>"` 색인에 사용한 애널라이저가 아닌 다른 애널라이저를 사용
                    * `"index" : <true | false>` 디폴트는 true. false로 설정하면 해당 필드는 역 색인을 만들지 않아 검색이 불가능
                    * `"boost" : <숫자 값>` 디폴트는 1. 1 보다 높으면 풀텍스트 검색 시 해당 필드 스코어 점수에 가중치 증가. 1보다 낮은 값을 입력하면 가중치 하락.
                    * `"fielddata" : <true | false>` 디폴트는 false. true로 설정하면 해당 필드의 색인된 텀 들을 가지고 집계(aggregation) 또는 정렬(sorting)이 가능
            * keyword
                * 입력된 문자열을 하나의 토큰으로 저장합니다. text 타입에 keyword 애널라이저를 적용 한 것과 동일
        * 숫자
            * long : 64비트 정수 (-9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807)
            * integer : 32비트 정수 (-2147483648 ~ 2147483647)
            * short : 16비트 정수 (-32768 ~ 32767)
            * byte : 8비트 정수 (-128 ~ 127)
            * double : 64비트 실수
            * float : 32비트 실수
            * half_float : 16비트 실수
            * scaled_float : 실수형이지만 부동소수점이 아니라 long 형태로 저장하고 옵션으로 소수점 위치를 지정
        * 날짜 - date
            * 날짜 값은 2016-01-01 또는 2016-01-01T10:15:30 과 같이 JSON 에서 일반적으로 사용되는 ISO8601 형식을 사용
                * EX) `GET phones/_search { "query": { "range": { "date": { "gt": "2016-01-01" } } } }` ( date 값이 2016-01-01 이후인 데이터 검색 )
            * 쿼리의 날짜 포맷을 다르게 하고 싶으면 `format` 옵션의 사용이 가능, `||` 을 사용해서 여러 값의 입력이 가능
                * EX) `GET phones/_search { "query": { "range": { "date": { "gt": "31/12/2015", "lt": "2018", "format": "dd/MM/yyyy||yyyy" } } } }` ( date 값이 2016-01-01 ~ 2018-01-01 사이의 데이터 검색 )
            * 날짜를 검색 할 때 예약어 사용가능
                * `now`(현재시간), `y`(년), `M`(월), `d`(일), `h`(시), `m`(분), `s`(초), `w`(주) 
                * 
                |심볼|의미|예) 2019-09-12T17:13:07.428+09:00RDB|
                |------|-----|-----|
                |yyyy|년도|2019|
                |MM|월 - 숫자|09|
                |MMM|월 - 문자 (3자리)|Sep|
                |MMMM|월 - 문자 (전체)|September|
                |dd|일|12|
                |a|오전 / 오후|PM|
                |HH|시각 (0~23)|17|
                |kk|시각 (01~24)|17|
                |hh|시각 (01~12)|05|
                |h|시각 (1~12)|5|
                |mm|분 (00~59)|13|
                |m|분 (0~59)|13|
                |ss|초 (00~59)|07|
                |s|초 (0~59)|7|
                |SSS|밀리초|428|
                |Z|타임존|+0900 / +09:00|
                |e|요일 (숫자 1:월 ~ 7:일)|4|
                |E|요일 (텍스트)|Thu|
        * 불리언 - boolean
            * true 와 false 두가지 값을 갖는 필드 타입. 
            * 선언은 "type": "boolean"
            * "true" 와 같이 문자열로 입력이 되어도 true 로 해석이 되어 저장.
        * Object
            * 한 필드 안에 하위 필드를 넣는 object, 즉 객체 타입의 값을 사용할 수 있다.
            * "name", "age", "side" 를 가진 object 타입 "characters" 필드
                * 선언
                ```
                PUT movie
                {
                  "mappings": {
                    "properties": {
                      "characters": {
                        "properties": {
                          "name": {
                            "type": "text"
                          },
                          "age": {
                            "type": "byte"
                          },
                          "side": {
                            "type": "keyword"
                          }
                        }
                      }
                    }
                  }
                }
                ```
                * 도큐먼트 입력
                ```
                PUT movie/_doc/1
                {
                  "characters": {
                    "name": "Iron Man",
                    "age": 46,
                    "side": "superhero"
                  }
                }
                ```
                * object 필드를 쿼리로 검색 하거나 집계를 할 때는 다음과 같이 `.` 를 이용해서 하위 필드에 접근
                ```
                GET movie/_search
                {
                  "query": {
                    "match": {
                      "characters.name": "Iron Man"
                    }
                  }
                }
                ```
                * 역 색인은 필드 별로 생성
                ```
                PUT movie/_doc/2
                {
                  "title": "The Avengers",
                  "characters": [
                    {
                      "name": "Iron Man",
                      "side": "superhero"
                    },
                    {
                      "name": "Loki",
                      "side": "villain"
                    }
                  ]
                }

                PUT movie/_doc/3
                {
                  "title": "Avengers: Infinity War",
                  "characters": [
                    {
                      "name": "Loki",
                      "side": "superhero"
                    },
                    {
                      "name": "Thanos",
                      "side": "villain"
                    }
                  ]
                }
                ```
                위와 같이 생성 시 역 색인 구조
                
                <img width="540" alt="image (12)" src="https://user-images.githubusercontent.com/55729930/102535202-3b38c300-40eb-11eb-82aa-7a30636e5dde.png">
         * Nested
            * 여러 개의 object 값들이 서로 다른 역 색인 구조를 갖도록 하려면 nested 타입으로 지정
            * nested 타입으로 지정하려면 매핑에 다음과 같이 "type": "nested" 를 명시합니다. 다른 부분은 object 와 동일
            * object 도큐먼트와 nested 도큐먼트를 그림으로 비교
            <img width="540" alt="image (13)" src="https://user-images.githubusercontent.com/55729930/102535443-936fc500-40eb-11eb-9444-c735ca9cfd7e.png">
        * 위치 정보 - Geo
            * Geo Point
                * 위도(latitude)와 경도(longitude) 두 개의 실수 값을 가지고 지도 위의 한 점을 나타내는 값
                *  geo_point 입력 예시
                ```
                object 형식
                PUT my_locations/_doc/1
                {
                  "location": {
                    "lat": 41.12,
                    "lon": -71.34
                  }
                }
                text 형식
                PUT my_index/_doc/2
                {
                  "location": "41.12,-71.34"
                }
                ```
                * Geo Point 필드 매핑선언
                ```
                PUT my_geo
                {
                  "mappings": {
                    "properties": {
                      "location": {
                        "type": "geo_point"
                      }
                    }
                  }
                }
                ```
                * geo_bounding_box 쿼리
                ```
                데이터 입력
                PUT my_geo/_bulk
                {"index":{"_id":"1"}}
                {"station":"강남","location":{"lon":127.027926,"lat":37.497175},"line":"2호선"}
                {"index":{"_id":"2"}}
                {"station":"종로3가","location":{"lon":126.991806,"lat":37.571607},"line":"3호선"}
                {"index":{"_id":"3"}}
                {"station":"여의도","location":{"lon":126.924191,"lat":37.521624},"line":"5호선"}
                {"index":{"_id":"4"}}
                {"station":"서울역","location":{"lon":126.972559,"lat":37.554648},"line":"1호선"}
                
                두 점을 기준으로 하는 네모 영역 안에 있는 도큐먼트 GET
                GET my_geo/_search
                {
                  "query": {
                    "geo_bounding_box": {
                      "location": {
                        "bottom_right": {
                          "lat": 37.4899,
                          "lon": 127.0388
                        },
                        "top_left": {
                          "lat": 37.5779,
                          "lon": 126.9617
                        }
                      }
                    }
                  }
                }
                ```
                <img width="320" alt="image (14)" src="https://user-images.githubusercontent.com/55729930/102537550-6c66c280-40ee-11eb-9b32-6ff45429fda7.png">
                
                ```
                한 점을 기준으로 입력한 반경의 원 안에 있는 도큐먼트 GET
                GET my_geo/_search
                {
                  "query": {
                    "geo_distance": {
                      "distance": "5km",
                      "location": {
                        "lat": 37.5358,
                        "lon": 126.9559
                      }
                    }
                  }
                }
                ```
                <img width="320" alt="image (15)" src="https://user-images.githubusercontent.com/55729930/102537559-6ec91c80-40ee-11eb-9b79-c25e2f1fa435.png">
                
            * Geo Shape
                * 선, 면 등의 2차원 값을 저장하고 쿼리가능
                * geo_shape 필드 선언
                ```
                PUT my_shape
                {
                  "mappings": {
                    "properties": {
                      "location": {
                        "type": "geo_shape"
                      }
                    }
                  }
                }
                ```
                * 단일 점 값 입력
                ```
                "type": "point" 형태의 geo_shape 값 입력
                PUT my_shape/_doc/1
                {
                  "location": {
                    "type": "point",
                    "coordinates": [
                      127.027926,
                      37.497175
                    ]
                  }
                }
                ```
                <img width="320" alt="image (16)" src="https://user-images.githubusercontent.com/55729930/102537563-6f61b300-40ee-11eb-819d-efd4f73c5594.png">
                
                * 여러 점 값 입력
                ```
                "type": "multipoint" 형태의 geo_shape 값 입력
                PUT my_shape/_doc/2
                {
                  "location": {
                    "type": "multipoint",
                    "coordinates": [
                      [ 127.027926, 37.497175 ],
                      [ 126.991806, 37.571607 ],
                      [ 126.924191, 37.521624 ],
                      [ 126.972559, 37.554648 ]
                    ]
                  }
                }
                ```
                <img width="320" alt="image (17)" src="https://user-images.githubusercontent.com/55729930/102537570-6ffa4980-40ee-11eb-9fc5-d3fce631d468.png">
                
                * 직선 값 입력
                ```
                "type": "linestring" 형태의 geo_shape 값 입력
                PUT my_shape/_doc/3
                {
                  "location": {
                    "type": "linestring",
                    "coordinates": [
                      [ 127.027926, 37.497175 ],
                      [ 126.991806, 37.571607 ]
                    ]
                  }
                }
                ```
                <img width="320" alt="image (18)" src="https://user-images.githubusercontent.com/55729930/102537573-7092e000-40ee-11eb-9fef-882c76dc85d9.png">
                
                * 여러개의 직선 값 입력
                ```
                "type": "multilinestring" 형태의 geo_shape 값 입력
                PUT my_shape/_doc/4
                {
                  "location": {
                    "type": "multilinestring",
                    "coordinates": [
                      [
                        [ 127.027926, 37.497175 ],
                        [ 126.991806, 37.571607 ]
                      ],
                      [
                        [ 126.924191, 37.521624 ],
                        [ 126.972559, 37.554648 ]
                      ]
                    ]
                  }
                }
                ```
                <img width="320" alt="image (19)" src="https://user-images.githubusercontent.com/55729930/102537574-712b7680-40ee-11eb-94e9-ee595903af8e.png">
                
                * 다각형 값 입력 - 배열 마지막에는 반드시 처음과 같은 점이 입력
                ```
                "type": "polygon" 형태의 geo_shape 값 입력
                PUT my_shape/_doc/5
                {
                  "location": {
                    "type": "polygon",
                    "coordinates": [
                      [
                        [ 127.027926, 37.497175 ],
                        [ 126.991806, 37.571607 ],
                        [ 126.924191, 37.521624 ],
                        [ 126.972559, 37.554648 ],
                        [ 127.027926, 37.497175 ]
                      ]
                    ]
                  }
                }
                ```
                <img width="320" alt="image (20)" src="https://user-images.githubusercontent.com/55729930/102537579-725ca380-40ee-11eb-9043-d1494dae946f.png">
                
                * 여러 개의 다각형 값 입력
                ```
                "type": "multipolygon" 형태의 geo_shape 값 입력
                PUT my_shape/_doc/6
                {
                  "location": {
                    "type": "multipolygon",
                    "coordinates": [
                      [
                        [
                          [ 127.027926, 37.497175 ],
                          [ 126.991806, 37.571607 ],
                          [ 126.924191, 37.521624 ],
                          [ 127.004943, 37.504810 ],
                          [ 127.027926, 37.497175 ]
                        ]
                      ],
                      [
                        [
                          [ 126.936893, 37.555134 ],
                          [ 126.967894, 37.529170 ],
                          [ 126.924191, 37.521624 ],
                          [ 126.936893, 37.555134 ]
                        ]
                      ]
                    ]
                  }
                }
                ```
                <img width="320" alt="image (21)" src="https://user-images.githubusercontent.com/55729930/102537583-72f53a00-40ee-11eb-8d40-5d70aacdadaa.png">
                
                * 직사각형 영역 값 입력
                ```
                "type": "envelope" 형태의 geo_shape 값 입력
                PUT my_shape/_doc/7
                {
                  "location": {
                    "type": "envelope",
                    "coordinates": [
                      [ 126.936893, 37.555134 ],
                      [ 127.004943, 37.50481 ]
                    ]
                  }
                }
                ```
                <img width="320" alt="image (22)" src="https://user-images.githubusercontent.com/55729930/102537587-74266700-40ee-11eb-82d7-8012ec9cf72d.png">
                
        * IP
            * IP 주소 형식을 저장
            * 매핑은 `"type": "ip"` 으로 선언
            * `"192.168.1.1"` 같은 IPv4 형식과 `"0:0:0:0:0:ffff:c0a8:105"` 같은 IPv6 형식을 문자열 처럼 입력
        * 범위(Range)
            * 숫자나 날짜, IP 등을 시작과 끝이 있는 2차원의 범위 형태로 저장
            * 매핑의 "type" 에integer_range, float_range, long_range, double_range, date_range, ip_range 등으로 선언
            ```
            integer_range 와 date_range 타입의 필드 선언
            PUT my_range
            {
              "mappings": {
                "properties": {
                  "amount": {
                    "type": "integer_range"
                  },
                  "days": {
                    "type": "date_range"
                  }
                }
              }
            }
            ```
            * 데이터의 범위는 다음과 같이 gt, gte, lt, lte 를 사용해서 지정
            ```
            integer_range, date_range 타입의 값을 가진 도큐먼트 입력
            PUT my_range/_doc/1
            {
              "amount": {
                "gte": 19,
                "lt": 28
              },
              "days": {
                "gt": "2019-06-01T09:00:00",
                "lt": "2019-06-20"
              }
            }
            ```
            * 범위 데이터를 range 쿼리로 검색 할 때는 추가로 relation 옵션의 값을 입력해야 하며 입력하지 않으면 오류
                * within : 도큐먼트 범위 값이 쿼리한 범위 안에 완전히 포함되는 도큐먼트들을 가져옴.
                * contains : within과 반대로 쿼리 범위가 도큐먼트 범위 값 안에 완전히 포함되는 도큐먼트들을 가져옴.
                * Intersects : 도큐먼트 범위 값과 쿼리 범위에 공통적인 부분이 있는 도큐먼트들을 가져옴.
        * Binary
            * "type": "binary" 로 지정해서 시스템 파일이나 이미지 정보 같은 바이너리 값을 저장
    * 멀티 (다중) 필드 - Multi Field
        * 도큐먼트에는 하나의 필드값만 있지만 이 필드의 값을 여러 개의 역 색인 및 doc_values 들로 저장
        * `"fields" : { }` 항목에서 다시 새로운 필드를 정의하고 설정
        *  여러 개의 애널라이저를 적용하기 위해 사용
        ```
         my_index 인덱스의 message 필드에 서로 다른 애널라이저들을 사용하는 english, nori 멀티 필드를 정의하는 예제
         PUT my_index
        {
          "settings": {
            "analysis": {
              "analyzer": {
                "nori_analyzer": {
                  "tokenizer": "nori_tokenizer"
                }
              }
            }
          },
          "mappings": {
            "properties": {
              "message": {
                "type": "text",
                "fields": {
                  "english": {
                    "type": "text",
                    "analyzer": "english"
                  },
                  "nori": {
                    "type": "text",
                    "analyzer": "nori_analyzer"
                  }
                }
              }
            }
          }
        }
        ```
        `{ "message": "My favorite 슈퍼영웅 is Iron Man" }` 이라는 값을 입력하면 다음과 같이 3개의 역 색인이 생성
        <img width="450" alt="image (23)" src="https://user-images.githubusercontent.com/55729930/102539077-80abbf00-40f0-11eb-9f76-1a35ff466a05.png">

### 2.2 Logstash
* 기능
   * 오픈소스 서버측 데이터 수집 & 처리 파이프라인도구
   * 각 데이터베이스의 데이터, 로우 데이터, 윈도우 이벤트 등으로부터 데이터를 수집
   * 다양한 플러그인을 이용하여 데이터를 집계 및 보관, 서버 데이터 처리, 파이프라인으로 데이터를 수집
   * 수집한 데이터를 필터를 통해 변환 후 ElasticSearch로 전송
   * Pipeline 구조(Input -> Filter -> Output)
   
* 개념정리

1) input
   - 입력을 사용하여 Logstash에 데이터를 받아들임
   - 데이터 타입 - file, syslog(RFC3164 형식), beats(Filebeat)
   
2) filter
   - Logstash 파이프 라인의 중간 처리 장치로 데이터를 가공
      - grok : 구문 분석 및 임의의 텍스트로 구성
      - mutate : 이벤트 필드에서 일반적인 변환을 수행
      - drop : 이벤트를 삭제
      - clone : 이벤트의 복사본 생성
      - geoip : ip 주소의 지리적 위치에 대한 정보를 추가
      
3) output
   - Logstash 파이프 라인의 최종 단계로 데이터를 출력
      - elasticsearch : 이벤트 데이터를 elasticsearch에 전송
      - file : 디스크 파일에 작성
      - graphite : graphite에 전송 (graphite : 메트릭을 저장하고 그래프로 작성하는데 사용되는 오픈 소스 도구)
      - statsd : 카운터 및 타이머와 같은 통계를 수신하고 UDP를 통해 전송되며, 하나 이상의 플러그 가능한 백엔드 서비스에 집계를 보내는 서비스
      
4) codec
   - 입력 또는 출력의 일부로 작동 할 수 있는 스트림 필터
   - 대표적인 코덱에는 json, msgpack, plain이 있다.
      - json : JSON 형식의 데이터를 인코딩하거나 디코딩
      - multiline : 자바 예외 및 스택 추척 메시지와 같은 여러 줄 텍스트 이벤트를 단일 이벤트로 병합

### 2.3 Kibana
* 기능
   * 사용자 Application으로 Elastic Search에 저장된 정보들을 검색 및 분석하고 실시간으로 시각화하는 기능
   
### 2.4 Filterbeats
* 기능
   * 서버에 경량 에이전트로 설치되어 다양한 유형의 데이터를 Logstash 또는 ElasticSearch로 전송(스트리밍)하는 오픈 소스 데이터 발송자
   * Filebeat, Metricbeat, Packetbeat, Winlogbeat, Heartbeat 등이 있으며 Libbeat을 이용하여 직접 구축도 가능
   * 개념정리

1) FileBeat
   - 서버에서 로그파일을 제공
2) PacketBeat
   - 응용 프로그램 서버간에 교환되는 트랜잭션에 대한 정보를 제공하는 네트워크 패킷 분석기 
3) MetricBeat
   - 서버에서 실행중인 운영 체제 및 서비스에서 Metrics를 주기적으로 수집하는 서버 모니터링 에이전트
4) WinlogBeat
   - Windows 이벤트 로그를 제공


* RDB가 읽기성능이 낮은 이유
    - 관계형 DB에서는 Text 열을 한 줄씩 찾아 내려가면서 키워드가 있으면 가져오고 없으면 넘어감. 
    - row 안의 내용을 모두 읽어야 하기 때문에 기본적으로 속도가 느림.
* Active-Standby
    * 평상 시에는 하나의 서버로 운영을 하고, 그와 스펙이 비슷한 서버를 예비로 준비. 예비서버는 언제라도 부팅하면 바로 사용 할 수 있는 상태로 준비 합니다. 장애가 발생되면 관리자가 판단하여 예비서버로 서비스 해야 한다고 판단이 되면 Data영역의 디스크를 분리하여 예비서버로 이전한 후 예비서버로 서비스를 운영하면 됩니다. 
    * 이 형태는 저렴한 비용으로 이중화를 구성할 수 있지만, 시스템 장애 시간이 길고 안정성이 적은 방법입니다.
* Active-Active
    * 사용자가 각 서버별로 고루 분산되도록 구성이 되고, 사용자는 자신이 속한 서버에서만 서비스를 이용 
        - 한쪽 서버로 사용자가 몰리는 경우가 생길 수 있으므로, 사용자 생성 시 고루 분포되도록 유의
    * 장애가 발생하면 클러스터링(Clustering) SW에 의해 감지가 되고 자동으로 서비스 절체 및 이전 
        - 서버#1에 장애가 발생하면 서버#1의 정보가 서버#2로 이전되어 서버#2에서 서버#1의 서비스를 대신 수행하므로서 시스템 장애 시간을 최소
2. split brain elasticsearch
    - Split Brain
        - 네트워크 단절로 마스터 후보 노드가 분리되면 각자가 서로 다른 클러스터로 구성되어 계속 동작, 네트워크가 복구 되고 하나의 클러스터로 다시 합쳐졌을 때 데이터 정합성에 문제가 생기고 데이터 무결성 손실되는 현상.
        - 마스터 후보 노드를 하나만 놓게 되면 그 마스터 노드가 유실되었을 때 클러스터 전체가 작동을 정지 할 위험
        - 마스터 후보 노드들은 3개 이상의 홀수 개를 놓는 것을 권장
        <img width="500" alt="image (3)" src="https://user-images.githubusercontent.com/55729930/102223064-91f59f80-3f27-11eb-96d6-a1df8eafb709.png">
        
3. elasticsearch shard performance
    * 샤드수가 많은 경우 데이터가 분산되어 검색 속도는 빨라질 수 있지만, 모든 샤드를 관리하는 마스터 노드의 부하도 덩달아서 증가하여 색인같은 작업도 덩달아 느려질 수 있다.
        - 마스터 노드의 다운은 클러스터 전체의 다운으로 이어지기 때문에 주의
    * 샤드의 크기가 어떻게 성능에 영향을 미치나요?
        - 일단 노드에 장애가 발생하면 프라이머리와 동일한 데이터를 가지고 있는 레플리카 샤드가 순간적으로 프라이머리 샤드로 전환되어 서비스된다. 그와 동시에 프라이머리로 전환된 샤드의 레플리카가 다른 노드에서 새로 생성된다. 시간이 지나 장애가 복구되면 복구된 노드로 일부 샤드들이 네트워크를 통해 이동한다. 시간이 지남에 따라 클러스터에 균형이 맞추어진다.
        - 이러한 프로세스가 있기때문에 단일 샤드의 물리적 크기가 크다면 recovery 가 신속히 이루어지지 못할 위험이 있다.
