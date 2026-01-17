# Power-Instability-System-P.IS-

P.IS is a security accountability system that records and verifies system integrity, authentication, and configuration states before, during, and after power instability events, preventing power outages from being used to obscure fraud, tampering, or unauthorized activity.


The Power Instability Accountability System (P.IS) is a conceptual security and forensic system designed to preserve system accountability during power instability events such as load shedding, brownouts, or deliberate power disruption.
P.IS does not attempt to solve electrical grid failures.
Instead, it addresses a recurring institutional problem:
Power instability is frequently used to obscure responsibility for data loss, fraud, unauthorized access, or system tampering.
P.IS establishes an independent, tamper-evident record of system state before, during, and after power disruptions, enabling post-event verification and forensic analysis.
This repository presents the architectural design, invariants, and operational model of P.IS for academic, research, and evaluation purposes.

# Problem Statement
Across government institutions, universities, utilities, and defence environments, critical events often coincide with power instability. Typical outcomes include:
Loss of audit trails
Unverifiable authentication claims
Configuration changes with no attribution
Claims of “system downtime” used as justification for anomalies
Without an independent mechanism to attest to what the system state actually was, accountability collapses.
P.IS exists to answer one question reliably:
What was the exact security and integrity state of the system when power failed?

# Core Design Principle

Power failure must not equal accountability failure.
P.IS treats power instability as a state transition event that must be cryptographically recorded and verifiable, not as an excuse for missing evidence.

# System Scope

P.IS monitors and attests to four critical system state domains:
Power State
Normal operation
Brownout
Outage
Recovery
Authentication State
Active sessions
Privilege levels
Login/logout transitions
Data Integrity State
Hashes of protected datasets
Write operations in progress
Integrity before/after outage
Configuration State
Security configuration snapshots
Firmware/software integrity
Critical parameter changes
Each state transition is recorded as an immutable event.

# High-Level Architecture

P.IS is designed as an out-of-band accountability module:
Operates independently from the primary system
Maintains power via auxiliary sources (battery/supercapacitor)
Records cryptographically signed state transitions
Emits events to secure audit pipelines (e.g. NFDAS)
P.IS does not control systems.
It observes, attests, and proves.

# Operational Model
System operates normally
P.IS continuously snapshots defined invariants
Power instability detected
Pre-outage state sealed and signed
During outage, minimal state monitoring continues
On recovery, post-outage state is captured
Differences are correlated and flagged
The result is a verifiable timeline immune to after-the-fact manipulation.

# Formal Invariants (Conceptual)

P.IS enforces invariant logic of the form:
If Power = Lost, then Integrity must remain unchanged
If Authentication changed during outage, then flag violation
If Configuration differs post-recovery, then require explanation
These invariants are modeled using discrete mathematics and implication logic, enabling formal reasoning about system behavior.

# Integration with Other Systems

P.IS is designed to integrate with higher-level security platforms such as:
NFDAS (National Fraud Detection and Accountability System)
For correlation of power events with fraud, audit, or insider threat activity.
SIEM / SOC platforms
As a trusted source of ground-truth state transitions.

# Use Cases

Government departments and public institutions
Defence and military communication environments
Utilities and infrastructure operators
Universities and examination systems
Financial and regulatory bodies
Anywhere power instability intersects with accountability risk.

# Ethical & Security Notice

This project is presented as a research and conceptual design.
Sensitive implementation details are intentionally withheld
No operational attack techniques are provided
The system is designed for defensive, forensic, and accountability purposes only
Unauthorized deployment or misuse is not permitted without explicit author approval.

# (Implementation code, where present, is illustrative only.)
Conceptual Design – Academic & Evaluation Stage
Further work may include:
Formal truth tables for invariants
Embedded firmware reference model
Secure key provisioning design.

# Author
Sandile Mabiba,Independent System Designer.
# Focus areas: Computer Science, Security Systems, Accountability Architecture

# Code Snippets 

# ESP32 Firmware (Core, Minimal, Serious)
Target assumptions:
ESP32
External RTC optional (ESP32 RTC is enough initially)
FRAM / flash
GPIO sensors (tamper switch, power detect, etc.)
Firmware Philosophy
No delays
No recovery logic
No auto-correction
Observe, record, sign

Core State Model (C++/Arduino)

#include <Arduino.h>

// ---------------- STATES ----------------
enum PowerState     { POWER_ON, POWER_OFF, POWER_TRANSITION };
enum AuthState      { LOCKED, UNLOCKED };
enum DataState      { CONSISTENT, DIRTY };
enum ConfigState    { BASELINE, MODIFIED };
enum PhysicalState  { SEALED, BREACHED };

// ---------------- GLOBAL STATE ----------------
PowerState powerState;
AuthState authState;
DataState dataState;
ConfigState configState;
PhysicalState physicalState;

// ---------------- PROTOTYPES ----------------
void sampleInputs();
void checkInvariants();
void logEvent(const char* msg);
void logViolation(const char* msg);

// ---------------- SETUP ----------------
void setup() {
    Serial.begin(115200);

    // Initialize states
    powerState    = POWER_ON;
    authState     = LOCKED;
    dataState     = CONSISTENT;
    configState   = BASELINE;
    physicalState = SEALED;

    logEvent("PIS booted");
}

// ---------------- LOOP ----------------
void loop() {
    sampleInputs();
    checkInvariants();
}

# Input Sampling(Hardware driven)

void sampleInputs() {
    // Example GPIO reads (replace with real pins)
    bool powerGood   = digitalRead(34);
    bool tamperTrip  = digitalRead(35);

    // Power state
    if (!powerGood && powerState != POWER_OFF) {
        powerState = POWER_TRANSITION;
        logEvent("Power loss detected");
        powerState = POWER_OFF;
    }

    if (powerGood && powerState != POWER_ON) {
        powerState = POWER_ON;
        logEvent("Power restored");
    }

    // Physical state
    if (tamperTrip && physicalState != BREACHED) {
        physicalState = BREACHED;
        logViolation("Physical breach");
    }
}

# Invariant Engine (finite-state verifier)

void checkInvariants() {

    if (powerState == POWER_OFF && authState != LOCKED) {
        logViolation("AUTH unlocked during power-off");
    }

    if (powerState == POWER_OFF && dataState != CONSISTENT) {
        logViolation("DATA modified during power-off");
    }

    if (physicalState == BREACHED) {
        logViolation("Physical seal breached");
    }

    if (configState == MODIFIED && powerState != POWER_ON) {
        logViolation("Config change without power");
    }
}

# Example SHA-256 + ECDSA

#include "mbedtls/sha256.h"
#include "mbedtls/ecdsa.h"

void signLog(const uint8_t* data, size_t len, uint8_t* signature) {
    uint8_t hash[32];
    mbedtls_sha256(data, len, hash, 0);

    // ECDSA signing using device private key
    // (Key provisioning happens once at manufacturing)
}
Each log record stores:
Plaintext event
Hash
Signature
If one byte changes → signature fails.
This is chain-of-custody inside a microcontroller.
