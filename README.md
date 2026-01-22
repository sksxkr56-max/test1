📝 AI 기반 서술형 평가 및 데이터 관리 시스템

이 프로젝트는 Streamlit을 활용하여 학생들의 서술형 답안을 수집하고, OpenAI GPT API를 통해 즉각적인 채점 및 피드백을 제공하며, 결과 데이터를 Supabase 클라우드 DB에 영구 저장하는 교육용 애플리케이션입니다.

📊 시스템 구조 (System Architecture)

이 시스템의 데이터 흐름과 처리 과정을 시각화한 구조도입니다.

graph TD
    A[👨‍🎓 학생] -->|1. 학번 & 답안 입력| B(💻 Streamlit 웹 인터페이스)
    B -->|2. 제출 버튼 클릭| C{입력값 검증}
    C -->|누락 확인| B
    C -->|정상 제출| D[세션 상태 업데이트]
    
    D -->|3. GPT 피드백 요청| E[🤖 OpenAI GPT API]
    E -->|4. 채점 & 피드백 생성| D
    
    D -->|5. 데이터 전송| F[🗄️ Supabase DB]
    F -->|6. 저장 완료 확인| D
    
    D -->|7. 결과 화면 출력| A


🛠️ 주요 기능 (Key Features)

학생용 인터페이스 (Step 1)

학번 입력 및 3개의 서술형 문항 제공.

st.form을 사용하여 답안을 한 번에 제출.

필수 입력값(학번, 빈칸) 검증 로직 포함.

AI 자동 채점 (Step 2)

Prompt Engineering: AI에게 교사의 페르소나를 부여하고, 채점 기준(Guideline)을 기반으로 평가하도록 지시.

정규화 로직: AI의 응답을 'O/X' 형식과 '200자 이내 피드백'으로 자동 보정하여 일관성 유지.

데이터 저장 (Supabase)

채점 결과, 학생 답안, AI 피드백, 사용된 채점 기준을 클라우드 DB에 실시간 저장.

supabase-py 클라이언트를 사용하여 안전하게 연동.

⚙️ 설치 및 설정 (Setup)

1. 라이브러리 설치

프로젝트 실행을 위해 requirements.txt에 다음 패키지들이 필요합니다.

pip install streamlit openai supabase


2. 환경 변수 설정 (.streamlit/secrets.toml)

프로젝트 루트 경로에 .streamlit 폴더를 만들고 secrets.toml 파일을 생성하여 API 키를 설정해야 합니다.

# .streamlit/secrets.toml

OPENAI_API_KEY = "sk-..."

SUPABASE_URL = "[https://your-project-id.supabase.co](https://your-project-id.supabase.co)"
SUPABASE_SERVICE_ROLE_KEY = "eyJ..."


📂 코드 상세 설명

1️⃣ Supabase 클라이언트 초기화

@st.cache_resource
def get_supabase_client() -> Client:
    # secrets에서 URL과 Key를 가져와 클라이언트 생성
    # @st.cache_resource를 사용하여 리소스 낭비 방지
    ...


데이터베이스 연결을 효율적으로 관리하기 위해 Streamlit의 캐싱 기능을 사용합니다.

2️⃣ UI 및 제출 처리 (Step 1-2)

with st.form("submit_form"):
    # 학번 및 문항 1, 2, 3 입력 필드 생성
    submitted = st.form_submit_button("제출")

if submitted:
    # 유효성 검사 후 세션 상태(st.session_state) 업데이트
    # 재제출 시 기존 피드백 초기화 로직 포함


사용자 입력을 받고, 제출 버튼이 눌렸을 때만 로직이 실행되도록 제어합니다.

3️⃣ AI 채점 및 DB 저장 (Step 2)

if st.button("GPT 피드백 확인", disabled=not st.session_state.submitted_ok):
    # OpenAI API 호출
    response = client.chat.completions.create(...)
    
    # 응답 데이터 구조화 (JSON/Dict)
    st.session_state.gpt_payload = { ... }
    
    # Supabase 저장 함수 호출
    save_to_supabase(st.session_state.gpt_payload)


채점 로직: 각 문항별로 반복문(for)을 돌며 API를 호출합니다.

데이터 구조화: 저장할 데이터를 Dictionary 형태로 묶어 DB 스키마와 일치시킵니다.

4️⃣ 응답 후처리 (Normalization)

def normalize_feedback(text: str) -> str:
    # AI 응답이 'O:' 또는 'X:'로 시작하도록 강제 보정
    # 피드백 길이가 200자를 넘지 않도록 자름


AI가 가끔 형식을 어길 경우를 대비한 방어 코드(Guardrail)입니다.

🚀 실행 방법

터미널에서 다음 명령어를 실행하여 앱을 시작합니다.

streamlit run app.py


Note: 코드 내 gpt-5-mini 모델명은 예시일 수 있으며, 실제 사용 시에는 사용 가능한 모델명(예: gpt-4o-mini, gpt-3.5-turbo)으로 확인 후 변경이 필요할 수 있습니다.
