# SysSight - Enterprise Workstation Monitoring Platform
### Case Study: Ben-Gurion University of the Negev

---

## Executive Summary

Ben-Gurion University of the Negev (BGU) operates one of Israel's largest and most active academic computing environments, with hundreds of Windows workstations distributed across labs, classrooms, and administrative departments. Managing this infrastructure manually had become untenable - IT staff lacked real-time visibility into machine health, struggled to track software installations across the fleet, and had no centralized way to push remote commands or plan hardware capacity.

**SysSight** was designed and built as a purpose-fit solution: a three-tier enterprise monitoring and remote management platform that gives BGU's IT administrators a single pane of glass into every workstation on the network - in real time.

---

## The Challenge

### A Fleet with No Central Visibility

BGU's IT department managed a large and geographically distributed fleet of Windows workstations without a unified monitoring solution. The challenges were operational and strategic:

- **No real-time status.** Administrators had no reliable way to know which machines were online, who was logged in, or what hardware resources were being consumed at any moment.
- **Reactive support model.** Faults and resource exhaustion were discovered only after users reported problems, increasing response time and friction.
- **Software drift.** There was no audit trail of installed software across the fleet, making compliance and license management difficult.
- **Manual remote operations.** Tasks like restarting a lab machine or logging off a session required physical presence or reliance on existing tools that were not purpose-built for this environment.
- **Capacity planning was guesswork.** Without usage data, decisions about hardware refresh cycles and lab sizing were based on intuition rather than evidence.
- **Multi-department complexity.** Different organizational units needed isolated visibility into their own machines, without exposing data across departments.

---

## The Solution: SysSight

SysSight is a full-stack enterprise monitoring and remote management platform built from the ground up for BGU's environment. It combines a lightweight Windows agent deployed to every managed machine, a high-performance REST API backend, and a modern web dashboard - giving IT administrators complete operational control from any browser.

---

## Architecture Overview

SysSight follows a clean, three-tier architecture designed for reliability, scalability, and ease of deployment.

```
┌─────────────────────────────────┐
│     Web Dashboard (Frontend)    │
│   React · TypeScript · MUI      │
│   English / Hebrew · RTL ready  │
└────────────────┬────────────────┘
                 │  HTTPS
┌────────────────▼────────────────┐
│     REST API (Backend)          │
│   FastAPI · Python · PostgreSQL │
│   JWT Auth · APScheduler        │
└────────────────┬────────────────┘
                 │  HTTPS
┌────────────────▼────────────────┐
│     Windows Agent               │
│   Python · Windows Service      │
│   Heartbeat · Events · Tasks    │
└─────────────────────────────────┘
```

### Windows Agent

A lightweight Python agent is deployed to each managed workstation as a native Windows Service. It runs silently in the background and:

- Sends a **heartbeat every 10 seconds**, reporting CPU, RAM, and disk utilization, current logged-in user, and machine status.
- Detects and reports **user login/logout events**, application open/close sessions, and power state changes (boot, shutdown, suspend, resume) in real time.
- **Polls for remote tasks** and executes them immediately - shell commands, restarts, logoffs, and agent self-updates.
- Collects a **full hardware and software inventory** on registration and on demand.
- Includes an optional **system tray application** visible to end users, providing a friendly onboarding wizard on first install.

The agent is distributed as a signed MSI installer, making fleet-wide deployment straightforward via standard IT tooling.

### Backend API

The backend is a high-performance asynchronous REST API built with **FastAPI** and **Python 3.11**, backed by **PostgreSQL**. It handles:

- Agent registration and authentication
- Real-time telemetry ingestion and storage
- Task orchestration and result collection
- Report generation across all data dimensions
- Scheduled task execution via an integrated job scheduler (APScheduler)
- Multi-tenant organization management and user access control

### Web Dashboard

The frontend is a responsive single-page application built with **React 18**, **TypeScript**, and **Material UI**. It features:

- Full internationalization with **English and Hebrew** support, including complete RTL layout rendering
- Live-updating computer list and dashboard (10-second auto-refresh)
- Interactive charts powered by **Recharts** and **Nivo**
- PDF export for formal reports via **jsPDF**
- Role-based UI that adapts to the permissions of the signed-in user

---

## Core Features

### Real-Time Fleet Monitoring

The Computers dashboard presents every managed machine in one view, with live status indicators, current user, logon time, and resource gauges. Administrators can filter by group, status, or search by hostname - and drill into any machine for a full detail view including telemetry history charts.

### Remote Control

From the dashboard, administrators can execute remote operations on any machine or group of machines without leaving the browser:

| Action | Description |
|--------|-------------|
| **Restart** | Gracefully restart the target machine |
| **Shutdown** | Power off on command |
| **Logoff** | End the current user session |
| **Wake-on-LAN** | Power on machines remotely via Magic Packet |
| **Shell Command** | Execute arbitrary commands and capture output |
| **Agent Update** | Push agent updates across the fleet without manual intervention |

All task results - including stdout, stderr, and exit code - are captured and stored for audit purposes.

### Computer Grouping

Machines can be organized into named groups (e.g., *Computer Lab 3*, *Library Wing*, *Admin PCs*). Groups serve as targets for bulk remote commands and scheduled tasks, enabling IT administrators to manage logical collections of machines as a unit.

### Scheduled Tasks

Administrators can define recurring maintenance tasks - daily, weekly, or one-time - targeting individual machines or entire groups. The integrated scheduler handles execution and retries automatically, and all results are logged for review.

### Comprehensive Reporting Suite

SysSight includes a full reporting suite covering every dimension of fleet usage:

| Report | What it shows |
|--------|--------------|
| **Hardware Inventory** | CPU, RAM, disk specs across the entire fleet |
| **Software Audit** | Installed applications per machine, filterable fleet-wide |
| **Login Duration** | Per-user session lengths and activity patterns |
| **Application Usage** | Which apps are used, by whom, and for how long |
| **Security Updates** | Machines missing critical Windows updates |
| **Power Sessions** | Boot, shutdown, and suspend/resume timelines |
| **Concurrent Usage** | How many machines were in active use at any point in time |

All reports support date range filtering and are exportable for further analysis.

### Capacity Planning Report

One of SysSight's most distinctive capabilities is its **statistical capacity planning engine**. Rather than presenting raw usage counts, it applies a **95th-percentile (P95) concurrent usage model** across configurable time buckets (15 or 30 minutes) to derive a statistically sound estimate of required compute capacity.

This transforms historical usage data into a concrete, evidence-based answer to the question: *"How many machines do we actually need?"* - removing guesswork from hardware procurement and lab sizing decisions. Reports are exportable as polished PDFs suitable for budget and planning discussions.

### Multi-Tenant Organization Management

SysSight supports multiple independent organizational units within a single deployment. Each organization sees only its own machines, users, and data. Users can belong to multiple organizations and switch context without logging out.

Three access roles enforce appropriate boundaries:

| Role | Capabilities |
|------|-------------|
| **Super Admin** | Full platform access; creates and manages organizations |
| **Org Admin** | Full access within their organization |
| **Viewer** | Read-only access to dashboards and reports |

### Localization

The entire interface - menus, labels, reports, dialogs, and error messages - is fully translated into **English and Hebrew**, with automatic RTL layout switching for Hebrew. This reflects BGU's bilingual operational environment and ensures the platform is native-feeling for all staff.

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| **Agent** | Python 3, cx_Freeze (MSI packaging), psutil, WMI, pywin32, pystray |
| **Backend** | Python 3.11, FastAPI, SQLAlchemy 2.0, Pydantic v2, APScheduler, Argon2 |
| **Database** | PostgreSQL 15 |
| **Frontend** | React 18, TypeScript 5, Vite, Material UI v5, Recharts, Nivo, i18next, jsPDF |
| **Infrastructure** | Docker Compose, self-hosted on BGU network infrastructure |

---

## Deployment & Integration

SysSight is deployed entirely on BGU's internal network infrastructure, ensuring that all monitoring data remains within the university's security perimeter.

- **Backend and database** run as Docker containers, simplifying deployment and upgrades.
- **The web dashboard** is served over HTTPS and accessible from any browser on the university network.
- **The Windows agent** is distributed as an MSI package, compatible with standard enterprise deployment tooling.
- **SMTP integration** with BGU's internal mail server enables automated email notifications for invitations and user management.

---

## Outcomes

SysSight transformed how BGU's IT department manages its workstation fleet:

**From reactive to proactive.** Real-time heartbeat monitoring and automated alerting mean faults are identified before users report them. Administrators see CPU spikes, machines going offline, or sessions left logged in - and can act immediately from the dashboard.

**Remote operations at scale.** Tasks that previously required physical presence or ad-hoc scripting - restarting a lab, logging off forgotten sessions, pushing a command to a group of machines - are now executed in seconds from the browser, with captured output for accountability.

**Evidence-based capacity planning.** The P95 capacity model gives IT leadership a defensible, data-driven basis for hardware procurement decisions. Lab sizing and refresh cycles are now driven by actual usage patterns rather than estimates.

**Software compliance visibility.** The software audit report provides a fleet-wide view of installed applications on demand, supporting license compliance and security posture reviews.

**Departmental self-service.** The multi-tenant model allows individual departments to manage their own machines independently, reducing the central IT team's coordination overhead while maintaining proper access boundaries.

**Bilingual, accessible interface.** Full Hebrew/English support with RTL layout means all staff - regardless of preferred language - can use the platform natively.

---

## About the Project

SysSight was designed and developed as a purpose-built internal tool for Ben-Gurion University of the Negev, Israel. It is deployed and actively used in production at `syssight.bgu.ac.il`.

The project demonstrates that enterprise-grade monitoring capabilities do not require expensive commercial software: with modern open-source tooling and a clear understanding of operational needs, a small engineering team can deliver a platform that matches - and in some respects exceeds - what commercial alternatives provide, with full control over data, customization, and integration.

---

*SysSight - Built for the people who keep the computers running.*
