# 예시 요청 — MAIN-016 3페이즈 저격

`AGENTS.md`, `DECISIONS.md`, `agents/05_companion.md`, `agents/02_combat.md`, `contracts/INTERFACES.md`, 5장 최신 확정 보스전 명세를 먼저 읽어줘. 이번 변경 대상은 Boss 도메인만으로 한정하고, 동료 관련 조건은 기존 `CompanionCombatCapable` 및 `CompanionDowned` 계약을 호출해. 지정 경비병 처치 + 동료 행동 가능일 때 4.5초 저격 창을 열고, 창 활성화 이후 동료가 다운되면 2초를 1회 연장해. 빗나감/시간 초과 시 실패 이벤트를 발생시키고 컷신 이후 페이즈3 시작 체크포인트를 복원하며 임무 전용 저격탄을 되돌려. 성공 시 단 한 번 보스 처치 이벤트를 발행해. 일반 소비 아이템을 복원하지 마. EditMode 단위 테스트와 PlayMode 확인 경로를 보고해.
