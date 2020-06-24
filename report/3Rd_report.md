# GET
 curl - XGET http://localhost:9200/class
 curl - XGET http://localhost:9200/class?pretty
 curl - XPUT http://localhost:9200/class
 curl - XPUT http://localhost:9200/class
 curl - XDELETE http://localhost:9200/class
 curl -XPOST http://localhost:9200/classes/class/1/ -d '
> {"title" : "Algorithm", "professor" : "John"}'
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


