## 1.엘라스틱 서치 데이터 입력 조회 삭제
|ES|RDB|의미|
|---|---|---|
|GET|Select|Index조회
|PUT|Update|Index생성
|POST|Insert|Doc생성|
|DELETE|Delete|Index삭제

1. Index 생성 

* class이름의 Index 생성
<pre>
<code>
 curl - XPUT http://localhost:9200/class
 </code>
</pre>
	

2. Index 조회

* class이름의 Index 조회
<pre>
<code>
 curl - XGET http://localhost:9200/class?pretty
 </code>
</pre>
	* ?pretty
		* 결과를 예쁘게 출력
		
3. 데이터 삭제

* Document, Type, Index 단위로 삭제할 수 있다.
* Index 단위 삭제 후 URL로 접근하면 404반환
* class이름의 Index 삭제
<pre>
<code>
 curl - XDELETE http://localhost:9200/class
 </code>
</pre>

4. Doc 생성


4.1. 직접 입력
* classes/class/1에 '{"title" : "Algorithm", "professor" : "John"}'(doc) 을 json포맷으로 입력
* {index}/{type}/{id}
<pre>
<code>
 curl -XPOST http://localhost:9200/classes/class/1/ -d '{"title" : "Algorithm", "professor" : "John"}' -H 'content-Type: application/json'
 </code>
</pre>
	* -d 옵션
		* 추가할 데이터를 json 포맷으로 전달
	* -H 옵션
		* 헤더를 명시 json으로 전달하므로 application/json으로 작성
* 
4.2. 파일 사용
* 동일 경로에 oneclass.json파일 입력
<pre>
<code>
 curl -XPOST http://localhost:9200/classes/class/1/ -d @oneclass.json -H 'content-Type: application/json'
 </code>
</pre>
	* @ 옵션
		* json파일로 데이터 추가

## 2.엘라스틱 서치 데이터 업데이트 
1. Doc 업데이트

1.1. 필드 추가
* classes(Index)/class(type)/1(_id)에 { "doc":{"unit":1}}'추가
<pre>
<code>
curl -XPOST http://localhost:9200/classes/class/1/_update -H 'content-Type: application/json'  -d ' 
{ "doc":{"unit":1}}'
 </code>
</pre>

1.2. 변경
* classes(Index)/class(type)/1(_id)의 unit을 2로 변경
<pre>
<code>
curl -XPOST http://localhost:9200/classes/class/1/_update -H 'content-Type: application/json'  -d ' 
{ "doc":{"unit":2}'
 </code>
</pre>

1.3. 연산
* Script를 통해 Document Update
* classes(Index)/class(type)/1(_id)의 unit에 +5 연산
<pre>
<code>
curl -XPOST http://localhost:9200/classes2/class/1/_update -H 'content-Type: application/json'  -d '
{ "script" : "ctx._source.unit += 5"}' 
 </code>
</pre>
	* ctx._source
		* 업데이트하려는 현재 소스 문서
		
## 3.엘라스틱 서치 - 벌크(Bulk)

* 입력할 데이터를 모아 한꺼번에 처리하므로 데이터를 각각 처리하고 결과를 반환할 때보다 속도가 매우 빠르다.
* 많은 Document를 한꺼번에 색인할 때 벌크를 사용하면 색인에 소요되는 시간을 크게 줄일 수 있다.
* Action Meta data와 Request Body가 각각 한 쌍씩 묶여 동작한다
	* Delete는 Action Meta data만 필요하다.
* 통상적으로 1000~5000개 정도의 작업이 바람직, 10000개 이상의 작업을 배치로 실행하면 오류가 발생할 확률이 높다.
Bulk Data Example
<pre>
<code>
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } } # 반드시 줄 바꿈으로 Meta data와 Body를 구분한다, ~인덱스에 ~타입에 아이디~번에 이하 doc를 넣어라
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } } # delete는 Body가 필요없다.
{ "create" : { "_index" : "test", "_type" : "_doc", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
</code>
</pre>

* classes.json 벌크 
<pre>
<code>
curl -XPOST localhost:9200/_bulk?pretty --data-binary @classes.json -H 'content-Type: application/json'
 </code>
</pre>
	*--data-binary @ 파일명
		* 벌크파일에서 데이터 불러옴

## 3.엘라스틱 서치 - 매핑(Mapping)


데이터의 포맷을 제대로 지정해주지 않으면 데이터를 관리할때 장애가 발생 
키바나와 효과적으로 연동하기 위해서 효율적인 매핑작업이 반드시 필요
1. Mapping 만들기
1.1. 직접 매핑 작성
1.2. 임시 인덱스를 만들고 그 자동 생성된 매핑을 가져와서 원하는 매핑을 만드는 방법
|필드값|설명|
|---|--|
|name|이름|
|age|나이|
|gender|성별|
|memo|메모|
|reg_date|등록일|
위와같은 인덱스를 만든다고 가정
<pre>
<code>
PUT imsi_my_test/person/1 
{
  "name": "kim",
  "age": 40,
  "gender": "male",
  "memo": "This is good man!",
  "reg_date": "2018-03-01"
}
 </code>
</pre>
임시인덱스 만들고, 자동 생성된 매핑 가져오기 (GET imsi_my_test/_mapping)
<pre>
<code>
{
  "imsi_my_test": {
    "mappings": {
      "person": {
        "properties": {
          "age": {
            "type": "long"
          },
          "gender": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "memo": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "reg_date": {
            "type": "date"
          }
        }
      }
    }
  }
}
 </code>
</pre>
가져온 후, 만들 인덱스에 생성 
<pre>
<code>
PUT my_test/
{
    "mappings": {
      "person": {
        "properties": {
          "age": {
            "type": "integer"
          },
          "gender": {
            "type": "keyword"
          },
          "memo": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "name": {
            "type": "keyword"
          },
          "reg_date": {
            "type": "date"
          }
        }
      }
    }
}
 </code>
</pre>

GET my_test/_mapping을 통해 매핑여부 확인


2. 매핑
* 인덱스 생성 시 매핑이 되어있지 않음 
<pre>
<code>
curl -XPUT 'http://localhost:9200/classes?pretty
</code>
</pre>

* classes/class/에 매핑 추가
<pre>
<code>
curl -XGET 'http://localhost:9200/classes/class/_mapping' -H 'content-Type: application/json' -d @classesRating_mapping.json
 </code>
</pre>
	* {index}/{type}/_mapping
		* 매핑
* 매핑 후 인덱스 추가하면 데이터 포맷 지정됨


--------------------------------------------
<pre>
<code>

 </code>
</pre>
# GET
 curl - XGET http://localhost:9200/class
 curl - XGET http://localhost:9200/class?pretty
 curl - XPUT http://localhost:9200/class
 curl - XPUT http://localhost:9200/class
 curl - XDELETE http://localhost:9200/class
 curl -XPOST http://localhost:9200/classes/class/1/ -d '{"title" : "Algorithm", "professor" : "John"}'
에러발생 
{"error":"Content-Type header [application/x-www-form-urlencoded] is not supported","status":406}
아래링크 참고하여 해결
https://abc2080.tistory.com/entry/%EC%97%90%EB%9F%AC-ContentType-header-applicationxwwwformurlencoded-is-not-supported
-H 'Content-Type: application/json'


curl -XPOST http://localhost:9200/classes/class/1/ -H 'content-Type: application/json' -d @onclass.json
onclass.json의 파일로 doc생성

curl -XPOST http://localhost:9200/classes/class/1/_update -H 'content-Type: application/json'  -d ' 
{ "doc":{"unit":1}}'
1 

curl -XPOST http://localhost:9200/classes2/class/1/_update -H 'content-Type: application/json'  -d '
{ "script" : "ctx._source.unit += 5"}' 

데이터 넣을때 타입 지정 안하면 데이터를 관리할때 장애가 발생 > 매핑하여 타입을 넣어주고 > 어그리게이션 시각화할때 타입활용해서 더욱 분석적으로 할  수 있음
property 추가하여 
curl -XGET 'http://localhost:9200/classes/class/_mapping' -H 'content-Type: application/json' -d @classesRating_mapping.json
매핑

curl -XPOST http://localhost:9200/_bulk?pretty --data-binary @classes.json -H 'content-Type: application/json'
벌크 매핑


curl -XPOST 'localhost:9200/_bulk' --data-binary @simple_basketball.json -H 'content-Type: application/json' 
add 서치할 documents 

curl -XGET localhost:9200/basketball/record/_search?pretty
               주소           인덱스    타입    서치옵션
서치하기

curl -XGET 'localhost:9200/basketball/record/_search?q=points:30&pretty'
 							 서치옵션:point=30인 도큐만

curl -XGET localhost:9200/basketball/record/_search-d?'
"query":{ 쿼리를 사용하는데
"term":{"points":30} 텀쿼리의 포인트가 30인 doc만 출력 
}
}'
리퀘스트 바디 사용 서치 -d 줌으로써 다이렉트옵션

# 매트릭 어그리게이션
curl -XPOST localhost:9200/_bulk --data-binary @simple_basketball.json -H 'content-Type: application/json' 
basketball.json 벌크등록


curl -XGET localhost:9200/_search?pretty --data-binary @(min,max,avg,sum,stats)_points_aggs.json -H 'content-Type: application/json' 

# 버킷 어그리게이션

curl -XPUT 'localhost:9200/basketball2/record/_mapping' -d @basketball_mapping.json -H 'content-Type: application/json' 
실행시 오류발생
오류코드 
{"error":{"root_cause":[{"type":"mapper_parsing_exception","reason":"No handler for type [string] declared on field [name]"}],"type":"mapper_parsing_exception","reason":"No handler for type [string] declared on field [name]"},"status":400}
구글링결과 string을 지원안함 text로 변경후 해결

curl -XPOST 'localhost:9200/_bulk' --data-binary @twoteam_basketball.json -H 'content-Type: application/json' 
doc 벌크 삽입

curl -XGET localhost:9200/_search?pretty --data-binary @terms_aggs.json -H 'content-Type: application/json' 
어그리게이션(팀별로 그룹) 결과 illegal_argument_exception 오류 발생 (해결못함)

팀별로 doc 묶은 후 points 별로 stats 표시 
curl -XGET localhost:9200/_search?pretty --data-binary @stats_by_team.json -H 'content-Type: application/json' 

어그리게이션(팀별로 그룹) 결과 illegal_argument_exception 오류 발생 (해결못함)


