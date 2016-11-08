> 2016년도 스타랩 성과물 작성을 위한 성능평가 기준

## 전체 개요

* Sysbench, Linkbench, BM Factory, SQL trace, TPC-C 등의 데이터베이스 벤치마크를 수행했을 때 나오는 결과값들을 정리하여야 함
* 모든 테스트 수행시 iostat과 blktrace 결과를 반드시 수집하여야 함
	* 필요에 따라서 perf, vtune, valgrind와 같은 트레이스 툴의 결과도 저장

### 1. 트랜잭션 처리량 

일반적인 벤치마크 수행시 결과값으로 나오는 Transactions Per Second (TPS), Transactions Per Minute (TPM), Operations Per Second (OPS) 등을 기록

자신이 수정하기 이전의 원본 데이터베이스의 성능을 N, 자신이 수정한 데이터베이스의 성능을 M이라고 가정

표기하여야할 수치는 M/N 을 100분율로 퍼센티지를 표시 (성능 증가량)

### 2. Flash Cache Hit Ratio

Bcache 작업하는 종백이와, FaCE 작업하는 미진이한테만 해당
실제 해당 cache의 hit ratio를 측정

### 3. IOPS
벤치마크 수행시 수집한 iostat 결과의 read/write iops의 평균값을 측정
트랜잭션 처리량과 동일하게 N (오리지널 성능) / M (내 성능)을 백분률로 표기

### 4. Write Amplification Factor (WAF)
Write양을 줄이는 실험을 하는 연구자들에게만 해당
- 미진: FaCE
- 종백: Hadoop 최적화 중 write cache 연구를 진행할때
- 영석: 향후 연구 중 write optimization 시
- Dat: WAF inside MS-SSDs
- 황교: MS-SSD 실험시 내부 WAF, 향후 IPL 진행시 WAF 
- 종혁: 현재 실험하고 있는 SQLite FSL 실험결과 그대로 리포트
- 주보: 종혁이 측정한 방식 물어보고 따라서 측정
- 다솜: Read optimization 이므로 해당하지 않음
- 재훈: 해당 없음
- 민지: FTL 실험 수행시 얻어냈던 copy count와 GC count 측정

**측정 방식 (Application Level WAF)**
1. 데이터베이스 코드 상에서 pwrite()와 같은 write를 유발하는 함수상에서 실제로 disk에 write 요청하는 데이터의 크기를 측정
2. 벤치마크 수행시 수집한 blktrace파일에서 실제 데이터 write 양을 계산
3. blktrace 결과 / 데이터베이스 write 양의 값을 WAF로 설정
4. 오리지널 코드의 WAF를 N, 내 코드의 WAF를 M 이라고 가정
5. M/N 의 값을 백분률로 표기 (M 과 N 위치가 트랜잭션 처리량과 반대임)

**측정 방식 2 (SSD Level WAF)**
1. 벤치마크 수행시 수집한 blktrace 를 통해 실제 데이터 write 양을 게산
2. smartctl 커맨드를 통해 SSD 내부 write 양 측정
```bash
# 실험 수행전
$> sudo smartctl -A /dev/내장치 &> before.smartctl
# 벤치마크 수행
# ....
# 벤치마크 수행후 
$> sudo smartctl -A /dev/내장치 &> after.smartctl
```
3. Total_LBAs_Written으로 표기된 수치의 차이값 계산 (Write 양)
4. Wear_Leveling_Count 로 표기된 수치의 차이값 계산 (GC 양)
5. 측정한 Total_LBAs_Written / blktrace write 양의 값 계산 => SSD Internal WAF
5. 오리지널 데이터베이스와 내가 수정한 데이터베이스에 대해 모두 측정
6. Application level WAF 와 동일하게 M/N의 값을 백분률로 표기
> Total_LBAs_Written과 Wear_leveling_Count의 경우 표기 이름이나 방식이 제조사나 SSD 종류별로 다를 수 있으므로 제조사의 white paper나 SSD의 스펙 문서를 확인하여야 함

### 5. 스토리지 읽기 성능/쓰기 성능

1. 벤치마크 수행시 수집한 iostat 결과 내의 read bandwidth의 평균값을 측정
2. 오리지널 데이터베이스와 내 데이터베이스의 차이를 백분률로 비교
3. 쓰기 성능도 동일하게 write bandwidth의 평균값으로 측정

### 6. IO 지연시간 (IO Latency)
특정 벤치마크의 경우 결과값으로 latency 정보를 제공해주기도 함. 그런 경우는 해당 latency 정보를 측정. 
그렇지 않은 경우 [IOPS를 측정](#3. IOPS)하여 1/IOPS 로 계산
