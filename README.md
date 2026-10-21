# Salon Assistant — Luna

A Python salon booking assistant built for the Week 16 assignment.

## Project

Luna helps customers browse services and available slots, book appointments,
reschedule bookings, and cancel appointments. Python code validates salon
rules and customer confirmation before any booking change.

## Main file

- `notebooks/HW_Week16_SalonAssistant.ipynb` — implementation, tests, live-model experiments, and report.

## Project Progress
- Created the `salon assistant` GitHub repository under `NazaninNosrati`. The project files include `.gitignore`, `README`, `requirements.txt`, `given.ipynb`, and `salon assistant.ipynb`.
- Ran `given.ipynb` and observed `Salon backend ready`, `NOW = 2026-10-19 10:00` (Monday), and `22 public scenarios loaded`.
- Started implementing `cancellation_fee_cents` based on the salon’s cancellation policy.
- Documented the conservative `interpret_reply` design: only a clear, full-string `yes` or `no` is accepted; ambiguous input returns `unclear`.
- Defined the `TOOL_SCHEMAS` design for seven allowed tools. The controller validates tool names and arguments, rejects unknown or invalid calls, and retains control over sensitive data and execution approval.
- Test results for these functions and schemas have not yet been recorded.
