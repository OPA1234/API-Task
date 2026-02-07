# API-Task

# Setvi RFQ API – Postman Collection Setup Guide

This repository contains a comprehensive Postman collection for testing the Setvi RFQ AI API endpoints, along with test data files and bug reports.

---

## 📁 Repository Contents

```
├── Setvi-API.postman_collection.json    # Main Postman collection
├── rfq-requests-data sets.zip                        # CSV/JSON data files for Collection Runner
├── bug-report-upload-free-text.md       # Bug report for upload-free-text endpoint
├── bug-report-upload-url-html.md        # Bug report for upload-url-html endpoint
```

---

## 🚀 Quick Start

### Prerequisites

- **Postman Desktop App** or **Postman Web** (recommended: Desktop for Runner features)
- Access to Setvi RFQ API Dev environment
- Provided RFQ API token 

---

## 📥 Step 1: Import the Postman Collection

### Option A: Import from File

1. Open Postman
2. Click **Import** (top-left corner)
3. Select **Upload Files**
4. Navigate to and select `Setvi-API.postman_collection.json`
5. Click **Import**
6. The collection **"Setvi API"** will appear in your Collections sidebar



## ⚙️ Step 2: Configure Environment Variables

### Create a New Environment

1. In Postman, click **Environments** (left sidebar)
2. Click **+** to create a new environment
3. Name it: `Setvi Dev`
4. Add the following variables:

| Variable Name | Type | Initial Value | Current Value |
|--------------|------|---------------|---------------|
| `devBaseUrl` | default | `https://intelligence-dev.setvi.com` | `https://intelligence-dev.setvi.com` |
| `token` | secret | `a7a91f48-0371-4680-b69d-7928d9c1c9ad` | `a7a91f48-0371-4680-b69d-7928d9c1c9ad` |

5. Click **Save**
6. Select **Setvi Dev** from the environment dropdown (top-right corner)


---

## 📦 Step 3: Extract Test Data Files

1. Locate `test-data.zip` in the repository
2. Extract the ZIP file to a folder on your computer
3. Inside you'll find JSON files for data-driven testing:
   - `rfq-upload-free-text-positive-data driven.json`
   - `rfq-upload-free-text-negative-empty text.json`
   - `rfq-upload-url-html-positive-data driven.json`
   - `rfq-upload-url-html-negative-empty-invalidURL.json`
   - `upload-url-html-negative-no-product.json`

> **Note:** Keep these files in an accessible location; you'll need to select them when running tests via Collection Runner.

---

## 🧪 Step 4: Run Individual Requests (Manual Testing)

### Test a Single Request

1. In the **Setvi API** collection, expand the folder:
   - `upload free text` or
   - `upload url html`
2. Select a request (e.g., `upload-free-text – positive – data-driven`)
3. Review the **Body** tab (variables like `{{text}}` or `{{url}}` are placeholders)
4. Click **Send**
5. Check the **Test Results** tab to see pass/fail status



## 🏃 Step 5: Run Tests with Collection Runner (Recommended)

Collection Runner allows you to execute multiple iterations using data files.

### Run Positive Flow Tests

1. Right-click the **Setvi API** collection → **Run collection**
2. In the Runner interface:
   - **Select requests:** Check only the positive requests you want to run:
     - `upload-free-text – positive – data-driven`
     - `upload-url-html – positive – data-driven`
3. Click **Select File** next to **Data**
4. Choose the corresponding JSON file:
   - For free-text: `upload-free-text-positive.json`
   - For URL: `upload-url-html-positive.json`
5. Click **Run Setvi API**
6. Review results:
   - Each row in the data file runs as a separate iteration
   - Test results show pass/fail for each assertion

### Run Negative Flow Tests

Follow the same process but select negative requests and their corresponding data files:

- `upload-free-text – negative – empty text` → `upload-free-text-negative.json`
- `upload-url-html – negative – empty or invalid URL` → `upload-url-html-negative-empty-invalid.json`
- `upload-url-html – negative – no product page` → `upload-url-html-negative-no-product.json`

---

## 📊 Collection Structure

```
Setvi API/
├── upload free text/
│   ├── upload-free-text – positive – data-driven
│   └── upload-free-text – negative – empty text
└── upload url html/
    ├── upload-url-html – positive – data-driven
    ├── upload-url-html – negative – empty or invalid URL
    └── upload-url-html – negative – no product page
```

---

## 🐛 Known Issues (Current Environment Blocker)

### ⚠️ Critical Bug: Unauthorized Response for Valid Requests

**Both RFQ endpoints currently return `Unauthorized` for all requests, including valid ones.**

#### Affected Endpoints
- `POST /api/rfq/upload-free-text`
- `POST /api/rfq/upload-url-html`

#### Symptoms
- **HTTP Status:** `200 OK` (misleading)
- **Response Body:**
  ```json
  {
    "error_code": "Unauthorized",
    "message": null,
    "isSuccess": false,
    "isCancelled": false
  }
  ```
- **Impact:** 
  - No `matchedItems` array is returned
  - All positive tests fail
  - Negative validation behavior is masked
  - Core RFQ matching functionality cannot be tested

#### Key Observations

1. **Swagger Example Values Issue**
   - Both endpoints on Swagger use **identical example request bodies**
   - The `upload-free-text` and `upload-url-html` Swagger examples are copy-pasted and don't reflect actual endpoint differences
   - This suggests incomplete API documentation or configuration

2. **Incorrect HTTP Status Code**
   - API returns `200 OK` with `error_code: "Unauthorized"` in the body
   - **Expected behavior:** Should return HTTP `401 Unauthorized` or `403 Forbidden` if token is invalid/expired
   - Current behavior violates REST API conventions (business errors wrapped in 200 response)

3. **Unclear Error Messaging**
   - Response contains `"message": null` — no explanation provided
   - **Cannot determine:**
     - Is the token expired?
     - Is the token invalid/malformed?
     - Is this an auth middleware misconfiguration?
     - Is the token missing required scopes/permissions?
   - Developers and QA cannot troubleshoot without clear error details

4. **Environment-Specific Issue**
   - Token `a7a91f48-0371-4680-b69d-7928d9c1c9ad` works in Swagger/curl (per assignment instructions)
   - Same token returns `Unauthorized` when used in Postman with identical headers/body
   - Suggests dev environment authorization middleware problem or token validation inconsistency

#### Bug Reports

Detailed bug reports are available in this repository:
- **Free-text endpoint:** `bug-report-upload-free-text.md`
- **URL endpoint:** `bug-report-upload-url-html.md`

Both reports include:
- Full reproduction steps
- Expected vs actual behavior
- Failed test assertions with exact error messages
- Severity/priority assessment

---

## ✅ Test Coverage

### Positive Flow Tests
- ✅ HTTP 200 status code
- ✅ No `Unauthorized` error in response
- ✅ `matchedItems` array present and non-empty
- ✅ At least one candidate matches expected product term
- ✅ Similarity/confidence scores present

### Negative Flow Tests
- ✅ Empty text input (free-text endpoint)
- ✅ Empty/invalid URL (URL endpoint)
- ✅ URL without product data (URL endpoint)
- ✅ Graceful error handling or empty candidates
- ✅ No `Unauthorized` returned for validation errors

---

## 🔧 Troubleshooting

### Tests Fail with "json is not defined"
- **Cause:** Test script missing response parsing code
- **Fix:** Each request's Tests tab should start with:
  ```javascript
  let json = {};
  try { json = pm.response.json(); } catch (e) { json = {}; }
  ```

### Tests Fail with "isUnauthorizedEnvelope is not defined"
- **Cause:** Helper function not defined in request Tests
- **Fix:** Tests should include:
  ```javascript
  function isUnauthorizedEnvelope(body) {
      return body && body.error_code === "Unauthorized" && body.isSuccess === false;
  }
  ```

### "Cannot find variable" errors in Runner
- **Cause:** Data file column names don't match variable names in request body
- **Fix:** Verify JSON data file keys match `{{variableName}}` in Body tab:
  - Free-text: `text`, `expectedItem`, `name`
  - URL: `url`, `expectedItem`, `name`

### Environment not selected
- **Symptom:** Variables like `{{devBaseUrl}}` appear as-is in requests
- **Fix:** Select **Setvi Dev** from environment dropdown (top-right)

---

## 📝 Notes for Reviewers

- **Data files:** All test data is in `rfq-requests-data sets.zip` for reproducibility
- **Self-contained tests:** Each request's Tests script is fully self-contained (no collection-level dependencies)
- **Data-driven design:** Requests use variables for Runner compatibility
- **Bug documentation:** Current blocker is documented with reproduction steps and expected fixes

---
