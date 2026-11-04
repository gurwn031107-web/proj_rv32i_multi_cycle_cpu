# 🖥️ RV32I Multi-Cycle CPU + APB Peripherals + UVM Verification

> SystemVerilog로 설계한 **RISC-V RV32I 멀티사이클 CPU**와 **APB 버스 주변장치**, 그리고 R/I-Type 명령어 **UVM 랜덤 검증 환경**

| 항목 | 내용 |
|---|---|
| 기간 | 2026.09 |
| 형태 | 팀 프로젝트 (3인) |
| 언어 / 도구 | SystemVerilog, UVM 1.2, Synopsys VCS, Verdi |
| 담당 | **UVM 검증 환경 설계 (`tb/tb_rv32i_uvm.sv`)**, datapath 수정 |

<br>

## 📌 구조

```
         ┌──────────────┐   instr    ┌──────────────────────────────┐
         │ Instruction  │──────────▶│        rv32i_cpu              │
         │     ROM      │◀──────────│  control_unit (FSM)           │
         └──────────────┘   addr     │  datapath (RegFile, ALU, PC)  │
                                     └──────────────┬───────────────┘
                                         bus (addr / wdata / we)
                                     ┌──────────────▼───────────────┐
                                     │  apb_requester (APB Master)  │
                                     └──┬──────────┬──────────┬─────┘
                                   PSEL0│     PSEL1│     PSEL2│
                                  ┌─────▼──┐ ┌─────▼──┐ ┌─────▼──┐
                                  │ BRAM   │ │  GPI   │ │  GPO   │
                                  └────────┘ └────────┘ └────────┘
```

- **Multi-cycle FSM**: `FETCH → DECODE → EXECUTE → MEMORY → WB` (명령어 타입별로 필요한 단계만 거침)
- **APB Master**: CPU의 Load/Store를 APB 트랜잭션(SETUP → ACCESS)으로 변환, 주소로 슬레이브 선택
- **주변장치**: APB BRAM, GPI(스위치 입력), GPO(LED 출력)

<br>

## ✅ UVM 검증 (담당)

R-Type 10개 + I-Type 9개, 총 **19개 명령어**를 대상으로 한 UVM 랜덤 검증 환경

| 구성 요소 | 역할 |
|---|---|
| Sequence | 명령어당 고르게 분배된 **80,000개 테스트**, 명령어별 처음 6회는 **directed edge case** |
| Driver | 명령어 인코딩 생성, 레지스터에 피연산자 세팅 후 실행 |
| Monitor | Register File write-back 시점의 결과 수집 |
| Scoreboard | 참조 모델과 비교 → 명령어별 PASS / FAIL / 통과율 리포트 |
| Coverage | opcode, rs1/rs2 경계값(0, 1, -1, MIN, MAX), imm, shamt, **opcode × edge cross** |

- Edge case: `0`, `1`, `0xFFFFFFFF`, `0x80000000`, `0x7FFFFFFF`, shamt `0/1/15/16/31` 등 → 19개 명령어 × 6 = **114개 edge 시나리오**

<br>

## 📁 폴더 구조

```
rtl/
├── rv32i_cpu.sv        # Top (CPU + ROM + APB + 주변장치)
├── control_unit.sv     # Multi-cycle FSM
├── datapath.sv
├── apb_requester.sv    # APB Master
├── apb_bram.sv  gpi_ctr.sv  gpo_ctr.sv
├── instruction_rom.sv  rom_code_*.mem
└── pkg/rv32i_pkg.sv
tb/
├── tb_rv32i.sv         # 기본 테스트벤치
└── tb_rv32i_uvm.sv     # UVM 검증 환경
Makefile                # VCS 컴파일 / 시뮬레이션 / Verdi
```

<br>

## ▶️ 실행

```bash
make run     # VCS 컴파일 (UVM 1.2, 커버리지 옵션 포함)
make simv    # 시뮬레이션
make verdi   # 파형 확인
```

<br>

## 👥 Team

- [@jeongmin-0113](https://github.com/jeongmin-0113)
- [@hongs0396-max](https://github.com/hongs0396-max)
- [@gurwn031107-web](https://github.com/gurwn031107-web)
