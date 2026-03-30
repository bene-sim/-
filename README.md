<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>드론 GCS 개발 구조 정리</title>
  <style>
    :root {
      --bg: #f4f7fb;
      --card: #ffffff;
      --line: #d9e2f0;
      --text: #1f2a37;
      --sub: #526071;
      --accent: #1f6feb;
      --accent-soft: #eaf2ff;
      --title: #0f172a;
    }

    * { box-sizing: border-box; }

    body {
      margin: 0;
      font-family: "Segoe UI", "Apple SD Gothic Neo", "Malgun Gothic", sans-serif;
      background: var(--bg);
      color: var(--text);
      line-height: 1.7;
    }

    .wrap {
      max-width: 1180px;
      margin: 0 auto;
      padding: 40px 20px 80px;
    }

    .hero {
      background: linear-gradient(135deg, #0f172a, #1e3a8a);
      color: #fff;
      border-radius: 24px;
      padding: 40px 32px;
      box-shadow: 0 14px 40px rgba(15, 23, 42, 0.18);
      margin-bottom: 28px;
    }

    .hero h1 {
      margin: 0 0 12px;
      font-size: 2.2rem;
      line-height: 1.25;
    }

    .hero p {
      margin: 0;
      font-size: 1.02rem;
      color: rgba(255,255,255,0.9);
    }

    .toc, .section, .summary-box {
      background: var(--card);
      border: 1px solid var(--line);
      border-radius: 20px;
      padding: 28px;
      box-shadow: 0 8px 24px rgba(15, 23, 42, 0.05);
      margin-bottom: 22px;
    }

    .toc h2, .section h2, .summary-box h2 {
      margin-top: 0;
      margin-bottom: 16px;
      color: var(--title);
      font-size: 1.55rem;
    }

    .toc ul {
      margin: 0;
      padding-left: 20px;
    }

    .toc li { margin: 8px 0; }

    .section h3 {
      margin-top: 28px;
      margin-bottom: 10px;
      font-size: 1.15rem;
      color: var(--accent);
    }

    .section p { margin: 10px 0; }

    .badge {
      display: inline-block;
      padding: 6px 12px;
      border-radius: 999px;
      background: var(--accent-soft);
      color: var(--accent);
      font-weight: 700;
      font-size: 0.9rem;
      margin-bottom: 14px;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
      gap: 16px;
      margin-top: 16px;
    }

    .mini-card {
      background: #f8fbff;
      border: 1px solid var(--line);
      border-radius: 16px;
      padding: 18px;
    }

    .mini-card h4 {
      margin: 0 0 8px;
      color: var(--title);
      font-size: 1rem;
    }

    .mini-card p {
      margin: 0;
      color: var(--sub);
      font-size: 0.95rem;
    }

    .flow {
      background: #0f172a;
      color: #e5eefc;
      border-radius: 16px;
      padding: 18px 20px;
      overflow-x: auto;
      font-family: Consolas, Monaco, monospace;
      font-size: 0.95rem;
      line-height: 1.8;
      margin: 16px 0;
      white-space: pre;
    }

    .point {
      border-left: 4px solid var(--accent);
      background: #f8fbff;
      padding: 14px 16px;
      border-radius: 12px;
      margin: 14px 0;
    }

    .point strong {
      color: var(--title);
    }

    .compare {
      width: 100%;
      border-collapse: collapse;
      margin-top: 16px;
      overflow: hidden;
      border-radius: 14px;
      background: #fff;
    }

    .compare th,
    .compare td {
      border: 1px solid var(--line);
      padding: 14px 12px;
      text-align: left;
      vertical-align: top;
    }

    .compare th {
      background: #eef4ff;
      color: var(--title);
      font-weight: 700;
    }

    .footer-note {
      margin-top: 10px;
      color: var(--sub);
      font-size: 0.95rem;
    }

    @media (max-width: 768px) {
      .hero h1 { font-size: 1.7rem; }
      .toc, .section, .summary-box { padding: 22px 18px; }
    }
  </style>
</head>
<body>
  <div class="wrap">
    <section class="hero">
      <h1>드론 GCS 개발 구조 정리</h1>
      <p>
        본 문서는 드론 GCS 개발 시 적용된 주요 구조와 개발 관점을 기준으로
        <strong>MVVM</strong>, <strong>CBD</strong>, <strong>C4 기반 아키텍처(C1~C3)</strong>,
        <strong>DB 대신 DTO 사용</strong>의 4가지 항목을 구분하여 정리한 자료입니다.
      </p>
    </section>

    <section class="toc">
      <h2>목차</h2>
      <ul>
        <li>1. MVVM (Model - View - ViewModel)</li>
        <li>2. CBD (Component-Based Development)</li>
        <li>3. C4 기반 아키텍처 (C1 ~ C3)</li>
        <li>4. DB 대신 DTO 사용</li>
        <li>5. 전체 관계 정리</li>
      </ul>
    </section>

    <section class="section">
      <div class="badge">1. MVVM</div>
      <h2>MVVM (Model - View - ViewModel)</h2>
      <p>
        MVVM은 화면, 화면과 연결되는 처리 로직, 데이터를 분리하는 아키텍처 패턴입니다.
        드론 GCS처럼 UI가 복잡하고 실시간 상태 반영이 중요한 시스템에서 자주 활용되는 구조입니다.
      </p>

      <div class="flow">View  ↔  ViewModel  ↔  Model</div>

      <h3>1) View</h3>
      <p>
        View는 사용자가 직접 보는 화면입니다. 지도, 드론 상태 패널, 배터리/속도/고도 표시,
        이륙·착륙·복귀 버튼, 영상 표출 영역, 경고 팝업 등이 여기에 해당합니다.
        View의 핵심 역할은 <strong>보여주기와 입력 받기</strong>입니다.
      </p>

      <h3>2) ViewModel</h3>
      <p>
        ViewModel은 View와 Model 사이의 중간 계층으로, 화면에서 발생한 이벤트를 실제 시스템 동작으로 연결하고,
        Model의 데이터를 화면에 맞는 형태로 가공합니다. 예를 들어 사용자가 이륙 버튼을 클릭하면 ViewModel이 이를 받아
        이륙 명령을 생성하고 통신 모듈로 전달합니다. 반대로 드론 상태가 수신되면 UI 표시 형태로 정리하여 View에 전달합니다.
      </p>

      <h3>3) Model</h3>
      <p>
        Model은 실제 데이터 자체를 의미합니다. 드론 위치, 속도, 자세값, 배터리 상태, 임무 상태,
        센서 데이터, 통신 메시지 구조 등이 Model에 해당합니다.
        즉, Model은 <strong>실제 상태와 데이터의 원본</strong>입니다.
      </p>

      <div class="grid">
        <div class="mini-card">
          <h4>MVVM의 장점 1</h4>
          <p>UI와 로직이 분리되어 화면 수정이 로직 전체에 영향을 주지 않습니다.</p>
        </div>
        <div class="mini-card">
          <h4>MVVM의 장점 2</h4>
          <p>실시간 드론 상태를 UI에 반영하는 흐름이 자연스럽게 구성됩니다.</p>
        </div>
        <div class="mini-card">
          <h4>MVVM의 장점 3</h4>
          <p>ViewModel은 UI 없이도 테스트 가능하여 유지보수성과 검증성이 좋아집니다.</p>
        </div>
      </div>

      <div class="point">
        <strong>GCS 관점 핵심:</strong>
        MVVM은 드론 GCS의 복잡한 화면을 안정적으로 운영하기 위해,
        화면과 데이터 처리 로직을 분리하는 구조라고 볼 수 있습니다.
      </div>
    </section>

    <section class="section">
      <div class="badge">2. CBD</div>
      <h2>CBD (Component-Based Development)</h2>
      <p>
        CBD는 소프트웨어를 하나의 큰 덩어리로 만들지 않고,
        기능별 컴포넌트 단위로 나누어 개발하는 방식입니다.
        드론 GCS는 통신, 제어, 영상, 지도, 임무, 경고 처리 등 다양한 기능이 함께 동작하므로,
        CBD 방식이 매우 잘 맞는 구조입니다.
      </p>

      <div class="flow">Communication Component
Telemetry Component
Mission Component
Video Component
Map Component
Alert Component
Control Component</div>

      <h3>1) 기능 분리</h3>
      <p>
        예를 들어 드론 상태 수신은 Telemetry Component,
        드론 명령 생성과 송신은 Control/Communication Component,
        영상 표출은 Video Component,
        지도와 위치 표시는 Map Component로 나눌 수 있습니다.
      </p>

      <h3>2) 유지보수 용이성</h3>
      <p>
        통신 방식이 바뀌면 Communication Component를, 영상 방식이 바뀌면 Video Component를 중심으로 수정하면 됩니다.
        즉, 하나의 기능 변경이 시스템 전체 변경으로 번지는 것을 줄일 수 있습니다.
      </p>

      <h3>3) 병렬 개발 가능</h3>
      <p>
        개발자별로 통신, UI, 임무, 영상, 지도 등을 나눠 동시에 개발할 수 있습니다.
        이는 대형 GCS 프로젝트에서 일정 관리와 역할 분담 측면에서 매우 큰 장점입니다.
      </p>

      <h3>4) 재사용성 확보</h3>
      <p>
        특정 통신 컴포넌트나 텔레메트리 처리 컴포넌트는 다른 드론 프로젝트에서도 다시 사용할 수 있습니다.
        따라서 CBD는 단순한 분리 방식이 아니라, 재사용성과 확장성을 높이는 개발 방식이기도 합니다.
      </p>

      <div class="point">
        <strong>GCS 관점 핵심:</strong>
        CBD는 드론 GCS를 통신, 제어, 영상, 지도, 임무 등 기능별로 독립 구성하여,
        변경 대응과 확장성을 높이는 방식입니다.
      </div>
    </section>

    <section class="section">
      <div class="badge">3. C4 기반 아키텍처</div>
      <h2>C4 기반 아키텍처 (C1 ~ C3)</h2>
      <p>
        C4는 코드를 작성하는 기술이 아니라,
        시스템 구조를 단계적으로 설명하고 이해하기 위한 아키텍처 표현 방식입니다.
        일반적으로 C1(Context), C2(Container), C3(Component), C4(Code) 수준으로 구분하지만,
        실무에서는 코드 변동성이 크기 때문에 주로 C1~C3까지만 활용하는 경우가 많습니다.
      </p>

      <h3>1) C1: Context</h3>
      <p>
        C1은 시스템을 가장 바깥 관점에서 바라봅니다.
        즉, 드론 GCS가 외부의 어떤 요소들과 연결되는지 설명하는 단계입니다.
      </p>
      <div class="flow">[관제사] ↔ [GCS] ↔ [드론]
                ↕
         [외부 서버 / AI / 영상 시스템]</div>
      <p>
        이 단계에서는 GCS가 전체 체계에서 어떤 위치에 있는지,
        어떤 외부 시스템과 데이터를 주고받는지 큰 그림으로 설명합니다.
      </p>

      <h3>2) C2: Container</h3>
      <p>
        C2는 GCS 내부를 큰 기능 단위로 나누는 단계입니다.
        예를 들면 UI Container, Communication Container, Mission/Control Container,
        Video Container, Logging Container 등으로 구분할 수 있습니다.
      </p>
      <p>
        여기서 Container는 Docker 같은 기술 명칭이 아니라,
        시스템 내부의 큰 실행 단위 또는 기능 블록으로 이해하면 됩니다.
      </p>

      <h3>3) C3: Component</h3>
      <p>
        C3는 각 Container 내부를 더 세부 기능으로 나누는 단계입니다.
        예를 들어 Communication Container 내부에는 MAVLink Parser, UDP Sender,
        UDP Receiver, Message Dispatcher, Telemetry Handler 등이 있을 수 있습니다.
      </p>
      <p>
        Mission/Control Container 내부에는 Mission Manager, Route Planner,
        Drone Command Builder, State Evaluator 같은 세부 구성 요소를 둘 수 있습니다.
      </p>

      <h3>4) 왜 C4(Code)는 유지하지 않는가</h3>
      <p>
        실제 개발이 진행되면 코드 구조는 계속 바뀝니다.
        클래스가 추가되고, 모듈이 분리되고, 통신 구조와 UI 로직이 변경되므로,
        코드 수준 문서를 끝까지 맞춰 유지하는 것은 관리 비용이 큽니다.
      </p>
      <p>
        그래서 실무에서는 C4를 주로 초기 설계 설명, 고객 보고, 구조 공유,
        신규 인력 온보딩 등에 활용하고, 운영 단계에서는 C1~C3 수준까지만 유지하는 경우가 많습니다.
      </p>

      <div class="point">
        <strong>GCS 관점 핵심:</strong>
        C4는 드론 GCS의 구조를 단계적으로 설명하기 위한 기준이며,
        실무에서는 주로 C1~C3까지만 관리하는 경우가 많습니다.
      </div>
    </section>

    <section class="section">
      <div class="badge">4. DTO 중심 구조</div>
      <h2>DB는 사용하지 않고 DTO 사용</h2>
      <p>
        이 표현은 보통 데이터를 장기 저장하는 DB 중심 구조가 아니라,
        <strong>실시간 전달 중심 구조</strong>로 설계했다는 의미에 가깝습니다.
        DTO(Data Transfer Object)는 데이터를 한 모듈에서 다른 모듈로 전달하기 위한 단순 객체입니다.
      </p>

      <div class="flow">[Drone] → [Communication] → [DTO] → [ViewModel] → [View]</div>

      <h3>1) DTO의 역할</h3>
      <p>
        DTO는 복잡한 비즈니스 로직을 담기보다,
        위치, 속도, 배터리, 자세값, 임무 상태 같은 데이터를 담아 다른 계층으로 전달하는 역할을 합니다.
      </p>

      <h3>2) DB를 쓰지 않는 이유</h3>
      <p>
        드론 GCS는 실시간성이 핵심입니다.
        드론 위치, 링크 상태, 배터리, 경고 정보는 즉시 보여주고 즉시 반응해야 합니다.
        이 경우 DB에 저장하고 다시 읽는 구조보다, DTO로 바로 전달하는 구조가 더 빠르고 단순합니다.
      </p>

      <h3>3) 스트리밍 데이터에 적합</h3>
      <p>
        텔레메트리 데이터는 한 번 저장해서 오래 보는 데이터라기보다,
        계속 들어오고 계속 갱신되는 데이터입니다.
        따라서 저장보다 전달과 반영이 중요한 구조이며, DTO 기반 흐름이 자연스럽습니다.
      </p>

      <h3>4) 구조 단순화</h3>
      <p>
        DB가 들어가면 스키마, 저장 로직, 조회 로직, 동기화, 트랜잭션 관리 등 추가 요소가 필요합니다.
        반면 DTO 중심 구조는 모듈 간 전달 구조만 명확하면 되므로,
        실시간 제어 시스템에 더 간결하게 적용할 수 있습니다.
      </p>

      <h3>5) 실무적 해석</h3>
      <p>
        다만 “DB를 사용하지 않는다”는 것이 시스템 전체에 기록이 전혀 없다는 뜻은 아닐 수 있습니다.
        실제로는 실시간 제어 경로는 DTO로 빠르게 처리하고,
        이력 관리나 이벤트 로그는 별도 파일 또는 저장 구조로 관리할 수 있습니다.
      </p>

      <div class="point">
        <strong>GCS 관점 핵심:</strong>
        DB 대신 DTO를 사용했다는 것은 드론 GCS를 저장 중심이 아니라,
        실시간 데이터 전달 중심으로 설계했다는 뜻입니다.
      </div>
    </section>

    <section class="summary-box">
      <h2>5. 전체 관계 정리</h2>
      <p>
        위 4가지 개념은 각각 독립적인 주제이지만, 실제 GCS 개발에서는 서로 연결되어 동작합니다.
      </p>

      <table class="compare">
        <thead>
          <tr>
            <th>구분</th>
            <th>의미</th>
            <th>드론 GCS 적용 관점</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>MVVM</td>
            <td>화면, 처리 로직, 데이터를 분리하는 UI 아키텍처</td>
            <td>복잡한 화면과 실시간 상태 반영을 안정적으로 관리</td>
          </tr>
          <tr>
            <td>CBD</td>
            <td>기능을 컴포넌트 단위로 나누어 개발하는 방식</td>
            <td>통신, 제어, 영상, 지도, 임무 기능을 독립적으로 구성</td>
          </tr>
          <tr>
            <td>C4 (C1~C3)</td>
            <td>시스템 구조를 단계별로 설명하는 방식</td>
            <td>초기 설계, 구조 설명, 고객 보고, 팀 커뮤니케이션에 활용</td>
          </tr>
          <tr>
            <td>DTO</td>
            <td>데이터 전달을 위한 단순 객체</td>
            <td>DB 저장보다 실시간 전달과 반영 중심으로 데이터 처리</td>
          </tr>
        </tbody>
      </table>

      <div class="flow">[C4]
전체 구조를 C1~C3 수준에서 설명

[CBD]
통신 / 제어 / 영상 / 지도 / 임무를 기능별 컴포넌트로 분리

[MVVM]
UI 내부는 View / ViewModel / Model로 분리

[DTO]
각 계층과 컴포넌트 사이의 데이터 전달에 사용</div>

      <p class="footer-note">
        정리하면, 본 GCS 구조는 <strong>기능 분리(CBD)</strong>, <strong>UI와 로직 분리(MVVM)</strong>,
        <strong>구조 설명 체계(C4)</strong>, <strong>실시간 데이터 전달(DTO)</strong>의 관점으로 이해할 수 있습니다.
      </p>
    </section>
  </div>
</body>
</html>
