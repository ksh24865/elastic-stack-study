## in tutorial1~2
	* Elasticsearch
		* 설치 및 head플러그인 설치
	* Kibana 
		* 설치 및 호스트0.0.0.0으로 설정하여 외부에서 접근가능하도록 변경 
		* Elasticsearch의 호스트인 9200과 연결
		* .kibana 인덱스 추가  // seongho인덱스 추가
	* Filebeat
		* 해당 경로의 *.log파일 스트리밍하도록 추가
		* output으로 Elasticsearch의 호스트인 9200과 연결
	* systemd에 service를 등록하여 Elastic Stack 실행

## in tutorial3

	* logstash
		* 필터 없이 input을 입력받아 output출력
## in tutorial4

	* logstash
		* input을 입력받아 grok filter 처리 하여 output출력
		* 필터에서 hello 뒤의 이름을 name key에 매칭

## in tutorial5
	* Elasticsearch
		* application/json 타입의 데이터를 localhost:9200/firstindex/_doc에 '{ "mykey": "myvalue" }'로 저장 // secondindex 추가
		* application/x-ndjson'타입의 데이터를 (?? 잘모르겠음)
## in tutorial6
	*Kibana
		* Kibana Management 메뉴 선택 > create index pattern > 인덱스 패턴 정의 > 인덱스 패턴 설정(ex:timestamp) > Discover 메뉴에서 인덱스 time range 설정 > Create a Visualize>Visualize 시각화 타입 선택 (ex: RegionMap) > 시각화할 인덱스 선택 > Kibana Visualize 메뉴에서 시각화 설정 정의 (ex: Metrics_Aggregation=count, Buckets_Aggregation = Terms, Buckets_Field = geo.src.keyword)  > 상단의 시각화 결과 save > 대시보드에 Visualization 추가
