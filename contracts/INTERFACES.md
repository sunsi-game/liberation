# 모듈 간 계약 — 구현 제안 (확정 API가 아님)

## 소유권과 의존성
| 소유자 | 제공하는 계약 | 의존 대상 |
|---|---|---|
| Player | `PlayerStateChanged`, `DamageTaken`, `PlayerDied`, `DashUsed` | Input, Combat |
| Combat | `AttackResolved`, `TargetDefeated`, `AmmoChanged` | Player, Enemy |
| Stealth | `TakedownResolved`, `BodyHidden`, `BodyDiscovered` | Combat, Enemy AI |
| Enemy AI | `GuardAlertChanged`, `GuardDowned` | Perception, Stealth |
| Companion | `CompanionStateChanged`, `CoverAvailable` | Combat, Boss |
| Quest | `QuestObjectiveProgressed`, `QuestReadyToReport`, `QuestReported` | Story, Inventory, World |
| Story | `StoryBeatRequested`, `CutsceneEnded`, `ChapterUnlocked` | Quest/Scene |
| World | `ZoneEntered`, `GateDenied`, `AreaExited` | Quest, Identity, Save |
| Identity | `AccessEvaluated`, `OutfitChanged`, `IdentityGranted` | Quest, Inventory |
| Save | `CheckpointCaptured`, `WorldRestored`, `PersistentItemChanged` | Domain snapshots |
| Boss | `BossPhaseChanged`, `SnipeWindowOpened`, `SnipeAttemptResolved` | Combat, Companion, Story |
| Online | `TradeStateChanged`, `SyncStatusChanged` | Server adapter, Persistent Inventory |
| UI | 구독 및 명령 발행만, 도메인 변경 로직 소유 금지 | 위 이벤트 전반 |

## ID · 데이터 계약 원칙
- 퀘스트·장·구역·씬·아이템·신분·NPC·스토리 이벤트 식별자는 **안정된 문자열 ID 또는 GUID**. `GameObject.name`·문구를 식별자로 사용하지 않는다.
- 게임 데이터 정의: ScriptableObject / 런타임 상태: 직렬화 가능한 C# DTO / 저장 포맷: 버전 명시.
- 모든 보상, 임무 아이템, 열쇠, 챕터 해금은 `transactionId`/`grantId`로 **멱등 처리**.
- 시간은 scaled vs unscaled 명시. `SnipeWindowTimer`는 컷신/일시정지 상태에서 시간 진행 여부가 **TBD**이며 UI 단독 카운트 금지.
- 임무 폭탄과 저격탄은 일반 거래 가능 소모품과 분리된 임무 전용 리소스.

## 최소 저장 레이어
1. `WorldCheckpoint`: 지역/구역/위치/HP/적 상태/임무 로컬 상태/동료/보스 페이즈.
2. `PersistentProgress`: 완료 퀘스트, 보상 이력, 평판, 의상·신분, 챕터 해금.
3. `PersistentConsumables`: 소비 아이템 변동 원장. 체크포인트 복원에서 제외.
4. `MissionResources`: 특수 폭탄/저격탄, 시도별 재지급 정책.
5. `OnlineTradeLedger`: 서버 검증 후 확정된 거래 변경 이력; 로컬 자료와 구분.

## MAIN-016 필수 상태 계약
```text
BossPhase = 1 | 2 | 3 | Defeated
SnipeState = Locked | Ready | Active | Hit | Failed
WindowOpens iff RequiredGuardsDefeated && CompanionCombatCapable
ActiveTime = 4.5s
OnCompanionDownWhileActive: remaining += 2.0s (once per attempt)
OnMissOrTimeout: FailureCutscene -> RestorePhase3Checkpoint -> RestoreMissionSniperAmmo
OnHitValidTargetWhileActive: BossDefeated -> Epilogue (only once)
```
- 다운으로 시간 연장 후에도 해당 저격 기회는 유지; 다운 이전에는 기회 활성화 불가.
- 다운 동료 자동 복귀 시간은 조정 데이터(TBD). 재시도 시 상태 스냅샷과 엄호 시작 순서를 명시적으로 초기화.

## 기능 변경 승인 체크
공유 API 변경 시: 사용처 검색 → 파급 영향 문서화 → 관련 담당 테스트 계획 → 변경 승인 → 코드 변경 → 테스트.
