# Google Summer of Code 2025
# SW360


**Contributor:** Ritankar Saha <br>
**Email:** [ritankar.saha786@gmail.com](ritankar.saha786@gmail.com)  
**Github:** [ritankarsaha](https://github.com/ritankarsaha)   
**LinkedIn:** [Ritankar Saha](https://www.linkedin.com/in/ritankar-saha-8041b9289/)   
**Organization:** [SW360](https://github.com/eclipse-sw360/sw360)    
**Project Topic:** [Improving Integration of SW360 with FOSSology](https://github.com/eclipse-sw360/sw360/discussions/2868#discussioncomment-11914065) <br>
**Initial Proposal Document:** [Improving Integration of SW360 with FOSSology](https://docs.google.com/document/d/1sdvsYlP0pN1dBAgNgK1af0vCcMywP0gIvDyteEy3aZo/edit?usp=sharing)<br>
**Mentors:** [Gaurav Mishra](https://github.com/gmishx/), Amrit Kumar Verma, Katharina Ettinger <br>
**Project Duration:** 350 Hours   

---

## Abstract

This project aims to enhance the integration between SW360 and FOSSology by leveraging FOSSology’s extended REST API (v2). The current integration is limited to basic scan triggers and status checks; the proposed solution will significantly expand functionality by enabling direct source uploads, checksum-based file reuse, selective scanning agent configuration, and multi-format report generation (SPDX, CycloneDX, etc.). A modular connector within SW360 will manage secure token-based authentication, file uploads, scan orchestration, and report retrieval, ensuring robust error handling and performance optimization. By eliminating redundant scans, improving flexibility in agent selection, and providing diverse compliance reports, this integration will increase efficiency and user experience for compliance teams. The project has followed a phased 12-week roadmap, from refactoring the existing connector to implementing advanced features, testing, and documentation. The outcome will be a secure, scalable, and future-proof integration that strengthens open-source compliance workflows and enhances collaboration across the SW360–FOSSology ecosystem.

## Problem Statement and Motivation

SW360 currently integrates with FOSSology using a limited set of endpoints, which restricts its ability to provide efficient, flexible, and user-friendly compliance scanning. While functional, the existing “Send to FOSSology” feature lacks modern RESTful capabilities, leading to several challenges:

- Complexity & Workflow Barriers:
- Redundant file uploads increase scan times and resource usage.
- Limited automation prevents smooth CI/CD integration.
- Error handling and job tracking remain fragmented and manual.

### Educational & Usability Issues:

- Lack of configurable scanning agents restricts users to a “one-size-fits-all” scan.
- Limited reporting formats hinder compliance with diverse regulatory and organizational standards.
- Absence of intuitive UI elements reduces accessibility for non-technical stakeholders.

### Technical & Security Limitations:

- Reliance on outdated APIs (v1 endpoints) limits functionality and long-term maintainability.
- Current authentication relies on static tokens with insufficient user-level granularity.
- Inadequate error recovery and fallback mechanisms reduce system robustness.

### Project Objectives:
This project aims to overcome these limitations by upgrading SW360’s integration with FOSSology to use the extended REST API (v2). The enhancements will introduce direct source uploads, checksum-based reuse, user-configurable scanning agent selection, and multi-format report generation. By improving efficiency, flexibility, and security, the integration will empower compliance teams, streamline workflows, and future-proof SW360’s role as a central hub for open-source license compliance.


## Technical Implementation

### COER WORK TO BE DONE: IMPROVING INTEGRATION OF SW360 WITH FOSSOLOGY

### Phase 1: V2 Endpoints Migration (Weeks 1-2)

**Main Deliverables**
- Refactoring the SW360 FOSSology connector to utilize RESTful endpoints (v2).
- Implemented combined upload + scan initiation (automatic scan starts after successful upload).
- Implemented automatic job scheduling & status polling using v2 job/status semantics.
- Implemented (initial) automatic report generation flow after scans (logic present, some issues still under investigation).
- Ensured attachments from SW360 are correctly uploaded to FOSSology and scans run successfully via v2.

**Milestones to be reached**
- Basic integration is operational using v2 endpoints, demonstrated by triggering a scan and retrieving an SPDX report linked to a release.

**Architecture & Code Refactoring**

- Rewrote FossologyRestClient.java for v2 endpoint structures (request/response handling, multipart upload).
- Refactored FossologyHandler.java core logic to use new client + workflow (upload → scan → (report)).
- Updated FossologyRestConfig / FossologyConfig for v2 base URL and capability checks.
- Introduced FossologyV2Models.java containing strongly‑typed response / request models for new API payloads.

**API & Workflow Changes**

- Upload Process: Switched to multipart form data with immediate scan trigger.
- Job Management: Enhanced extraction & tracking of job IDs returned by v2 API.
- Scan Status Monitoring: Updated polling logic to consume new v2 status schema.
- File Search: Updated to leverage /filesearch v2 endpoint.
- Report Flow: Added v2-compatible report generation trigger & retrieval pipeline (issue remains in automatic final attachment step).

**Error Handling & Robustness**

- Improved exception propagation with clearer SW360Exception pathways.
- Added endpoint/version validation at connection setup (fail fast if v2 API not reachable / wrong base path).
- More explicit handling of asynchronous job states (pending, running, completed, failed).

**Data / Model Layer Enhancements**

New POJOs / model classes for:
- Upload responses (including scan job references).
- Job status & progress (multi-stage).
- File search results.
- Report generation and retrieval metadata.
- Migration / Backward Compatibility

Deprecated v1 usage entirely (v1 endpoints no longer referenced) and successfully migrated to v2 endpoints.

PR Work and API Testing

- **Link to PR:** [PR for Phase 1 - v2 Endpoints Refactor](https://github.com/eclipse-sw360/sw360/pull/3264)
- **Live Demo:** [Working Demo Demonstration](https://github.com/user-attachments/assets/4b6e6f21-2e0c-41d5-921c-70abd1e98cd2)
- **Proper API Endpoint Testing** [Proper API Testing and Description](https://github.com/eclipse-sw360/sw360/pull/3264#issue-3215689391)
Please visit the last link for more details on API testing and Changelog and Issue Description. 

### Phase 2: Checksum Computation, Filesearch Functionality and Reuse Functionality (Weeks 2-7)

**Main Deliverables**
- Integrating pre-upload checksum computation (e.g., SHA-1) into the SW360 REST API.
- Implementing file search functionality using the /filesearch v2 endpoint.
- Updating the SW360 data model to store file checksums and corresponding FOSSology upload IDs.
- Enhancing the reuse functionality by leveraging FOSSology's "Reuser" agent capabilities, ensuring that clearing decisions and related metadata are reused as described.

**Milestones to be Reached**
- Duplicating detection and reuse are verified, enabling reuse of existing scan data for identical file content without needing immediate UI updates

**Main Work Done in Short**
- Generic attachment upload with automatic md5/sha1/sha256 hashing.
- Component‑scoped attachment upload (multipart + JSON metadata).
- Uniform attachment response enrichment (hash flags, HAL links).
- Global batch SHA1 file search returning occurrences (uploadId, folderId, filename, hashes, size).
- Release‑scoped SHA1 file search.
- Component‑scoped SHA1 file search.
- Advanced file search combining: sha1 list, filename, (optional) content term, uploadId filter, maxResults, includeContent toggle.
- File search result caching layer.
- Explicit cache invalidation endpoint.
- Request/response model additions for advanced search + attachment metadata.

A bunch of new API Endpoints have been implemented in this PR, some of them are below - 
- POST /resource/api/attachments
- POST /resource/api/components/{componentId}/attachments
- POST /resource/api/filesearch/sha1
- POST /resource/api/releases/{releaseId}/filesearch
- POST /resource/api/components/{componentId}/filesearch
- POST /resource/api/filesearch/advanced
- POST /resource/api/filesearch/cache/clear
- GET /resource/api/attachments/{attachmentId}
- DELETE /resource/api/attachments/{attachmentId}
- GET /resource/api/components/{componentId}/attachments
- GET /resource/api/releases/{releaseId}/attachments
- Possibly PATCH/PUT endpoints for updating attachment metadata
- GET /resource/api/filesearch/sha1 (single SHA via query param) or GET variant (if implemented)
- POST /resource/api/components/{componentId}/filesearch/advanced (scoped advanced search variant)
- POST /resource/api/releases/{releaseId}/filesearch/advanced
- GET /resource/api/filesearch/cache/status (if you exposed cache metrics)

PR Work and API Testing :-

- **Link to PR:** [PR for Phase 2 - Checksum, file search functionality and reuse functionality](https://github.com/eclipse-sw360/sw360/pull/3297)
- **Proper API Endpoint Testing** [Proper API Testing and Description](https://github.com/eclipse-sw360/sw360/pull/3297#issue-3264636301)
Please visit the last link for more details on API testing and Changelog and Issue Description and proper API Endpoints Responses.

### Phase 3: User Configurable Scanning Agent

**Main Deliverables**
- Enhancing the SW360 REST API to accept user-configurable scanning agent options via API calls.
- Modifying backend logic to construct appropriate JSON payloads for triggering scans.
- Validating the integration by triggering scans with various agent configurations.

**Milestone to be Reached**
- Customizable scan options are available via the SW360 REST API. (UI changes can be deferred to a later phase.)

**Work Done**

### REST API Enhancements
- New Endpoint: /api/releases/{id}/triggerFossologyProcessWithOptions - POST endpoint for triggering FOSSology scans with custom agent
configurations
- Request Model: ScanOptionsRequest class with comprehensive validation and default option management
- Validation Framework: ScanOptionsValidator with comprehensive validation logic and meaningful error messages
- Configurable Scan Options
- Analysis Agents: Support for bucket, copyrightEmailAuthor, ecc, ipra, keyword, mime, monk, nomos, ojo, pkgagent, reso
- Decider Agents: Support for nomosMonk, bulkReused, newScanner, ojoDecider
- Reuse Options: Support for reuseMain, reuseEnhanced, reuseReport, reuseCopyright

### Backend Integration
- Thrift Interface: New ScanOptions struct and processWithScanOptions() service method
- FOSSology Handler: Enhanced with custom scan options support throughout the workflow
- REST Client: JSON payload construction for FOSSology v2 API with proper option mapping

### Service Layer
- Release Service: New executeFossologyProcessWithOptions() method with async processing
- Option Conversion: Automatic conversion between REST and Thrift data models
- Workflow Integration: Seamless integration with existing FOSSology process management

PR Work and API Testing :-

- **Link to PR:** [PR for Phase 3 - Checksum, Configurable Scan Options via REST API](https://github.com/eclipse-sw360/sw360/pull/3341)
- **Proper API Endpoint Testing** [Proper API Testing and Description](https://github.com/eclipse-sw360/sw360/pull/3341#issue-3317458982)
Please visit the last link for more details on API testing and Changelog and Issue Description and proper API Endpoints Responses.


### Technology Stack
- **Frontend:** React
- **Backend:** Java (SpringBoot)
- **Database:** CouchDB
- **Others** Apache Tomcat, Maven, FOSSology


## Acknowledgments

This project was completed under the mentorship of Gaurav Mishra, Katherina Ettinger, Rudra Chopra and Amrit Kumar Verma as part of Google Summer of Code 2025 with SW360. The successful implementation reflects the collaborative nature of open-source educational technology development and the commitment of the SW360 community to accessible learning tools.

---

**Project Completion:** Google Summer of Code 2025  
**Organization:** SW360
