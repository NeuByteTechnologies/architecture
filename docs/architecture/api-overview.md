---
layout: default
title: API Overview
description: High‑level architectural overview of the NeuByte Fitness App API, including purpose, structure, authentication model, service boundaries, and integration patterns.
---

# API Overview  
*NeuByte Fitness App — Backend API Architecture Summary*

The NeuByte Fitness App API is the backend subsystem responsible for authentication, program management, workout logging, weight tracking, notifications, and user profile operations.  
It exposes a stateless REST interface consumed by the Fitness App frontend and is designed for clarity, modularity, and long‑term maintainability.

This document provides a high‑level architectural overview of the API, its responsibilities, service boundaries, authentication model, and integration patterns.  
For detailed endpoint specifications, refer to the **Unified API Endpoint List** in the API repository.

---

# 1. Purpose of the API

The API serves as the **authoritative source of truth** for all application data and business logic.  
Its primary responsibilities include:

- Authenticating users (CAPTCHA → Credentials → MFA → Session)
- Managing exercise programs and daily workout structures
- Logging workout performance metrics
- Tracking body weight and personal progress
- Delivering notifications and unread counts
- Providing dashboard composite data
- Supporting reporting and insights
- Enforcing security, validation, and data integrity

The API is intentionally **stateless**, **REST‑based**, and **modular**, enabling horizontal scalability and clean separation of concerns.

---

# 2. High‑Level Architecture

The API is composed of several logical services, each responsible for a specific domain:

