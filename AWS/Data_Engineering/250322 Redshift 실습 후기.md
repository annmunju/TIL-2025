- Amazon Redshift는 빠른 완전관리형 [데이터 웨어하우스](https://aws.amazon.com/data-warehouse/)로서 표준 SQL과 기존 비즈니스 인텔리전스(BI) 도구를 사용하여 모든 데이터를 간편하고 비용 효과적으로 분석할 수 있습니다.

### Introduction to Amazon Redshift
1. Redshift 에서 클러스터 만들기
	- VPC 설정 및 노드 개수 지정. IAM 규칙 설정 (실습에서는 s3 버킷에서 txt 파일 읽는게 포함되어 있어서 해당 권한이 추가됨)
	- Security groups도 인바운드 규칙(redshift, 동일 vpc 내)이 추가되어 있는 것으로 사용
2. 쿼리 편집기를 통해 테이블 생성 및 샘플 데이터 로드
	- s3 버킷에 있는 text 파일을 사용
```SQL
COPY users FROM 's3://awssampledbuswest2/tickit/allusers_pipe.txt'
CREDENTIALS 'aws_iam_role=YOUR-ROLE'
DELIMITER '|';
```
3. 필요한 쿼리 요청

### Building with Amazon Redshift clusters
- Amazon Redshift는 대용량 병렬 처리와 열 기반 데이터 스토리지, 그리고 매우 효율적인 데이터 압축 인코딩 방식을 조합하여 스토리지 효율성과 쿼리 성능 최적화를 구현
- 행을 테이블에 추가할 때 데이터 값의 열에 적용되는 압축 유형은 압축 인코딩에 따라 결정
