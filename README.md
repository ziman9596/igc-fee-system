# IGC Samundri: Fee & Records System

A fee and records system for my father's college. I worked in its accounts office as a Data Entry intern and saw the problem up close: one accountant, one Excel ledger, entries skipped because the day ran out of hours.

<!-- ![Screenshot](screenshot.png) -->

**Design decisions**
- Fee vouchers lock after printing, so nothing changes after the fact
- Payments are logged manually, with an automatic timestamp
- Partial payments are logged separately from formal fee-split requests
- Offline-first desktop app with automatic backup, plus a lightweight web portal for students and the director

It's a management tool, not a payment system. It takes error-prone tracking off the accountant's plate, it doesn't change how money is handled.

**Status:** In progress, delivery planned before Nov 1.

*Source code is private (college data).*
