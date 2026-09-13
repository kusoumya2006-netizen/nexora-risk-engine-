# Nexora — Real-Time Risk Analysis Engine

**SIH Problem Statement:** SIH22104 — AI-Powered Real-Time Detection and Prevention of Voice Cloning Impersonation Attack
**Project Name:** Nexora
**Module:** Conversation & Risk Analysis + Risk-Based Scoring
**Module Owner:** Member 4

---

## 1. Project Overview

**Nexora** is an AI-powered real-time anti-impersonation system designed to detect and prevent voice cloning and social-engineering attacks during live calls.

The system does not rely on a single signal. Instead, it combines three major sources of evidence:

1. **Voice Authenticity** — determines whether the caller's voice is genuine or potentially synthetic.
2. **Speaker Verification** — determines whether the caller matches the claimed identity.
3. **Conversation Risk Analysis** — determines whether the caller's behavior and requests indicate an impersonation or social-engineering attack.

These signals are combined to produce a single **Overall Risk Score**, which is then converted into a risk level, explanation, and recommended action.

---

# 2. Complete Nexora System Architecture

The following workflow shows how the contributions of all team members connect during a live call.

```text
                              LIVE CALL
                                  |
                                  v
                         +----------------+
                         |  AUDIO STREAM  |
                         +----------------+
                                  |
                  +---------------+---------------+
                  |                               |
                  v                               v
        +-------------------+             +-------------------+
        |     MEMBER 2      |             |     MEMBER 3      |
        | Voice Spoof       |             | Speaker           |
        | Detection         |             | Verification      |
        +-------------------+             +-------------------+
                  |                               |
                  v                               v
        voice_authenticity_score       speaker_verification_score
                  |                               |
                  |                               |
                  +---------------+---------------+
                                  |
                                  |
                                  v
                         +----------------+
                         |    MEMBER 4    |
                         | Risk Analysis  |
                         +----------------+
                                  |
                  +---------------+---------------+
                  |                               |
                  v                               v
          SPEECH-TO-TEXT                  AI VOICE SIGNALS
              (STT)                       FROM MEMBERS 2 & 3
                  |
                  v
             TRANSCRIPT
                  |
                  v
          SENTENCE SPLITTING
                  |
                  v
       18 PRIMARY BEHAVIOR DETECTORS
                  |
                  v
       CONTEXTUAL SUPPRESSION
                  |
                  v
            DEDUPLICATION
                  |
                  v
       DETECTED RISK CATEGORIES
                  |
                  v
       COMPOUND RISK ANALYSIS
                  |
                  v
       CONVERSATION RISK SCORE
                  |
                  +-------------------+
                                      |
                                      v
                           MULTI-MODAL RISK FUSION
                                      |
                                      v
                             OVERALL RISK SCORE
                                  (0–100)
                                      |
                                      v
                         RISK LEVEL + EXPLANATION
                                      |
                                      v
                           RECOMMENDED ACTION
                                      |
                     +----------------+----------------+
                     |                                 |
                     v                                 v
             +---------------+                 +---------------+
             |   MEMBER 5    |                 |   MEMBER 6    |
             |   Frontend    |                 | Alert /       |
             |   Dashboard   |                 | Response      |
             +---------------+                 +---------------+
```

---

# 3. Contribution of Each Team Member

## Member 2 — Voice Spoof Detection

Member 2 is responsible for analyzing the **authenticity of the caller's voice**.

The module determines whether the incoming voice appears to be:

* Genuine
* Synthetic
* Potentially voice-cloned

It provides the following output to Member 4:

```text
voice_authenticity_score
```

Score convention:

```text
1.0 → Genuine voice
0.0 → Cloned / synthetic voice
```

Member 4 converts this confidence value into a voice risk score:

```text
voice_risk = (1.0 - voice_authenticity_score) × 100
```

This allows voice authenticity to become one of the inputs to the final multi-modal risk decision.

---

## Member 3 — Speaker Verification

Member 3 is responsible for determining whether the caller is actually the person they claim to be.

The speaker verification module compares the incoming speaker against the expected identity and produces:

```text
speaker_verification_score
```

Score convention:

```text
1.0 → Speaker identity matched
0.0 → Imposter / identity mismatch
```

Member 4 converts this into:

```text
speaker_risk = (1.0 - speaker_verification_score) × 100
```

This provides the identity-verification component of the final risk calculation.

---

## Member 4 — Conversation & Risk Analysis

Member 4 is responsible for the **decision-making layer of Nexora**.

This module analyzes what is happening inside the conversation and determines whether the caller's behavior indicates a potential social-engineering or impersonation attack.

The module receives:

```text
transcript
voice_authenticity_score
speaker_verification_score
```

It then performs:

1. Sentence-level conversation analysis
2. Primary behavioral detection
3. Contextual false-positive suppression
4. Category deduplication
5. Compound attack detection
6. Conversation risk scoring
7. Voice and speaker risk conversion
8. Multi-modal risk fusion
9. Risk-level classification
10. Human-readable explanation generation
11. Recommended-action generation

The output is:

```text
overall_risk_score
risk_level
conversation_risk_score
risk_factors
explanation
recommended_action
```

Member 4 therefore acts as the **risk intelligence and decision layer** connecting the voice-analysis modules with the frontend and real-time response modules.

---

## Member 5 — Frontend Dashboard

Member 5 consumes the decision produced by the Risk Analysis Engine.

The dashboard receives:

```text
overall_risk_score
risk_level
risk_factors
explanation
```

The purpose of the frontend is to make the system's risk assessment understandable to the operator or user in real time.

Instead of exposing only a numerical score, the dashboard can present:

```text
Overall Risk
Risk Level
Detected Factors
Evidence
Explanation
```

This allows the user to understand not only **how risky the call is**, but also **why the system considers it risky**.

---

## Member 6 — Real-Time Alert & Response

Member 6 consumes the final action recommendation generated by Member 4.

The possible actions are:

```text
ALLOW_CALL
VERIFY_IDENTITY
CHALLENGE_OR_BLOCK
```

The response layer uses this decision to determine what should happen next during the live call.

The overall flow is:

```text
Risk Engine
    |
    v
Recommended Action
    |
    +---- ALLOW_CALL
    |
    +---- VERIFY_IDENTITY
    |
    +---- CHALLENGE_OR_BLOCK
```

---

# 4. Member 4 Internal Workflow

The Member 4 module itself consists of two major stages.

```text
                  LIVE TRANSCRIPT
                        |
                        v
                SENTENCE SPLITTING
                        |
                        v
             +----------------------+
             |      STAGE 1         |
             | Behavioral Detection |
             +----------------------+
                        |
                        v
              18 Risk Categories
                        |
                        v
           Contextual Suppression
                        |
                        v
                 Deduplication
                        |
                        v
              Detected Categories
                        |
                        v
             +----------------------+
             |      STAGE 2         |
             | Compound Risk        |
             | Analysis              |
             +----------------------+
                        |
                        v
             Conversation Risk
                  Score 0–100
```

The conversation score is then combined with the outputs from Members 2 and 3.

```text
Voice Authenticity Score
            |
            v
        Voice Risk
            |
            |
Speaker Verification Score
            |
            v
       Speaker Risk
            |
            |
Conversation Analysis
            |
            v
    Conversation Risk
            |
            +-------------+
                          |
                          v
                Multi-Modal Fusion
                          |
                          v
                 Overall Risk Score
                          |
                          v
                  Risk Classification
                          |
                          v
                 Explanation + Action
```

---

# 5. Conversation Analysis Methodology

Nexora uses a **Two-Stage Contextual and Rule-Based NLP Pipeline** designed for high-precision and low-latency hackathon demonstration.

The current NLP implementation is a contextual rule-based system optimized for real-time prototype demonstration. It is not a clinically or scientifically validated statistical probability model.

---

## Stage 1 — Primary Behavioral Detection

The first stage analyzes the transcript at sentence level and detects 18 primary suspicious behaviors.

### 1. OTP Request

Detects requests for:

* OTPs
* Authentication codes
* Passcodes

Example:

```text
"Tell me the OTP you just received."
```

---

### 2. Money Transfer Request

Detects requests involving:

* UPI transfers
* Bank transfers
* Wire transfers
* Sending money

---

### 3. Password / PIN Request

Detects requests for:

* Passwords
* ATM PINs
* Account credentials

---

### 4. Bank Credential Request

Detects requests for:

* Bank account numbers
* CVVs
* Netbanking credentials
* Banking information

---

### 5. Card Information Request

Detects requests for:

* Credit/debit card numbers
* Expiry dates
* Card details

---

### 6. Remote Access Request

Detects requests to provide remote access using applications such as:

```text
AnyDesk
TeamViewer
```

---

### 7. Software Installation Request

Detects instructions to install:

* Unknown applications
* APK files
* Remote-support software

---

### 8. Possible Bank Impersonation

Detects claims of representing:

* Banks
* Fraud departments
* Banking security teams

Example:

```text
"I'm calling from your bank."
```

---

### 9. Police / Government Impersonation

Detects claims of representing:

* Police
* CBI
* Tax authorities
* Government departments
* Legal authorities

---

### 10. Technical Support Impersonation

Detects claims of representing:

* Customer care
* Technical support
* IT helpdesk

---

### 11. Account Blocking Threat

Detects threats such as:

```text
"Your account will be blocked."
```

---

### 12. Fear / Threat Language

Detects:

* Arrest threats
* Legal-action threats
* Penalty threats
* Fear-based manipulation

---

### 13. Unusual Urgency / Pressure

Detects pressure to act immediately without allowing independent verification.

Examples include:

```text
"Do this immediately."
"You only have five minutes."
"Don't wait."
```

---

### 14. Suspicious Link

Detects instructions to open:

* Shortened URLs
* Suspicious links
* Potential phishing URLs

---

### 15. Bypass Verification Attempt

Detects instructions such as:

```text
"Don't contact the bank."
"Don't visit the branch."
"Don't tell customer support."
```

---

### 16. Secrecy Request

Detects requests to hide the conversation or transaction from:

* Family members
* Banks
* Authorities
* Other trusted parties

---

### 17. Confidential Information Request

Detects requests for sensitive personal information.

---

### 18. Urgent Payment Request

Detects immediate payment requests presented as emergencies or urgent situations.

---

# 6. Contextual False-Positive Handling

The system does not treat every suspicious keyword as an attack.

For example:

```text
"The bank told me never to share my OTP with anyone over the phone."
```

The word `OTP` is present, but the speaker is warning against sharing it.

The contextual detection layer therefore suppresses the OTP risk.

This prevents the system from incorrectly classifying legitimate security-related conversations as attacks.

---

# 7. Deduplication

Repeated occurrences of the same suspicious behavior are prevented from unnecessarily increasing the score.

For example, if a caller repeatedly asks for an OTP, the system does not continuously add the same category score for every occurrence.

This keeps the scoring mechanism bounded and prevents repetitive speech from artificially inflating the conversation risk.

---

# 8. Stage 2 — Compound Risk Analysis

Individual suspicious behaviors can become significantly more meaningful when they occur together.

The compound risk layer therefore evaluates combinations of detected categories.

The purpose is to identify **coordinated multi-vector social-engineering attacks** rather than treating each behavior independently.

To maintain balanced scoring, at most **one compound factor is added per conversation**.

---

## Compound Pattern 1 — Coordinated Financial Social-Engineering

**Score:** +25

### Trigger

```text
bank_impersonation
+
credential request
+
financial action request
```

Credential requests include:

```text
otp_request
bank_credential_request
card_info_request
password_pin_request
```

Financial requests include:

```text
financial_transfer
urgent_payment
```

### Evidence

```text
Bank impersonation combined with credential collection
and financial action request.
```

This pattern represents a coordinated attempt to establish authority, obtain sensitive credentials, and initiate a financial action.

---

## Compound Pattern 2 — Coordinated Remote-Access Scam

**Score:** +25

### Trigger

```text
tech_support_impersonation
+
remote_access_request
OR
software_install_request
```

### Evidence

```text
Technical support impersonation combined with remote access
or software installation request.
```

---

## Compound Pattern 3 — Coercive Impersonation

**Score:** +20

### Trigger

```text
Impersonation
+
account_blocking_threat OR fear_threat_language
+
unusual_urgency OR urgent_payment
```

### Evidence

```text
Impersonation combined with threat language
and immediate pressure.
```

---

## Compound Pattern 4 — Coordinated Social-Engineering

**Score:** +20

### Trigger

```text
Impersonation
+
Sensitive Information Request
+
Urgency OR Secrecy OR Bypass Verification
```

### Evidence

```text
Impersonation combined with sensitive information request
and manipulation tactics.
```

---

# 9. Conversation Risk Scoring

The individual behavioral detections and applicable compound factor are combined to produce a raw conversation score.

The resulting score is bounded:

```text
conversation_risk = min(100, raw_score)
```

Therefore:

```text
0   → No significant conversational risk
100 → Maximum conversational risk
```

---

# 10. Multi-Modal Risk Fusion

Conversation analysis alone is not used to make the final decision.

Nexora combines:

```text
Voice Risk
Speaker Risk
Conversation Risk
```

The configured weights are:

| Signal                     | Weight |
| -------------------------- | -----: |
| Voice Authenticity Risk    |    30% |
| Speaker Verification Risk  |    30% |
| Conversation Behavior Risk |    40% |

The final score is:

```text
Overall Risk Score =
    (0.30 × Voice Risk)
  + (0.30 × Speaker Risk)
  + (0.40 × Conversation Risk)
```

The final value is rounded and bounded between 0 and 100.

---

# 11. Risk Levels and Decision Mapping

| Score Range | Risk Level | Recommended Action   | Operator Guidance                                                                            |
| ----------: | ---------- | -------------------- | -------------------------------------------------------------------------------------------- |
|        0–39 | LOW        | `ALLOW_CALL`         | Normal conversation detected. No intervention required.                                      |
|       40–69 | MEDIUM     | `VERIFY_IDENTITY`    | Suspicious behavior or voice uncertainty detected. Additional verification is advised.       |
|      70–100 | HIGH       | `CHALLENGE_OR_BLOCK` | Strong evidence of impersonation or coordinated attack. Challenge, alert, or block the call. |

---

# 12. Explainability Layer

Nexora does not return only a numerical risk score.

The engine also produces a human-readable explanation containing:

* Overall risk level
* Overall score
* Detected behaviors
* Evidence from the conversation
* Voice risk
* Speaker risk
* Recommended action

Example:

```text
HIGH RISK (78/100)

Impersonation attack likely.

Reasons:
- OTP request
- Money transfer request
- Possible bank impersonation
- Account blocking threat
- Unusual urgency / pressure
- Coordinated financial social-engineering pattern

Voice Risk: 65%
Speaker Risk: 60%

Recommended Action:
CHALLENGE_OR_BLOCK
```

This makes the decision auditable and understandable rather than presenting a black-box score.

---

# 13. API Documentation

## POST `/analyze`

The endpoint evaluates the conversation transcript and AI voice signals and returns the complete risk assessment.

### Request

```json
{
  "transcript": "I'm calling from your bank. Your account will be blocked. Tell me your OTP and transfer ₹20,000 immediately.",
  "voice_authenticity_score": 0.35,
  "speaker_verification_score": 0.40
}
```

### Input Parameters

| Parameter                    | Type   | Description                                 |
| ---------------------------- | ------ | ------------------------------------------- |
| `transcript`                 | string | Speech-to-text conversation transcript      |
| `voice_authenticity_score`   | float  | Voice authenticity confidence from Member 2 |
| `speaker_verification_score` | float  | Speaker identity confidence from Member 3   |

---

# 14. Example End-to-End Analysis

Consider the following live conversation:

```text
"I'm calling from your bank. Your account will be blocked.
Tell me your OTP and transfer ₹20,000 immediately."
```

Member 2 provides:

```text
voice_authenticity_score = 0.35
```

Member 3 provides:

```text
speaker_verification_score = 0.40
```

Member 4 converts these values:

```text
voice_risk   = (1 - 0.35) × 100 = 65
speaker_risk = (1 - 0.40) × 100 = 60
```

The conversation detector identifies:

```text
OTP request
Money transfer request
Possible bank impersonation
Account blocking threat
Unusual urgency / pressure
```

The compound layer identifies:

```text
Coordinated financial social-engineering pattern
```

The conversation reaches:

```text
conversation_risk_score = 100
```

The multi-modal fusion produces:

```text
Overall Risk Score = 78
```

Therefore:

```text
Risk Level = HIGH
Recommended Action = CHALLENGE_OR_BLOCK
```

Member 5 can display this result on the dashboard, while Member 6 can trigger the appropriate real-time response.

---

# 15. Verified Demo Scenarios

| Scenario           | Example                                                                                                        | Conversation Risk | Overall Score | Risk Level | Action               | Compound Pattern                         |
| ------------------ | -------------------------------------------------------------------------------------------------------------- | ----------------: | ------------: | ---------- | -------------------- | ---------------------------------------- |
| Normal             | "Hey, are we still meeting at 5 today?"                                                                        |                0% |             4 | LOW        | `ALLOW_CALL`         | None                                     |
| Suspicious         | "Can you send me your account details? I need to verify something."                                            |               30% |            42 | MEDIUM     | `VERIFY_IDENTITY`    | None                                     |
| Bank Impersonation | "I'm calling from your bank. Your account will be blocked. Tell me your OTP and transfer ₹20,000 immediately." |              100% |            78 | HIGH       | `CHALLENGE_OR_BLOCK` | Coordinated financial social-engineering |
| Remote Access      | "I'm from technical support. Install this application and give me remote access to your computer."             |              100% |            84 | HIGH       | `CHALLENGE_OR_BLOCK` | Coordinated remote-access scam           |
| Contextual OTP     | "The bank told me never to share my OTP with anyone over the phone."                                           |                0% |             4 | LOW        | `ALLOW_CALL`         | Contextually suppressed                  |

---

# 16. Testing

The project contains **22 automated tests** covering:

* Individual behavioral detectors
* Compound risk detection
* Contextual false-positive suppression
* Score normalization
* Multi-modal risk fusion
* FastAPI endpoint routes

Run the complete test suite:

```bash
./venv/bin/pytest tests/ -v
```

Expected result:

```text
22 passed
```

---

# 17. Running the Project

Navigate to the risk-analysis module:

```bash
cd risk-analysis
```

Run the automated tests:

```bash
./venv/bin/pytest tests/ -v
```

Run the interactive demo:

```bash
./venv/bin/python run_demo_analysis.py
```

Start the FastAPI server:

```bash
./venv/bin/python app.py
```

The API server runs locally at:

```text
http://localhost:8000
```

---

# 18. Integration Contract

The following contract defines how the Member 4 Risk Engine communicates with the other Nexora modules.

### Input from Member 2

```text
voice_authenticity_score
```

Range:

```text
1.0 = Genuine
0.0 = Cloned
```

---

### Input from Member 3

```text
speaker_verification_score
```

Range:

```text
1.0 = Identity matched
0.0 = Imposter
```

---

### Input from Speech-to-Text

```text
transcript
```

The transcript is analyzed by the Member 4 conversation engine.

---

### Output to Member 5

```text
overall_risk_score
risk_level
conversation_risk_score
risk_factors
explanation
```

---

### Output to Member 6

```text
recommended_action
```

Possible values:

```text
ALLOW_CALL
VERIFY_IDENTITY
CHALLENGE_OR_BLOCK
```

---

# 19. Complete Data Contract

The complete Member 4 interface can therefore be represented as:

```text
                    INPUTS
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
   Transcript     Voice Score     Speaker Score
       |               |               |
       +---------------+---------------+
                       |
                       v
              MEMBER 4 RISK ENGINE
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
 Conversation     Multi-Modal     Explainability
   Analysis          Fusion           Layer
       |               |               |
       +---------------+---------------+
                       |
                       v
                    OUTPUT
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
 Overall Risk      Explanation     Recommended
    Score                            Action
       |
       v
   Member 5
  Dashboard

Recommended Action
       |
       v
   Member 6
Alert / Response
```

---

# 20. Security and Privacy

* Call transcripts are processed transiently in memory and discarded.
* Real authentication codes, PINs, passwords, or credentials are never logged or stored.
* Demonstration scenarios use synthetic hackathon data.
* The system is designed as a real-time risk decision-support component of the Nexora platform.

---

# 21. Role of the Member 4 Module in Nexora

The Member 4 module serves as the **central risk decision layer** of Nexora.

The individual modules answer different questions:

```text
Member 2
"Does the voice appear genuine?"
                |
                v
       Voice Authenticity


Member 3
"Does the speaker match the claimed identity?"
                |
                v
       Speaker Verification


Member 4
"Does the conversation indicate an attack,
and how should all available signals be combined?"
                |
                v
       Risk Analysis + Fusion
                |
                v
       Final Risk Decision


Member 5
"How should the risk be presented to the user?"
                |
                v
           Dashboard


Member 6
"What should the system do about the risk?"
                |
                v
        Alert / Response
```

Together, these modules form the complete Nexora real-time anti-impersonation pipeline.
