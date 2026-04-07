# JD 파서 모듈

채용 공고를 JSON으로 파싱하는 모듈이다.
텍스트 입력, 이미지 업로드, URL 크롤링 3가지 입력 방식을 지원한다.
   

---


## ① 설치

```bash
# 필수
pip install requests

# 이미지 파싱 또는 URL 크롤링을 사용할 경우
pip install openai playwright
playwright install chromium
```


   


## ② 입력 방식


### 1. 텍스트 직접 입력 (기본)

사용자가 채용 공고 텍스트를 복사해서 붙여넣는 방식이다.
```python
from agent.parsers.jd_parser_free import parse_jd_from_text

result = parse_jd_from_text("채용 공고 텍스트...")
print(result)
```

   
### 2. 이미지 업로드

채용 공고 스크린샷을 GPT-4o Vision 또는 Gemini로 텍스트 변환 후 파싱한다.
잡코리아, 사람인 등 이미지 기반 채용 공고에 사용한다.

```python
from agent.parsers.jd_parser_free import parse_jd_from_image

result = parse_jd_from_image("screenshot.png")
```
   
처리 흐름:

```
이미지 업로드
  -> Vision AI로 텍스트 추출
  -> JD 파싱 프롬프트로 JSON 추출
```
   

### 3. URL 크롤링

Playwright로 채용 공고 페이지를 크롤링하여 파싱한다.
"상세 정보 더 보기" 같은 동적 로딩 콘텐츠도 자동으로 처리한다.

```python
from agent.parsers.jd_parser_free import parse_jd_from_url

result = parse_jd_from_url("https://www.wanted.co.kr/wd/328573")
```
   
처리 흐름:

```
URL 입력
  -> Playwright로 페이지 열기
  -> "더 보기" 버튼 자동 클릭
  -> 텍스트 추출
  -> JD 파싱 프롬프트로 JSON 추출
```

지원 사이트: 원티드, 잡코리아, 사람인, 프로그래머스
   

---


## ③출력 형식

모든 입력 방식에서 동일한 JSON 형식으로 출력된다.

```json
{
  "job_title": "Backend 개발자",
  "company": "콘텐츠웨이브(wavve)",
  "required_skills": ["Java", "AWS", "K8S"],
  "preferred_skills": ["MSA 설계 경험", "대용량 트래픽 처리"],
  "soft_skills": ["커뮤니케이션 스킬", "협업 능력", "주인의식"],
  "experience_years": "3년 이상",
  "main_tasks": ["서비스 API 설계 및 개발"],
  "interview_keywords": ["API 설계", "클라우드 운영", "대용량 트래픽"]
}
```
   
각 필드 설명:

- required_skills : 자격요건에 해당하는 기술 스택, 도구, 프레임워크
- preferred_skills : 우대사항에 해당하는 기술 스택, 도구, 프레임워크
- soft_skills : 인성, 태도, 협업 관련 항목 (면접 인성 질문 생성에 활용)
- interview_keywords : 기술 + 인성을 종합하여 면접에서 검증할 키워드를 추론


---


## ④ 에러 핸들링

파싱 결과는 3가지 상태로 반환된다.

   
### 1 ) 성공 (success)

모든 필수 항목이 정상적으로 파싱된 경우
   

### 2 ) 부분 성공 (partial)

일부 항목이 누락된 경우
사용자에게 부족한 부분을 텍스트로 직접 입력하라고 안내한다.

```
채용 공고를 파싱했지만, 일부 항목(자격요건, 우대사항)을 찾지 못했습니다.
다음 항목을 텍스트로 직접 입력해주시면 더 정확한 면접 질문을 생성할 수 있습니다.
```
   

### 3 ) 실패 (failed)

파싱이 불가능한 경우
입력 방식에 따라 다른 대체 방법을 안내한다.

- URL 실패시: "스크린샷으로 찍어서 이미지로 업로드" 또는 "텍스트로 붙여넣기" 안내
- 이미지 실패시: "더 선명하게 다시 캡처" 또는 "텍스트로 직접 입력" 안내
- 텍스트 실패시: "주요업무, 자격요건, 우대사항이 포함되어 있는지 확인" 안내

   
---


## CLI 테스트 (무료)


### 사전 준비

1. https://aistudio.google.com/apikey 에서 Gemini API 키 발급 (무료)

2. API 키를 환경변수로 설정

```bash
# Windows PowerShell
$env:GEMINI_API_KEY = "your-key"

# Mac / Linux
export GEMINI_API_KEY=your-key
```

3. 필요한 패키지 설치

```bash
pip install requests
```

   
### 테스트 실행

```bash
# 텍스트 입력 테스트
python jd_parser_free.py text

# 이미지 파싱 테스트
python jd_parser_free.py image screenshot.png

# URL 크롤링 테스트 
pip install playwright
playwright install chromium
python jd_parser_free.py url https://www.wanted.co.kr/wd/328573
```
   

---


## 테스트 결과 (프롬프트 v2)


### ① wavve Backend 개발자

- required_skills: Go, Java, Kotlin, Node.js 등 11개 -- 정상
- preferred_skills: MSA, 대용량 트랜잭션, Kafka 등 7개 -- 정상
- soft_skills: 빈 배열 (공고에 인성 항목 없음) -- 정상

   
### ② 배민 Server(푸드주문시스템)

- required_skills: Java, Kotlin, Spring Framework 등 6개 -- 정상
- preferred_skills: AWS, Kafka, Redis, Elasticsearch 등 14개 -- 정상
- soft_skills: 커뮤니케이션, 주인의식, 협업 능력 등 9개 -- 정상
- 인성/기술 분리 정상 동작 확인
   

---


## 파일 구조

```
agent/parsers/
  README.md              -- 파일 설명
  jd_parser_prompt.py    -- 프롬프트 v2 (System Prompt + 이미지 변환 프롬프트)
  jd_parser.py           -- 메인 파서 (OpenAI 전용 코드)
  jd_parser_free.py      -- 무료 테스트 버전 (Gemini 전용 + 에러 핸들링 코드 추가)
```
   

---


## 다음 단계

- 이력서 파싱 프롬프트 개발 (resume_parser_prompt.py)
- 답변 평가 프롬프트 개발 (answer_evaluator_prompt.py)
- 면접관 페르소나 프롬프트 개발 (persona_*.py)
