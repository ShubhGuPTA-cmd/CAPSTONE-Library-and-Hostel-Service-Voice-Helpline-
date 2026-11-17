# CampusCare Voice Desk

**Capstone Project 26 --- Team 26**

CampusCare Voice Desk is a student-services conversational assistant
designed to support students through spoken telephony and digital chat.
It provides a single point of contact for common **library and hostel
services**, using a welcome-desk agent, a specialist service agent,
simulated backend tools, and safety guardrails.

> **Project status:** Demonstrated as a multi-agent prototype in CX
> Agent Studio. The backend data is simulated, and the reported
> evaluation evidence is from the text channel---not confirmed
> end-to-end telephony testing.

------------------------------------------------------------------------

## Table of Contents

-   [Overview](#overview)
-   [Key Features](#key-features)
-   [System Architecture](#system-architecture)
-   [Supported Services and Business
    Rules](#supported-services-and-business-rules)
-   [Safety and Guardrails](#safety-and-guardrails)
-   [Technology and Implementation](#technology-and-implementation)
-   [Testing and Evaluation](#testing-and-evaluation)
-   [Limitations](#limitations)
-   [Team Members](#team-members)

## Overview

Students often need to contact different offices for routine library and
hostel queries. CampusCare Voice Desk brings selected services into one
conversational interface.

The assistant can:

-   Search the library catalogue and report availability and shelf
    location.
-   Place a hold on an available book after the student confirms.
-   Calculate overdue library fines and apply the book replacement-price
    cap.
-   Check hostel bed availability by block and room category.
-   Submit eligible hostel room-change requests.
-   Escalate emergencies and route unsupported requests to the
    appropriate helpdesk.

The assistant is an assistance channel. It does **not** replace library,
hostel, administrative, or emergency staff.

## Key Features

  -----------------------------------------------------------------------
  Feature                             Description
  ----------------------------------- -----------------------------------
  Student identification              Validates student IDs in
                                      `STU-12345` format before
                                      transactions.

  Book search                         Searches by book title or book ID
                                      and reports available copies and
                                      shelf location.

  Book holds                          Places a hold only when a copy is
                                      available and the caller confirms.

  Fine calculation                    Calculates overdue fines using the
                                      defined tier structure and
                                      replacement-price cap.

  Hostel availability                 Checks vacancies by hostel block
                                      and room category, including
                                      applicable surcharge information.

  Room-change requests                Submits eligible requests and
                                      returns an HRC ticket number.

  Emergency escalation                Prioritises fire, medical, and
                                      physical-safety emergencies.

  Human assistance                    Offers transfer to a librarian or
                                      the relevant helpdesk.

  Voice-friendly responses            Keeps spoken replies short and
                                      confirms IDs and numbers digit by
                                      digit.

  Security guardrails                 Protects sensitive information and
                                      blocks prompt-injection attempts
                                      and harmful content.
  -----------------------------------------------------------------------

## System Architecture

The project follows a **hub-and-spoke multi-agent architecture**.

``` text
             Student / Caller
                    |
          +---------+---------+
          |                   |
      Voice channel       Digital chat
          |                   |
          +---------+---------+
                    |
       CampusCare Welcome Desk
               (Root Agent)
                    |
       Greeting, student ID, triage,
       scope check and emergency detection
                    |
          +---------+----------+
          |                    |
   Emergency detected?    Supported request?
          |                    |
          v                    v
 Campus Emergency       CampusCare Student
     Services             Services Agent
                                |
                 +--------------+--------------+
                 |              |              |
             Book lookup    Fine calculator  Hostel services
                 |              |              |
                 +--------------+--------------+
                                |
                     Simulated service data
                                |
                       Response to caller
```

The Welcome Desk handles greeting, identification, request triage, scope
checks, and emergency detection. Supported library and hostel requests
are routed to the CampusCare Student Services specialist agent.

### Backend tools

  -----------------------------------------------------------------------
  Tool                                Responsibility
  ----------------------------------- -----------------------------------
  `check_book_availability`           Searches the book catalogue,
                                      available copies, and shelf
                                      location.

  `calculate_library_fine`            Calculates the overdue fine and
                                      applies the replacement-price cap.

  `check_hostel_room_availability`    Checks hostel block, room category,
                                      available beds, and surcharge.

  `submit_room_change_request`        Checks room-change eligibility and
                                      creates an HRC ticket when
                                      permitted.
  -----------------------------------------------------------------------

Emergency calls follow a priority route from the root agent to **Campus
Emergency Services**. Requests outside the supported library and hostel
scope are redirected to the **Main University Helpdesk**.

## Supported Services and Business Rules

### 1. Student identification

-   A student ID is required before any transaction.
-   The accepted format is `STU-` followed by five digits, for example
    `STU-10001`.
-   Missing, incorrectly formatted, or unknown IDs must not trigger a
    transaction.
-   If identification remains unclear after repeated attempts, the
    assistant offers human assistance.

### 2. Library book search and holds

-   Books can be searched by title or book ID.
-   Book IDs follow the `BK-123` format.
-   When copies are available, the assistant reports the number of
    copies and shelf location.
-   A hold is placed only on an available copy and only after the
    student confirms.
-   A book with zero available copies has **no waitlist**. The assistant
    explains this and transfers the request to a librarian.
-   Ambiguous titles require a short clarification.

### 3. Overdue fine calculation

The fine is calculated using the following tiers:

  -----------------------------------------------------------------------
  Overdue period                                                     Fine
  ------------------------------ ----------------------------------------
  0 days                                                               ₹0

  1--7 days                                                    ₹5 per day

  8--30 days                                       ₹35 + ₹10 × (days − 7)

  More than 30 days               Stop automated calculation and escalate
                                                        to administration
  -----------------------------------------------------------------------

Examples:

-   5 days: `5 × ₹5 = ₹25`
-   7 days: `7 × ₹5 = ₹35`
-   8 days: `₹35 + ₹10 × 1 = ₹45`
-   30 days: `₹35 + ₹10 × 23 = ₹265`

Negative or non-numeric overdue days are invalid and must be requested
again.

**Replacement-price cap:** The final fine is the lower of the calculated
fine and the book's full replacement price.

### 4. Hostel bed availability

-   Availability is checked using the hostel block, room category,
    number of free beds, and surcharge.
-   Supported blocks are A--D.
-   The assistant must never claim a vacancy when no beds are available.
-   If the requested category is full or invalid, it suggests
    alternatives or asks the student to choose a valid category.

### 5. Hostel room-change requests

-   A request requires the student ID, current room, requested room
    type, and caller confirmation.
-   An eligible request creates an HRC ticket.
-   The stated review time is **3--5 business days**.
-   Only one room change is allowed per student per semester.
-   A second request in the same semester is rejected and no ticket is
    created.

### 6. Emergency and unsupported requests

-   Fire, medical, and physical-safety emergencies take priority over
    every other workflow.
-   Emergency escalation does not require a student ID.
-   The assistant gives one short safety instruction and transfers to
    Campus Emergency Services.
-   Requests outside library and hostel services---for example, bus,
    cafeteria, or grades---are redirected to the Main University
    Helpdesk.
-   Library issues that require a person are routed to a librarian.

## Safety and Guardrails

CampusCare includes guardrails for emergency handling, privacy, prompt
security, and responsible AI.

  -----------------------------------------------------------------------
  Guardrail                           Purpose
  ----------------------------------- -----------------------------------
  Domain & Emergency Guardrail        Detects fire, medical, and
                                      physical-safety emergencies and
                                      triggers priority escalation.

  PII & Payment Card Redaction        Prevents the assistant from
                                      repeating or handling card numbers,
                                      CVVs, bank PINs, and sensitive
                                      credentials.

  Prompt Security                     Blocks requests to reveal system
                                      prompts, jailbreak attempts, role
                                      impersonation, and code-execution
                                      requests.

  Responsible-AI safety filter        Blocks harmful content, including
                                      hate, harassment, sexually explicit
                                      content, and dangerous guidance.
                                      The configured threshold is
                                      `BLOCK_LOW_AND_ABOVE`.
  -----------------------------------------------------------------------

Additional response rules:

-   Voice replies should contain no more than three sentences and should
    avoid markdown, URLs, tables, and symbols.
-   IDs and amounts spoken by callers are read back digit by digit for
    confirmation.
-   If a backend tool fails or times out, the assistant must not invent
    a result. It explains that the service is temporarily unavailable
    and offers retry or staff escalation.
-   A blocked message must not trigger a backend tool call.

## Technology and Implementation

The project report records implementation and testing in **CX Agent
Studio**.

The implementation consists of:

-   A root Welcome Desk agent for greeting, triage, identification,
    emergency detection, and routing.
-   A specialist Student Services agent for library and hostel
    workflows.
-   Four simulated backend tools for book lookup, fine calculation,
    hostel availability, and room-change submission.
-   Simulated student, book-catalogue, and hostel-inventory data.
-   Configured safety and security guardrails.
-   Voice/telephony and digital-chat channel designs.

The supplied project data is fictional. The student records, catalogue,
and hostel inventory are sample data---not a live university database.

## Testing and Evaluation

The requirements and test-data document defines:

-   **12 user stories**
-   **32 edge cases**
-   **33 test-data scenarios**

The test coverage includes student-ID validation, book lookup and holds,
fine boundaries and price caps, hostel availability, room-change
eligibility, emergency escalation, unsupported requests, human transfer,
privacy, prompt injection, abusive language, and backend failures.

### Reported evaluation

The capstone report records a 16-scenario verification suite containing
four happy paths, four business-negative cases, four technical-fallback
cases, and four guardrail refusals.

  Evaluation run               Reported score
  -------------------------- ----------------
  Earlier run                             50%
  Second run                              60%
  Latest run (15 Nov 2026)            **88%**

The latest run recorded **14 passed, 2 mixed, and 0 failed** across 80
results in the text channel. The hostel-availability happy-path case was
mixed, with 3 of 5 trial results passing.

Manual preview evidence includes:

-   Book search: 3 copies of *Database System Concepts* on Shelf CS-3B.
-   Fine calculation: ₹25 for 5 overdue days.
-   Hostel availability: 1 available bed in a Double AC room in Block B.
-   Room change: HRC ticket `HRC-7107`, with a 3--5 business-day review
    period.
-   Emergency, payment-card redaction, prompt-injection refusal, and
    abusive-message handling.

**Testing evidence note:** The supplied screenshots show typed
interactions in the agent Preview interface. They do not establish that
speech input, audio output, or a live telephony call was tested end to
end.

## Limitations

-   Backend records and service data are simulated, not connected to a
    live university database.
-   The available catalogue, student records, and hostel inventory are
    limited sample datasets.
-   Correct operation depends on valid student IDs, clear user input,
    and functioning tool integrations.
-   Voice recognition may mishear names, book titles, or numbers;
    spoken-number ambiguity and background noise need validation.
-   Emergency escalation depends on a valid configured campus contact
    path.
-   Voice/telephony testing remains open; the reported evaluation
    evidence is from the text channel.
-   The latest evaluation included two mixed cases, and the
    hostel-availability happy path requires further investigation.
-   Additional evidence is still needed for some negative cases,
    including zero-stock handling, price-cap behaviour, full-room
    handling, and room-change policy rejection.

CampusCare is a prototype assistance channel and is not a replacement
for university staff or emergency services.

## Team Members

**Team Number: 26**

  Member              Register Number
  ------------------- -----------------
  Shubh Gupta         RA2411026010717
  Avni Gautam         RA2411026010714
  Anvi Gautam         RA2411026010678
  Subhakanta Pandab   RA2411026010665

------------------------------------------------------------------------

*CampusCare Voice Desk --- Capstone Project 26, Team 26.*
