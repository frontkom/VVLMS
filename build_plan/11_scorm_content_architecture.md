# 11. SCORM Content Serving & Sandboxing Architecture

> **ADR-006**: SCORM Content Delivery Architecture
> **Status**: Proposed
> **Date**: 2026-03-22
> **Supersedes**: None
> **Context**: This is the highest technical risk in the build. Many custom LMS projects fail here.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [The SCORM Communication Problem](#2-the-scorm-communication-problem)
3. [Architecture Decision: Dual-Mode Content Serving](#3-architecture-decision-dual-mode-content-serving)
4. [Content Upload & Extraction Pipeline](#4-content-upload--extraction-pipeline)
5. [CloudFront Multi-Origin Architecture](#5-cloudfront-multi-origin-architecture)
6. [SCORM Player Implementation](#6-scorm-player-implementation)
7. [Multi-Tenant Content Isolation](#7-multi-tenant-content-isolation)
8. [Security Architecture](#8-security-architecture)
9. [Performance & Scalability](#9-performance--scalability)
10. [cmi5/xAPI Future Path](#10-cmi5xapi-future-path)
11. [Implementation Plan](#11-implementation-plan)
12. [Risk Register](#12-risk-register)
13. [Sources](#13-sources)

---

## 1. Executive Summary

SCORM content runs as arbitrary HTML/JavaScript inside an iframe. The content must locate a JavaScript API object (`window.API` for SCORM 1.2, `window.API_1484_11` for SCORM 2004) by walking up the iframe's parent window hierarchy. This requires **same-origin** between the iframe and the LMS parent window, or a **postMessage bridge** to bypass that constraint.

### Decision

We use a **dual-mode architecture**:

| Mode | When | How | Security |
|------|------|-----|----------|
| **Primary: Same-origin** | All content (MVP) | Single CloudFront distribution serves both the app and SCORM content under the same domain. SCORM API discovery works natively. | CSP headers + signed cookies |
| **Enhanced: Cross-origin** | Untrusted content (Phase 2) | Content served from a separate origin (`scorm-content.veileder.no`). scorm-again's built-in `CrossFrameAPI`/`CrossFrameLMS` postMessage bridge handles communication. Iframe sandboxed without `allow-same-origin`. | Genuine iframe sandbox isolation |

### Why This Approach

- **Same-origin mode** is what Moodle, ILIAS, and Rustici Content Controller use. It works with 100% of SCORM packages, including non-standard ones. Zero risk of content compatibility issues.
- **Cross-origin mode** provides genuine security isolation for when we allow non-admin content uploads or integrate third-party content marketplaces. scorm-again already ships this capability — we just need to wire it up.
- **Both modes share** the same S3 storage, CloudFront distribution, upload pipeline, and database schema. The only difference is which CloudFront behavior serves the content and whether the postMessage bridge is activated.

---

## 2. The SCORM Communication Problem

### 2.1 SCORM 1.2 API Discovery

The SCORM 1.2 spec defines this algorithm (simplified):

```javascript
function findAPI(win) {
    let attempts = 0;
    while (win.API == null && win.parent != null && win.parent != win) {
        if (++attempts > 7) return null;  // Max 7 levels
        win = win.parent;
    }
    return win.API;
}

function getAPI() {
    let api = findAPI(window);
    if (!api && window.opener) {
        api = findAPI(window.opener);
    }
    return api;
}
```

**Rules:**
- Search `window.parent` chain for `window.API` object
- Maximum 7 levels deep
- Fall back to `window.opener` (for popup-based launch)
- API methods: `LMSInitialize`, `LMSFinish`, `LMSGetValue`, `LMSSetValue`, `LMSCommit`, `LMSGetLastError`, `LMSGetErrorString`, `LMSGetDiagnostic`

### 2.2 SCORM 2004 API Discovery

Same pattern but:
- Object name: `window.API_1484_11`
- Maximum **500** levels (up from 7)
- Checks `window.top.opener` instead of `window.opener`
- Methods renamed: `Initialize`, `Terminate`, `GetValue`, `SetValue`, `Commit`, `GetLastError`, `GetErrorString`, `GetDiagnostic`

### 2.3 The Same-Origin Constraint

Both algorithms access `window.parent.API` (or `window.parent.API_1484_11`). Browsers block this cross-origin:

```
// If LMS is at app.veileder.no and content at cdn.veileder.no:
// Chrome: "Blocked a frame with origin 'https://cdn.veileder.no'
//          from accessing a cross-origin frame."
// Firefox: "SecurityError: Permission denied to access property 'API'
//           on cross-origin object"
```

The `document.domain` workaround (setting both frames to the base domain) was deprecated and removed from Chrome 115+. It is not a viable path.

### 2.4 The postMessage Solution

`window.postMessage()` works cross-origin by design. scorm-again v3.0+ ships a built-in bridge:

**Parent (LMS) side — `CrossFrameLMS`:**
```javascript
import { Scorm12API } from 'scorm-again';
import CrossFrameLMS from 'scorm-again/cross-frame-lms';

const api = new Scorm12API({ autocommit: true });
new CrossFrameLMS(api, 'https://scorm-content.veileder.no');
```

**Child (content) side — `CrossFrameAPI`:**
```javascript
import CrossFrameAPI from 'scorm-again/cross-frame-api';
window.API = new CrossFrameAPI('https://app.veileder.no');
```

Key design: SCORM requires **synchronous** API calls, but postMessage is **async**. scorm-again solves this with a **cache-first pattern**:
- `GetValue` returns cached CMI data synchronously, then async-refreshes the cache
- `SetValue` updates the cache synchronously, then async-posts to the parent
- `Initialize`/`Commit`/`Terminate` return `"true"` synchronously, then fire background `getFlattenedCMI` to warm the cache
- Rate-limited to 100 messages/second with 30-second heartbeat for connection monitoring

---

## 3. Architecture Decision: Dual-Mode Content Serving

### 3.1 Options Evaluated

| # | Approach | SCORM Compat | Security | Complexity | Used By |
|---|----------|-------------|----------|------------|---------|
| A | Same-origin via CloudFront multi-origin | 100% | Lower (content is same-origin with LMS) | Low | Moodle, ILIAS, Rustici Content Controller |
| B | Cross-origin with postMessage bridge | ~95% (breaks non-standard content) | Higher (genuine iframe sandbox) | Medium | Rustici SCORM Cloud, Catalyst scormremote |
| C | Service worker (client-side extraction) | 100% (same-origin) | Lower | High | Experimental only |
| D | Reverse proxy through application server | 100% | Lower | Low | Small-scale LMS only |

### 3.2 Decision: A (MVP) + B (Phase 2)

**MVP: Option A — Same-origin via CloudFront multi-origin distribution**

Rationale:
1. Works with 100% of SCORM packages — no compatibility risk for Norwegian public sector's existing content library
2. Simplest implementation — CloudFront path-based behaviors, no bridge code needed
3. Industry standard — this is how Moodle, ILIAS, and Rustici's self-hosted Content Controller work
4. scorm-again's standard API injection (`window.API = new Scorm12API()`) works directly

**Phase 2: Add Option B — Cross-origin with postMessage bridge**

Rationale:
1. Needed when we open content upload to non-admin users or integrate content marketplaces
2. scorm-again already ships `CrossFrameAPI`/`CrossFrameLMS` — no custom bridge code needed
3. Enables `sandbox="allow-scripts allow-forms"` without `allow-same-origin` — genuine security isolation
4. Can be enabled per-content-package (admin-uploaded = same-origin, user-uploaded = cross-origin)

### 3.3 Options Rejected

**Option C (Service Worker)** rejected because:
- Client-side ZIP extraction of 100MB+ packages is slow and memory-intensive
- Adds complexity (service worker lifecycle, scope management, cache storage limits)
- No production LMS uses this approach

**Option D (Reverse proxy through app server)** rejected because:
- All SCORM asset requests (hundreds per package) flow through the application server
- No CDN caching — every image, CSS, JS file hits the backend
- Does not scale to thousands of concurrent users with large packages

---

## 4. Content Upload & Extraction Pipeline

### 4.1 Overview

```
┌──────────┐    ┌──────────────┐    ┌────────────────┐    ┌─────────────┐
│  Admin    │───>│  Upload API  │───>│  S3 Staging    │───>│  Celery     │
│  Browser  │    │  (FastAPI)   │    │  Bucket        │    │  Worker     │
└──────────┘    └──────────────┘    └────────────────┘    └──────┬──────┘
                                                                │
                    ┌───────────────────────────────────────────┘
                    │
                    v
            ┌───────────────┐    ┌──────────────┐    ┌──────────────┐
            │  1. Validate  │───>│  2. Scan     │───>│  3. Extract  │
            │  Manifest     │    │  (ClamAV)    │    │  to S3       │
            └───────────────┘    └──────────────┘    └──────┬───────┘
                                                           │
                                                           v
                                                    ┌──────────────┐
                                                    │  4. Parse    │
                                                    │  Manifest &  │
                                                    │  Update DB   │
                                                    └──────────────┘
```

### 4.2 Step-by-Step

#### Step 1: Upload

```python
# FastAPI endpoint
@router.post("/courses/{course_id}/content/scorm")
async def upload_scorm_package(
    course_id: UUID,
    file: UploadFile,
    tenant: TenantContext = Depends(get_tenant),
    user: User = Depends(require_permission("content.upload")),
):
    # Validate file size (max 500MB)
    if file.size > 500 * 1024 * 1024:
        raise HTTPException(413, "Package exceeds 500MB limit")

    # Validate file extension
    if not file.filename.endswith('.zip'):
        raise HTTPException(400, "SCORM package must be a ZIP file")

    # Generate version ID
    version = uuid4()
    staging_key = f"staging/{tenant.id}/{course_id}/{version}/{file.filename}"

    # Upload to S3 staging (multipart for large files)
    await s3.upload_fileobj(file, STAGING_BUCKET, staging_key)

    # Enqueue async processing
    task = process_scorm_package.delay(
        tenant_id=str(tenant.id),
        course_id=str(course_id),
        version=str(version),
        staging_key=staging_key,
        uploaded_by=str(user.id),
    )

    return {"task_id": task.id, "status": "processing"}
```

#### Step 2: Async Processing (Celery Worker)

```python
@celery.task(bind=True, max_retries=2)
def process_scorm_package(self, tenant_id, course_id, version, staging_key, uploaded_by):
    """
    Process uploaded SCORM package:
    1. Download from staging
    2. Validate imsmanifest.xml exists and is valid
    3. Scan with ClamAV
    4. Determine SCORM version (1.2 vs 2004)
    5. Extract all files to content bucket
    6. Set Content-Type metadata per file
    7. Parse manifest for SCO tree, entry point, metadata
    8. Update database (course_contents, scorm_runtime_data)
    9. Clean up staging
    """

    # Download ZIP from staging to temp
    with tempfile.TemporaryDirectory() as tmpdir:
        zip_path = os.path.join(tmpdir, "package.zip")
        s3.download_file(STAGING_BUCKET, staging_key, zip_path)

        # --- Validation ---
        with zipfile.ZipFile(zip_path, 'r') as zf:
            # Check for imsmanifest.xml at root
            names = zf.namelist()
            if 'imsmanifest.xml' not in names:
                raise ScormValidationError("Missing imsmanifest.xml at package root")

            # Check for zip bombs (total uncompressed size)
            total_size = sum(info.file_size for info in zf.infolist())
            if total_size > 2 * 1024 * 1024 * 1024:  # 2GB limit
                raise ScormValidationError("Uncompressed size exceeds 2GB limit")

            # Check for path traversal attacks
            for name in names:
                if name.startswith('/') or '..' in name:
                    raise ScormValidationError(f"Unsafe path in package: {name}")

        # --- ClamAV Scan ---
        scan_result = clamd.scan(zip_path)
        if scan_result and scan_result[zip_path][0] == 'FOUND':
            raise ScormSecurityError(f"Malware detected: {scan_result[zip_path][1]}")

        # --- Extract ---
        extract_dir = os.path.join(tmpdir, "extracted")
        with zipfile.ZipFile(zip_path, 'r') as zf:
            zf.extractall(extract_dir)

        # --- Parse Manifest ---
        manifest = parse_imsmanifest(os.path.join(extract_dir, 'imsmanifest.xml'))
        scorm_version = detect_scorm_version(manifest)
        entry_point = find_entry_point(manifest)
        sco_tree = build_sco_tree(manifest)  # For SCORM 2004 sequencing

        # --- Upload to Content Bucket ---
        content_prefix = f"{tenant_id}/scorm/{course_id}/v{version}"

        for root, dirs, files in os.walk(extract_dir):
            for filename in files:
                file_path = os.path.join(root, filename)
                relative_path = os.path.relpath(file_path, extract_dir)
                s3_key = f"{content_prefix}/{relative_path}"

                content_type = guess_content_type(filename)
                s3.upload_file(
                    file_path,
                    CONTENT_BUCKET,
                    s3_key,
                    ExtraArgs={
                        'ContentType': content_type,
                        'CacheControl': 'public, max-age=31536000, immutable',
                    }
                )

        # --- Update Database ---
        update_course_content(
            tenant_id=tenant_id,
            course_id=course_id,
            content_type=f'scorm_{scorm_version}',  # 'scorm_12' or 'scorm_2004'
            scorm_version=scorm_version,
            entry_point=entry_point,
            sco_tree=sco_tree,
            s3_prefix=content_prefix,
            file_count=len(names),
            total_size_bytes=total_size,
            uploaded_by=uploaded_by,
        )

        # --- Cleanup Staging ---
        s3.delete_object(Bucket=STAGING_BUCKET, Key=staging_key)
```

### 4.3 Manifest Parsing

```python
def detect_scorm_version(manifest_tree) -> str:
    """Detect SCORM version from imsmanifest.xml."""
    root = manifest_tree.getroot()
    nsmap = root.nsmap

    # SCORM 2004: has adlcp namespace with version
    for ns_uri in nsmap.values():
        if 'adlcp' in ns_uri:
            # Check schemaversion element
            schema_ver = root.find('.//{http://www.adlnet.org/xsd/adlcp_v1p3}schemaversion')
            if schema_ver is not None:
                return '2004'

    # SCORM 1.2: check for adlcp:scormtype attribute
    for resource in root.iter('{http://www.imsproject.org/xsd/imscp_rootv1p1p2}resource'):
        scorm_type = resource.get('{http://www.adlnet.org/xsd/adlcp_rootv1p2}scormtype')
        if scorm_type:
            return '12'

    # Fallback: look for schemaversion text
    for elem in root.iter():
        if 'schemaversion' in elem.tag.lower():
            text = (elem.text or '').strip()
            if '2004' in text:
                return '2004'
            if '1.2' in text:
                return '12'

    # Default to 1.2 (more common, more lenient)
    return '12'


def find_entry_point(manifest_tree) -> str:
    """Find the launch URL from the manifest."""
    root = manifest_tree.getroot()

    # Find the default organization's first item
    # Then find the resource it references
    organizations = root.find('.//{http://www.imsproject.org/xsd/imscp_rootv1p1p2}organizations')
    if organizations is None:
        # Try without namespace
        organizations = root.find('.//organizations')

    default_org = organizations.get('default', '')

    # Find first resource of type "sco"
    for resource in root.iter():
        if 'resource' in resource.tag.lower():
            href = resource.get('href')
            scorm_type = None
            for attr_name, attr_value in resource.attrib.items():
                if 'scormtype' in attr_name.lower():
                    scorm_type = attr_value

            if scorm_type and scorm_type.lower() == 'sco' and href:
                return href

    raise ScormValidationError("No launchable SCO found in manifest")
```

### 4.4 Content-Type Detection

```python
SCORM_CONTENT_TYPES = {
    '.html': 'text/html',
    '.htm': 'text/html',
    '.js': 'application/javascript',
    '.css': 'text/css',
    '.json': 'application/json',
    '.xml': 'application/xml',
    '.xsd': 'application/xml',
    '.dtd': 'application/xml-dtd',
    '.jpg': 'image/jpeg',
    '.jpeg': 'image/jpeg',
    '.png': 'image/png',
    '.gif': 'image/gif',
    '.svg': 'image/svg+xml',
    '.webp': 'image/webp',
    '.mp4': 'video/mp4',
    '.webm': 'video/webm',
    '.mp3': 'audio/mpeg',
    '.ogg': 'audio/ogg',
    '.wav': 'audio/wav',
    '.woff': 'font/woff',
    '.woff2': 'font/woff2',
    '.ttf': 'font/ttf',
    '.eot': 'application/vnd.ms-fontobject',
    '.pdf': 'application/pdf',
    '.swf': 'application/x-shockwave-flash',  # Legacy, won't run
}

def guess_content_type(filename: str) -> str:
    ext = os.path.splitext(filename)[1].lower()
    return SCORM_CONTENT_TYPES.get(ext, 'application/octet-stream')
```

### 4.5 S3 Storage Layout

```
s3://veileder-scorm-content/
├── {tenant_id}/
│   └── scorm/
│       └── {course_id}/
│           └── v{version_uuid}/          # Immutable version
│               ├── imsmanifest.xml
│               ├── index.html            # Entry point
│               ├── content/
│               │   ├── slide1.html
│               │   ├── slide2.html
│               │   └── ...
│               ├── assets/
│               │   ├── images/
│               │   ├── video/
│               │   └── audio/
│               └── scripts/
│                   ├── SCORM_API_wrapper.js  # From authoring tool
│                   └── ...
```

**Key design decisions:**
- **Versioned paths** (`v{uuid}`): No cache invalidation needed. Old versions remain accessible for learners mid-session. New uploads create new version paths.
- **Per-tenant prefix**: CloudFront signed cookies scoped to `/{tenant_id}/*` enforce tenant isolation.
- **Single bucket**: Simpler operations. Isolation via signed cookies + IAM policies, not bucket boundaries.
- **Immutable objects**: `Cache-Control: public, max-age=31536000, immutable` — SCORM content never changes after extraction.

---

## 5. CloudFront Multi-Origin Architecture

### 5.1 Architecture Diagram

```
                    ┌─────────────────────────────────────────┐
                    │         CloudFront Distribution          │
                    │         app.veileder.no                  │
                    │                                         │
                    │  Behaviors:                              │
                    │  ┌────────────────────────────────────┐  │
                    │  │ /api/*        → ALB (app servers)  │  │
                    │  │ /app/*        → ALB (app servers)  │  │
                    │  │ /scorm/*      → S3 (via OAC)       │  │
                    │  │ /_next/*      → ALB (Next.js)      │  │
                    │  │ Default (*)   → ALB (Next.js)      │  │
                    │  └────────────────────────────────────┘  │
                    └──────────┬──────────┬───────────────────┘
                               │          │
                    ┌──────────┘          └──────────┐
                    v                                v
             ┌──────────────┐              ┌──────────────────┐
             │     ALB      │              │   S3 Bucket      │
             │  (ECS Tasks) │              │   (SCORM content)│
             │              │              │   via OAC        │
             │ - FastAPI    │              │                  │
             │ - Next.js    │              │ /{tenant}/scorm/ │
             └──────────────┘              └──────────────────┘
```

### 5.2 Why This Works for SCORM

The browser sees a single origin: `https://app.veileder.no`. When the SCORM content iframe loads from `https://app.veileder.no/scorm/{tenant_id}/{course_id}/v{version}/index.html`, it is **same-origin** with the parent LMS page at `https://app.veileder.no/app/player`. The SCORM API discovery algorithm finds `window.parent.API` without any cross-origin errors.

CloudFront routes `/scorm/*` requests directly to S3 (via Origin Access Control) — the application servers never see SCORM asset requests. Full CDN caching, global edge delivery, automatic Gzip/Brotli compression for text assets.

### 5.3 CloudFront Configuration

#### Distribution Settings

```yaml
# Terraform/CDK pseudocode
cloudfront_distribution:
  aliases: ["app.veileder.no"]
  default_root_object: ""
  price_class: PriceClass_100  # EU + North America edges

  origins:
    - id: alb-origin
      domain_name: internal-alb.eu-north-1.elb.amazonaws.com
      custom_origin_config:
        origin_protocol_policy: https-only

    - id: s3-scorm-origin
      domain_name: veileder-scorm-content.s3.eu-north-1.amazonaws.com
      origin_access_control:
        signing_behavior: always
        signing_protocol: sigv4
        origin_type: s3

  behaviors:
    # SCORM content — S3 origin, signed cookies required
    - path_pattern: "/scorm/*"
      target_origin: s3-scorm-origin
      viewer_protocol_policy: https-only
      allowed_methods: [GET, HEAD, OPTIONS]
      cached_methods: [GET, HEAD]
      compress: true  # Auto Gzip/Brotli for text assets
      cache_policy: scorm-cache-policy
      trusted_key_groups: [scorm-signing-key]  # Signed cookies
      response_headers_policy: scorm-security-headers

    # API — ALB origin, no caching
    - path_pattern: "/api/*"
      target_origin: alb-origin
      viewer_protocol_policy: https-only
      allowed_methods: [GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE]
      cache_policy: CachingDisabled
      origin_request_policy: AllViewer

    # Next.js static assets — ALB origin, cached
    - path_pattern: "/_next/static/*"
      target_origin: alb-origin
      viewer_protocol_policy: https-only
      compress: true
      cache_policy: CachingOptimized

    # Default — ALB origin (Next.js SSR)
    - path_pattern: "*"
      target_origin: alb-origin
      viewer_protocol_policy: https-only
      cache_policy: CachingDisabled
      origin_request_policy: AllViewer

  # Cache policy for SCORM content
  scorm-cache-policy:
    min_ttl: 86400        # 1 day minimum
    default_ttl: 31536000  # 1 year
    max_ttl: 31536000      # 1 year
    # SCORM content is immutable (versioned paths) so aggressive caching is safe

  # Response headers for SCORM content iframe
  scorm-security-headers:
    content_security_policy: >-
      frame-ancestors 'self' https://app.veileder.no;
      default-src 'self';
      script-src 'self' 'unsafe-inline' 'unsafe-eval';
      style-src 'self' 'unsafe-inline';
      img-src 'self' data: blob:;
      media-src 'self' data: blob:;
      font-src 'self' data:;
      connect-src 'self' https://app.veileder.no;
    x_content_type_options: nosniff
    x_frame_options: SAMEORIGIN
    referrer_policy: strict-origin-when-cross-origin
```

### 5.4 Per-Tenant Subdomains (Future)

For tenant-branded deployments (`svv.veileder.no`, `nav.veileder.no`), the same CloudFront distribution handles multiple aliases. The `/scorm/*` behavior routes all tenants to the same S3 bucket — signed cookies are scoped to the tenant's path prefix, so `svv.veileder.no` cannot access `nav`'s content.

---

## 6. SCORM Player Implementation

### 6.1 Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│  Next.js Page: /app/courses/{courseId}/player                    │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  <ScormPlayer>  React Component                          │    │
│  │                                                          │    │
│  │  1. Fetches signed cookies via API                       │    │
│  │  2. Initializes scorm-again API on window                │    │
│  │  3. Configures event listeners (commit, terminate)       │    │
│  │  4. Renders iframe pointing to SCORM entry point         │    │
│  │                                                          │    │
│  │  ┌────────────────────────────────────────────────────┐  │    │
│  │  │  <iframe>  SCORM Content                           │  │    │
│  │  │                                                    │  │    │
│  │  │  Content finds window.parent.API                   │  │    │
│  │  │  Calls LMSInitialize(), LMSSetValue(), etc.        │  │    │
│  │  │  scorm-again handles all SCORM data model logic    │  │    │
│  │  │                                                    │  │    │
│  │  └────────────────────────────────────────────────────┘  │    │
│  │                                                          │    │
│  │  On commit/terminate → POST to /api/scorm/runtime        │    │
│  └──────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘
```

### 6.2 Launch Flow

```
1. User clicks "Start Course" on course detail page
2. Frontend calls POST /api/scorm/sessions
   - Backend validates enrollment, creates/resumes SCORM session
   - Backend generates CloudFront signed cookies scoped to tenant's content path
   - Returns: { session_id, launch_url, cookies, scorm_version, cmi_data }
3. Frontend sets signed cookies on the browser (via Set-Cookie response headers)
4. Frontend navigates to /app/courses/{id}/player
5. ScormPlayer component:
   a. Creates scorm-again API instance (Scorm12API or Scorm2004API)
   b. Loads existing CMI data from session (for resume)
   c. Attaches API to window.API or window.API_1484_11
   d. Registers event listeners for LMSCommit/Commit
   e. Renders iframe with src={launch_url}
6. SCORM content loads in iframe, discovers API, calls Initialize
7. On each Commit: ScormPlayer POSTs CMI data to /api/scorm/runtime
8. On Terminate: ScormPlayer POSTs final CMI data, updates completion status
```

### 6.3 React Component (Simplified)

```tsx
// components/ScormPlayer.tsx
'use client';

import { useEffect, useRef, useCallback, useState } from 'react';
import { Scorm12API, Scorm2004API } from 'scorm-again';

interface ScormPlayerProps {
  sessionId: string;
  launchUrl: string;
  scormVersion: '12' | '2004';
  existingCmiData?: Record<string, unknown>;
  onComplete?: (data: ScormCompletionData) => void;
}

export function ScormPlayer({
  sessionId,
  launchUrl,
  scormVersion,
  existingCmiData,
  onComplete,
}: ScormPlayerProps) {
  const iframeRef = useRef<HTMLIFrameElement>(null);
  const apiRef = useRef<Scorm12API | Scorm2004API | null>(null);
  const [status, setStatus] = useState<'loading' | 'running' | 'completed' | 'error'>('loading');

  // Persist CMI data to backend
  const persistCmiData = useCallback(async () => {
    if (!apiRef.current) return;

    const cmiData = apiRef.current.renderCMIToJSONObject();

    await fetch(`/api/scorm/sessions/${sessionId}/runtime`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        cmi_data: cmiData,
        // Extract indexed fields for query performance
        lesson_status: cmiData.cmi?.core?.lesson_status,
        completion_status: cmiData.cmi?.completion_status,
        success_status: cmiData.cmi?.success_status,
        score_raw: cmiData.cmi?.core?.score?.raw ?? cmiData.cmi?.score?.raw,
        score_scaled: cmiData.cmi?.score?.scaled,
        total_time: cmiData.cmi?.core?.total_time ?? cmiData.cmi?.total_time,
      }),
    });
  }, [sessionId]);

  useEffect(() => {
    // Create scorm-again API instance
    const ApiClass = scormVersion === '12' ? Scorm12API : Scorm2004API;
    const api = new ApiClass({
      autocommit: true,
      autocommitSeconds: 30,
      logLevel: process.env.NODE_ENV === 'development' ? 4 : 1,
    });

    // Load existing CMI data for resume
    if (existingCmiData) {
      api.loadFromJSON(existingCmiData);
    }

    // Listen for SCORM events
    api.on('LMSCommit', () => {
      persistCmiData();
    });

    api.on('LMSFinish', () => {
      persistCmiData().then(() => {
        setStatus('completed');
        if (onComplete) {
          onComplete({
            sessionId,
            cmiData: api.renderCMIToJSONObject(),
          });
        }
      });
    });

    // For SCORM 2004
    api.on('Commit', () => persistCmiData());
    api.on('Terminate', () => {
      persistCmiData().then(() => {
        setStatus('completed');
        if (onComplete) {
          onComplete({
            sessionId,
            cmiData: api.renderCMIToJSONObject(),
          });
        }
      });
    });

    // Attach to window for SCORM content to discover
    if (scormVersion === '12') {
      window.API = api;
    } else {
      window.API_1484_11 = api;
    }

    apiRef.current = api;
    setStatus('running');

    // Cleanup
    return () => {
      if (scormVersion === '12') {
        delete (window as any).API;
      } else {
        delete (window as any).API_1484_11;
      }
      apiRef.current = null;
    };
  }, [scormVersion, sessionId, existingCmiData, persistCmiData, onComplete]);

  // Handle beforeunload — persist on accidental close
  useEffect(() => {
    const handleBeforeUnload = () => {
      if (apiRef.current) {
        // Use sendBeacon for reliable delivery during page unload
        const cmiData = apiRef.current.renderCMIToJSONObject();
        navigator.sendBeacon(
          `/api/scorm/sessions/${sessionId}/runtime`,
          JSON.stringify({ cmi_data: cmiData, event: 'unload' })
        );
      }
    };

    window.addEventListener('beforeunload', handleBeforeUnload);
    return () => window.removeEventListener('beforeunload', handleBeforeUnload);
  }, [sessionId]);

  return (
    <div className="scorm-player" style={{ width: '100%', height: '100vh' }}>
      {status === 'loading' && <div className="scorm-loading">Loading course...</div>}
      <iframe
        ref={iframeRef}
        src={launchUrl}
        title="SCORM Content"
        style={{ width: '100%', height: '100%', border: 'none' }}
        allow="fullscreen; autoplay; microphone; camera"
        // No sandbox attribute in same-origin mode — see Section 8.3
      />
    </div>
  );
}
```

### 6.4 Backend Session & Runtime API

```python
# routes/scorm.py

@router.post("/scorm/sessions")
async def create_scorm_session(
    request: CreateScormSessionRequest,
    tenant: TenantContext = Depends(get_tenant),
    user: User = Depends(get_current_user),
    enrollment_repo: EnrollmentRepository = Depends(),
    scorm_repo: ScormRuntimeRepository = Depends(),
    content_repo: CourseContentRepository = Depends(),
):
    """Create or resume a SCORM session. Returns launch URL and signed cookies."""

    # Verify enrollment
    enrollment = await enrollment_repo.get_by_user_and_course(
        user_id=user.id, course_id=request.course_id
    )
    if not enrollment:
        raise HTTPException(403, "Not enrolled in this course")

    # Get content metadata
    content = await content_repo.get_scorm_content(request.course_id)
    if not content:
        raise HTTPException(404, "No SCORM content found for this course")

    # Get or create runtime data
    runtime = await scorm_repo.get_or_create(
        enrollment_id=enrollment.id,
        sco_id=request.sco_id or 'default',
        attempt_number=request.attempt or enrollment.current_attempt,
    )

    # Build launch URL
    launch_url = (
        f"/scorm/{tenant.id}/scorm/{content.course_id}"
        f"/v{content.version}/{content.entry_point}"
    )

    # Generate CloudFront signed cookies
    cookies = generate_signed_cookies(
        resource_path=f"/scorm/{tenant.id}/*",
        expires_minutes=480,  # 8-hour session
    )

    return ScormSessionResponse(
        session_id=runtime.id,
        launch_url=launch_url,
        scorm_version=content.scorm_version,
        cmi_data=runtime.cmi_data,  # For resume
        cookies=cookies,
    )


@router.put("/scorm/sessions/{session_id}/runtime")
async def update_scorm_runtime(
    session_id: UUID,
    data: UpdateScormRuntimeRequest,
    tenant: TenantContext = Depends(get_tenant),
    user: User = Depends(get_current_user),
    scorm_repo: ScormRuntimeRepository = Depends(),
):
    """Persist SCORM CMI data commit."""

    runtime = await scorm_repo.get(session_id)
    if not runtime:
        raise HTTPException(404)

    # Update runtime data
    await scorm_repo.update(session_id, {
        'cmi_data': data.cmi_data,
        'lesson_status': data.lesson_status,
        'completion_status': data.completion_status,
        'success_status': data.success_status,
        'score_raw': data.score_raw,
        'score_scaled': data.score_scaled,
        'total_time': data.total_time,
        'updated_at': datetime.utcnow(),
    })

    # Append to event log (for audit/debugging)
    await scorm_repo.append_event(
        session_id=session_id,
        event_type='commit',
        cmi_snapshot=data.cmi_data,
        user_id=user.id,
    )

    # Check for completion → trigger downstream workflows
    if data.lesson_status in ('completed', 'passed') or \
       data.completion_status == 'completed':
        await handle_scorm_completion(runtime, data, tenant)

    return {"status": "ok"}
```

### 6.5 Signed Cookie Generation

```python
import json
import base64
from datetime import datetime, timedelta
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import padding

def generate_signed_cookies(
    resource_path: str,
    expires_minutes: int = 480,
) -> dict:
    """Generate CloudFront signed cookies for SCORM content access."""

    expires = datetime.utcnow() + timedelta(minutes=expires_minutes)
    expires_epoch = int(expires.timestamp())

    # Custom policy allows wildcard paths
    policy = {
        "Statement": [{
            "Resource": f"https://app.veileder.no{resource_path}",
            "Condition": {
                "DateLessThan": {
                    "AWS:EpochTime": expires_epoch
                }
            }
        }]
    }

    policy_json = json.dumps(policy, separators=(',', ':'))
    policy_b64 = _cf_base64(policy_json.encode())

    # Sign with CloudFront private key
    private_key = serialization.load_pem_private_key(
        settings.CLOUDFRONT_PRIVATE_KEY.encode(), password=None
    )
    signature = private_key.sign(
        policy_json.encode(),
        padding.PKCS1v15(),
        hashes.SHA1(),  # CloudFront requires SHA1
    )
    signature_b64 = _cf_base64(signature)

    return {
        "CloudFront-Policy": policy_b64,
        "CloudFront-Signature": signature_b64,
        "CloudFront-Key-Pair-Id": settings.CLOUDFRONT_KEY_PAIR_ID,
    }

def _cf_base64(data: bytes) -> str:
    """CloudFront-safe base64 encoding."""
    return base64.b64encode(data).decode().replace('+', '-').replace('=', '_').replace('/', '~')
```

### 6.6 Multi-SCO Support (SCORM 2004)

SCORM 2004 packages can contain multiple SCOs (Sharable Content Objects) organized in an activity tree with sequencing rules. The `sco_tree` JSONB field in `course_contents` stores this structure:

```json
{
  "organization": "default-org",
  "items": [
    {
      "id": "module-1",
      "title": "Trafikksikkerhet grunnkurs",
      "type": "sco",
      "href": "module1/index.html",
      "children": []
    },
    {
      "id": "module-2",
      "title": "Førstehjelp ved trafikkulykker",
      "type": "sco",
      "href": "module2/index.html",
      "prerequisites": "module-1",
      "children": [
        {
          "id": "module-2-quiz",
          "title": "Kunnskapstest",
          "type": "sco",
          "href": "module2/quiz.html"
        }
      ]
    }
  ]
}
```

The frontend renders a navigation sidebar from this tree. Each SCO gets its own `scorm_runtime_data` row (keyed by `sco_id`). The player handles SCO transitions by:
1. Calling `Terminate` on current SCO
2. Persisting CMI data
3. Creating/loading `scorm_runtime_data` for next SCO
4. Updating iframe `src` to the next SCO's href
5. Calling `Initialize` for the new SCO

---

## 7. Multi-Tenant Content Isolation

### 7.1 Defense in Depth

```
Layer 1: CloudFront Signed Cookies
  └── Cookie policy scoped to /scorm/{tenant_id}/*
      Tenant A's cookie cannot authenticate requests for tenant B's path

Layer 2: S3 Origin Access Control (OAC)
  └── S3 bucket denies all direct access
      Only the CloudFront distribution (via OAC) can read objects
      No public S3 URLs exist

Layer 3: Application-Level Validation
  └── Before issuing signed cookies, backend verifies:
      - User belongs to tenant (from JWT/session)
      - User is enrolled in the specific course
      - Course belongs to the tenant
      - SCORM session is created with audit trail

Layer 4: IAM Policies for Backend
  └── ECS task roles scoped per-tenant for write operations:
      "Resource": "arn:aws:s3:::veileder-scorm-content/${tenant_id}/*"
```

### 7.2 Cookie Scoping

Each tenant's signed cookies are restricted to their content prefix:

```python
# For tenant "svv" (Statens vegvesen):
generate_signed_cookies(resource_path="/scorm/svv/*")
# Result: cookie only valid for https://app.veileder.no/scorm/svv/*

# For tenant "nav" (NAV):
generate_signed_cookies(resource_path="/scorm/nav/*")
# Result: cookie only valid for https://app.veileder.no/scorm/nav/*
```

A user at SVV cannot access NAV's SCORM content even if they manually construct the URL — CloudFront will reject the request because the signed cookie's policy doesn't cover `/scorm/nav/*`.

### 7.3 Upload Isolation

The upload pipeline uses the tenant context from the authenticated session:

```python
# The tenant_id comes from the JWT/TenantContext middleware
# It is NEVER derived from user input (URL path, form field, etc.)
staging_key = f"staging/{tenant.id}/{course_id}/{version}/{filename}"
content_prefix = f"{tenant.id}/scorm/{course_id}/v{version}"
```

---

## 8. Security Architecture

### 8.1 The Fundamental SCORM Security Problem

SCORM packages contain **arbitrary HTML and JavaScript** that executes in the learner's browser. As Rustici Software (the authors of the SCORM spec) acknowledge: "the LMS has no way to block evil JavaScript because it has no idea which JavaScript is evil."

Real-world impact: Tenable discovered XSS vulnerabilities in Rustici SCORM Engine (CVE-2022-21 series) that allowed full account takeover when SCORM content ran same-origin with the customer application.

**We cannot fully sandbox SCORM content while maintaining compatibility.** The `sandbox` attribute with `allow-scripts` + `allow-same-origin` (both required for same-origin SCORM) effectively negates the sandbox — embedded content can remove its own sandbox attribute.

### 8.2 Mitigation Strategy (MVP: Same-Origin Mode)

Since we cannot fully sandbox, we apply defense-in-depth:

#### 8.2.1 Upload Scanning Pipeline

```
ZIP Upload → ClamAV Scan → Manifest Validation → File Type Audit → Admin Review → Go Live
```

| Check | What | Blocks |
|-------|------|--------|
| ClamAV | Virus/malware scan of ZIP and extracted files | Known malware |
| Manifest validation | Parse imsmanifest.xml against SCORM schema | Malformed/weaponized manifests |
| File type audit | Verify extensions match expected MIME types; flag `.php`, `.jsp`, `.exe`, `.bat` | Server-side code injection |
| Path traversal check | Reject ZIP entries with `..` or absolute paths | Directory traversal attacks |
| ZIP bomb detection | Check ratio of compressed-to-uncompressed size | Denial of service |
| Size limits | 500MB compressed, 2GB uncompressed, 15,000 files | Resource exhaustion |

#### 8.2.2 CSP Headers on SCORM Content

Applied via CloudFront Response Headers Policy on the `/scorm/*` behavior:

```
Content-Security-Policy:
  frame-ancestors 'self' https://app.veileder.no;
  default-src 'self';
  script-src 'self' 'unsafe-inline' 'unsafe-eval';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: blob:;
  media-src 'self' data: blob:;
  font-src 'self' data:;
  connect-src 'self' https://app.veileder.no;

X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Referrer-Policy: strict-origin-when-cross-origin
```

Key protections:
- `frame-ancestors`: SCORM content can only be embedded by our LMS — prevents clickjacking and unauthorized embedding
- `connect-src 'self'`: Blocks outbound XHR/fetch to external servers (data exfiltration)
- `'unsafe-inline'` and `'unsafe-eval'` are required because Articulate Storyline, Adobe Captivate, and iSpring all generate inline scripts and use `eval()`

#### 8.2.3 Short-Lived Signed Cookies

- Cookies expire after 8 hours (single work session)
- Scoped to specific tenant path
- New cookies generated per course launch
- `Secure` flag (HTTPS only)
- `SameSite=Lax` (sent with top-level navigation and same-site requests)

#### 8.2.4 Admin-Only Upload (MVP)

In MVP, only users with `content.upload` permission (admin/content manager roles) can upload SCORM packages. This is the most effective mitigation — the attack surface is limited to trusted content from known authoring tools.

### 8.3 Enhanced Security Mode (Phase 2: Cross-Origin)

When we allow non-admin content uploads or integrate content marketplaces:

```
┌──────────────────────────────────────────────────────────────┐
│  app.veileder.no  (LMS parent frame)                         │
│                                                              │
│  window.API = scorm-again API                                │
│  CrossFrameLMS(api, 'https://scorm-content.veileder.no')     │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐  │
│  │  <iframe                                               │  │
│  │    src="https://scorm-content.veileder.no/..."         │  │
│  │    sandbox="allow-scripts allow-forms allow-popups"    │  │ ← No allow-same-origin!
│  │  >                                                     │  │
│  │                                                        │  │
│  │  Content cannot access:                                │  │
│  │  - window.parent.API (cross-origin blocked)            │  │
│  │  - LMS cookies, localStorage, DOM                      │  │
│  │  - Cannot remove its own sandbox attribute             │  │
│  │                                                        │  │
│  │  Content CAN only:                                     │  │
│  │  - Execute scripts                                     │  │
│  │  - Submit forms                                        │  │
│  │  - Use postMessage to communicate with parent          │  │
│  │                                                        │  │
│  │  Injected bridge script:                               │  │
│  │  window.API = new CrossFrameAPI('https://app...')      │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

This requires injecting the `CrossFrameAPI` bridge script into the SCORM package during the upload pipeline (inserted as a `<script>` tag in the entry point HTML before the content's own scripts). The bridge replaces the standard API discovery with postMessage-based communication.

**Trade-off**: ~5% of SCORM packages with non-standard API discovery may break. These can be flagged and switched to same-origin mode per-package.

### 8.4 What We Do NOT Need to Handle

- **Flash content**: Flash Player was discontinued Dec 2020. No modern browser supports it. We do not need to support Flash-based SCORM content. If uploaded, we display a clear error message suggesting HTML5 conversion.
- **Java applets**: Same — discontinued, no browser support.
- **ActiveX controls**: IE-only, not supported in any modern browser.

---

## 9. Performance & Scalability

### 9.1 Typical SCORM Package Profiles

| Type | Size | Files | Example |
|------|------|-------|---------|
| Text/quiz course | 1-10 MB | 20-100 | Compliance training, Artikulerte kurs |
| Interactive with images | 10-50 MB | 100-500 | Storyline courses with interactions |
| Video-heavy course | 50-500 MB | 100-1000 | Safety training with video demos |
| Complex simulation | 100 MB-1 GB | 1000+ | Equipment operation simulators |

SVV target: ~200 courses, mostly in the 10-100 MB range. Largest likely ~500 MB (safety videos).

### 9.2 CloudFront Performance

| Aspect | Configuration | Impact |
|--------|--------------|--------|
| Edge caching | `max-age=31536000, immutable` on all SCORM assets | After first user accesses a course, subsequent users get edge-cached response (<10ms) |
| Compression | Auto Gzip/Brotli for text assets | 60-80% size reduction for HTML/CSS/JS |
| HTTP/2 | Enabled by default | Multiplexing for parallel asset loading (SCORM packages have many small files) |
| Regional edges | PriceClass_100 (EU + North America) | Low latency for Norwegian users via Stockholm/Frankfurt edges |

### 9.3 Upload Processing Performance

| Package Size | Expected Processing Time | Strategy |
|-------------|------------------------|----------|
| < 10 MB | 2-5 seconds | Synchronous possible, but always async for consistency |
| 10-100 MB | 5-30 seconds | Celery worker, S3 multipart upload |
| 100-500 MB | 30-120 seconds | Celery worker, S3 Transfer Acceleration |
| > 500 MB | Rejected | 500 MB upload limit |

### 9.4 Concurrent User Scaling

- **SCORM content serving**: CloudFront handles unlimited concurrent users. Content is static assets served from edge cache. No application server involvement.
- **CMI data commits**: Each active learner commits ~every 30 seconds (autocommit) + on page transitions. For 1,000 concurrent learners: ~33 writes/second to `scorm_runtime_data` + `scorm_runtime_events`. PostgreSQL handles this easily.
- **Session creation**: One API call per course launch. Negligible load.

### 9.5 Storage Costs

```
Assumptions:
- 200 courses, average 50 MB each = 10 GB base
- 3 versions average per course = 30 GB total
- 10 tenants = 300 GB total S3 storage

S3 Standard (eu-north-1): $0.023/GB/month = ~$7/month
CloudFront data transfer: $0.085/GB (first 10 TB)
  - 1000 users × 50 MB average course × 2 courses/month = 100 GB/month
  - CloudFront cost: ~$8.50/month
  - Most assets cached at edge, so actual origin transfers much lower

Total infrastructure cost for SCORM content: ~$20-50/month for SVV-scale deployment
```

---

## 10. cmi5/xAPI Future Path

### 10.1 Why cmi5 Eliminates the Problem

cmi5 (the xAPI profile for LMS launch) uses **HTTP REST** instead of JavaScript API injection:

| Aspect | SCORM | cmi5/xAPI |
|--------|-------|-----------|
| Communication | `window.parent.API.SetValue()` | `fetch('https://lrs.veileder.no/xapi/statements', {...})` |
| Discovery | Walk iframe parent chain | Launch URL query parameters: `?endpoint=...&fetch=...&actor=...` |
| Same-origin needed? | Yes | No |
| Content hosting | Must be same-origin as LMS | Can be anywhere (CDN, different domain, native app) |

### 10.2 Implementation Plan

**Phase 2** (after MVP):
1. Add cmi5 launch endpoint that generates launch URLs with auth tokens
2. Support cmi5 package upload (different manifest format: `cmi5.xml`)
3. Route cmi5 statements to our SQL LRS
4. Add content type field: `scorm_12`, `scorm_2004`, `cmi5`
5. Same upload pipeline, but no postMessage bridge needed

**Long-term**: Encourage Norwegian public sector content creators to publish cmi5/xAPI content instead of SCORM. This eliminates the iframe security problem entirely.

---

## 11. Implementation Plan

### 11.1 Sprint Breakdown

| Sprint | Tasks | Deliverable |
|--------|-------|-------------|
| **S1** (2 weeks) | S3 bucket setup, CloudFront distribution with multi-origin, OAC, signed cookie generation | Infrastructure: SCORM content served from CloudFront |
| **S2** (2 weeks) | Upload API endpoint, Celery worker for extraction, manifest parsing, ClamAV integration | Upload pipeline: ZIP → validate → scan → extract → S3 |
| **S3** (2 weeks) | ScormPlayer React component, scorm-again integration, session API, CMI persistence | Player: SCORM content plays in browser, data persists |
| **S4** (1 week) | Multi-SCO navigation, resume (suspend data), attempt management | Full SCORM lifecycle support |
| **S5** (1 week) | SCORM-to-xAPI conversion, completion workflows, testing with real packages | End-to-end: upload → play → complete → certificate |

**Total: 8 weeks** (aligns with E3 Content Engine epic in build plan)

### 11.2 Testing Strategy

#### Test Packages

Source real SCORM packages from these authoring tools (covering 90%+ of market):

| Tool | SCORM Version | Known Quirks |
|------|--------------|--------------|
| Articulate Storyline 360 | 1.2 and 2004 | Uses `eval()`, inline scripts, localStorage |
| Articulate Rise | 1.2 and 2004 | Responsive HTML5, relatively clean |
| Adobe Captivate | 1.2 and 2004 | May use deprecated APIs, complex framesets |
| iSpring Suite | 1.2 and 2004 | Generally well-behaved |
| Lectora | 1.2 and 2004 | Uses frames within frames (depth > 2) |
| Adapt Framework | 1.2 and 2004 (via plugin) | Open-source, clean output |

#### ADL Test Suites

The ADL (Advanced Distributed Learning) provides official SCORM conformance test suites:
- SCORM 1.2 Conformance Test Suite v1.2.7
- SCORM 2004 4th Edition Conformance Test Suite

Run these as part of CI to verify our SCORM runtime implementation.

#### Automated Tests

```python
# Example: Test SCORM upload pipeline
async def test_scorm_upload_and_play():
    # Upload a known-good SCORM 1.2 package
    with open("tests/fixtures/scorm12_basic.zip", "rb") as f:
        response = await client.post(
            "/api/courses/test-course/content/scorm",
            files={"file": ("test.zip", f, "application/zip")},
        )
    assert response.status_code == 200
    task_id = response.json()["task_id"]

    # Wait for processing
    await wait_for_task(task_id)

    # Create session
    response = await client.post("/api/scorm/sessions", json={
        "course_id": "test-course",
    })
    assert response.status_code == 200
    session = response.json()
    assert session["scorm_version"] == "12"
    assert session["launch_url"].startswith("/scorm/")
    assert "CloudFront-Policy" in session["cookies"]

    # Simulate CMI commit
    response = await client.put(
        f"/api/scorm/sessions/{session['session_id']}/runtime",
        json={
            "cmi_data": {"cmi": {"core": {"lesson_status": "completed", "score": {"raw": "85"}}}},
            "lesson_status": "completed",
            "score_raw": 85,
        },
    )
    assert response.status_code == 200
```

---

## 12. Risk Register

| # | Risk | Impact | Likelihood | Mitigation |
|---|------|--------|------------|------------|
| R1 | SCORM packages from Norwegian authoring tools use non-standard API discovery | Content fails to load | Low (Norway mostly uses Articulate/iSpring) | Test with actual SVV content early; fallback to postMessage bridge per-package |
| R2 | Large packages (>200 MB) cause upload timeouts | Admin frustration | Medium | S3 multipart upload, async processing, progress feedback |
| R3 | SCORM packages contain malicious JavaScript | XSS, data theft | Low (admin-only upload in MVP) | ClamAV scan, CSP headers, admin review workflow; cross-origin mode in Phase 2 |
| R4 | CloudFront signed cookies blocked by browser privacy features | Content access denied | Low (first-party cookies) | Cookies are same-site (not third-party), so privacy features don't apply |
| R5 | SCORM content uses localStorage extensively | Data lost across sessions, conflicts between courses | Medium | Content is same-origin, so localStorage works; namespace by course_id if needed |
| R6 | Autocommit interval too aggressive for high-concurrency | Database write pressure | Low | 30-second autocommit is conservative; can batch on backend |
| R7 | Package extraction on upload is slow for large packages | Delayed availability | Medium | Async processing via Celery; show upload progress; set expectations in UI |
| R8 | SCORM 2004 sequencing rules are complex | Incorrect course navigation | Medium | Start with simple sequential; use scorm-again's built-in sequencing support |

---

## 13. Sources

### SCORM Specifications
- [SCORM API Discovery Algorithms](https://scorm.com/scorm-explained/technical-scorm/run-time/api-discovery-algorithms/)
- [Fixing the SCORM 2004 API Discovery Algorithm](https://scorm.com/wp-content/assets/old_articles/apifinder/SCORMAPIFinder.htm)
- [SCORM Cross-Domain JavaScript Restrictions](https://scorm.com/scorm-cross-domain/)

### scorm-again Library
- [scorm-again GitHub Repository](https://github.com/jcputney/scorm-again) — MIT license, full SCORM 1.2/2004 runtime
- [scorm-again npm Package](https://www.npmjs.com/package/scorm-again)
- [Cross-Frame API (postMessage bridge)](https://github.com/jcputney/scorm-again) — `CrossFrameAPI`/`CrossFrameLMS` classes

### Architecture References
- [Rustici Content Controller: S3 and CloudFront](https://docs.contentcontroller.com/self-hosting/S3andCloudFront.html) — same multi-origin pattern we use
- [Rustici Cross Domain (RXD)](https://rusticisoftware.com/products/rustici-cross-domain/) — commercial postMessage bridge
- [Serving SCORM from S3 and CloudFront in Moodle](https://developerck.com/serving-scorm-using-s3-and-cloudfront-in-moodle/)
- [SCORM Engine High Availability Architecture for AWS](https://support.scorm.com/hc/en-us/articles/115000031733)
- [Catalyst moodle-mod_scormremote](https://github.com/catalyst/moodle-mod_scormremote) — Moodle postMessage bridge plugin

### AWS Documentation
- [CloudFront Signed Cookies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-cookies.html)
- [CloudFront Origin Access Control](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
- [Multi-Tenant S3 Access Patterns](https://aws.amazon.com/blogs/apn/partitioning-and-isolating-multi-tenant-saas-data-with-amazon-s3/)
- [CloudFront Compression](https://cloudfix.com/blog/cloudfront-compression/)

### Security
- [SCORM Security — Some Perspective (Rustici)](https://scorm.com/blog/scorm-security-some-perspective/)
- [SCORM Security Risk (KnowQo)](https://knowqo.com/guide/scorm-security-risk)
- [XSS in Rustici SCORM Engine (Tenable CVE-2022-21)](https://www.tenable.com/security/research/tra-2022-21)
- [iframe sandbox security (Mozilla)](https://discourse.mozilla.org/t/can-someone-explain-the-issue-behind-the-rule-sandboxed-iframes-with-attributes-allow-scripts-and-allow-same-origin-are-not-allowed-for-security-reasons/110651)

### Alternative Approaches
- [MultiDomainScormCourse (postMessage PoC)](https://github.com/pituki/MultiDomainScormCourse)
- [Handling SCORM files on Frontend Using a Service Worker](https://victaz.dev/posts/handling-scorm-files-on-frontend/)
- [cmi5 vs SCORM Comparison](https://xapi.com/cmi5/comparison-of-scorm-xapi-and-cmi5/)
- [ADL SCORM-to-xAPI Wrapper](https://github.com/adlnet/SCORM-to-xAPI-Wrapper)

---

## Appendix A: Open-Source LMS SCORM Implementations Comparison

Research across 5 major open-source LMS platforms confirms our architecture decisions and reveals patterns we've incorporated.

### Cross-Platform Comparison

| Aspect | Moodle | Canvas | Open edX | ILIAS | Sakai |
|--------|--------|--------|----------|-------|-------|
| **Content storage** | Moodle file API (DB-backed) | SCORM Cloud (AWS S3) | Django storage (local/S3) | Local filesystem | Virtual FS over ZIP |
| **Content serving** | Same-origin via `pluginfile.php` | Cross-origin (SCORM Cloud CDN) | Same-origin proxy or direct S3 | Same-origin via WAC signed paths | Same-origin via Content Handler |
| **iframe sandbox** | **None** | N/A (external service) | **None** | **None** | **None** |
| **API bridge** | PHP-generated JS in parent frame | Provided by SCORM Cloud | JS XBlock with AJAX handlers | Concatenated JS with JSON data | ADL reference impl + Wicket AJAX |
| **Data persistence** | AJAX POST + Beacon API fallback | SCORM Cloud handles it | Batched async AJAX to handlers | JSON POST + sendBeacon | Synchronous Wicket form submit |
| **Multi-tenancy** | Context-scoped file areas | SCORM Cloud Applications | XBlock scope + hashed paths | Client-based directories + WAC | Site-scoped with RBAC |
| **SCORM 2004 sequencing** | Basic | Full (SCORM Cloud) | Basic | Full (dedicated impl) | ADL reference impl |

### Key Lessons Incorporated

| Lesson | Source | How We Apply It |
|--------|--------|-----------------|
| No platform uses iframe `sandbox` for SCORM | All 5 platforms | We don't sandbox in same-origin mode; use cross-origin + postMessage bridge in Phase 2 for genuine isolation |
| `navigator.sendBeacon()` for page unload | Moodle, ILIAS | Our ScormPlayer uses sendBeacon in beforeunload handler |
| Content hash/version for cache busting | Moodle (SHA1 + revision), Open edX (SHA1), ILIAS | We use UUID-versioned paths — even simpler, no hash computation |
| Same-origin content serving is the standard | Moodle, ILIAS, Open edX, Sakai | Our primary mode uses CloudFront multi-origin (same domain) |
| Delegating to SCORM Cloud works but has tradeoffs | Canvas | We build our own — avoids per-registration cost, data sovereignty issues, and external dependency |
| Cryptographic content access signing | ILIAS (WAC signed paths) | Our CloudFront signed cookies serve the same purpose with CDN-native implementation |
| Proxy-based serving doesn't scale | Open edX (proxy mode) | We serve directly from S3 via CloudFront — no application server in the content path |
| On-demand ZIP extraction is fragile | Sakai (virtual FS) | We extract on upload, store individual files in S3 with immutable cache headers |
