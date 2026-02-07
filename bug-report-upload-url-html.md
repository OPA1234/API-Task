# Bug Report – Upload URL HTML RFQ Returns Unauthorized

## Title
RFQ upload-url-html endpoint returns "Unauthorized" envelope for valid, authenticated requests

## Environment
- **Application:** Setvi RFQ AI API – Dev
- **Base URL:** `https://intelligence-dev.setvi.com`
- **Endpoint:** `POST /api/rfq/upload-url-html`
- **Authentication:** `Authorization: a7a91f48-0371-4680-b69d-7928d9c1c9ad` (valid RFQ token from provided curl/Swagger)
- **Test Client:** Postman collection "Setvi API"
- **Test Request:** `upload-url-html – positive – data-driven`

---

## Steps to Reproduce

1. Open the Postman request `upload-url-html – positive – data-driven`.
2. Configure request headers:
   - `Authorization: a7a91f48-0371-4680-b69d-7928d9c1c9ad`
   - `Content-Type: application/json`
   - `Accept: text/plain`
3. Set request body with valid product URL:

   ```json
   {
     "url": "https://www.webstaurantstore.com/choice-24-x-18-x-1-2-green-polyethylene-cutting-board/40724185GN.html",
     "topK": 3,
     "threshold": 0.5,
     "enablePrivateLabelRanking": false,
     "enableStockProductRanking": false,
     "enableVendorRanking": false,
     "enableProductRanking": false,
     "useOldReranking": true
   }
   ```

4. Execute the POST request to `/api/rfq/upload-url-html`.
5. Observe response body and test results.

## Actual Result

**Response:**
- HTTP Status: `200 OK`
- Response Body:

  ```json
  {
    "error_code": "Unauthorized",
    "message": null,
    "isSuccess": false,
    "isCancelled": false
  }
  ```

**Failed Test Assertions:**

1. **`URL positive: no Unauthorized in response body`**
   - Status: FAILED
   - Error: `AssertionError: Valid upload-url-html request returned Unauthorized in response body: expected true to be false`
   - Root cause: API returned `error_code: "Unauthorized"` despite valid authentication.

2. **`URL candidates list is present and not empty`**
   - Status: FAILED
   - Error: `AssertionError: Blocked: response is Unauthorized, no candidates list returned`
   - Root cause: No `candidates` array present in response due to Unauthorized envelope.

3. **`At least one URL candidate contains 'cutting board'`**
   - Status: FAILED
   - Error: `AssertionError: Blocked: response is Unauthorized, cannot verify candidates against expectedItem`
   - Root cause: Cannot validate product matching from URL because no candidate data is returned.

4. **`At least one URL candidate has similarity or confidence score`**
   - Status: FAILED
   - Error: `AssertionError: Blocked: response is Unauthorized, no similarity/confidence to verify`
   - Root cause: No ranking/confidence data available for verification.

**Impact:**
- Positive flow for URL-based RFQ matching is completely blocked.
- No product candidates are extracted or returned for valid product URLs.
- AI similarity/confidence scoring cannot be validated.
- URL parsing and product extraction functionality cannot be tested.
- Business logic for URL-to-product matching cannot be verified.

---

## Expected Result

For a valid, authenticated request with a properly formatted product page URL:

### Response Structure
- **HTTP Status:** `200 OK`
- **Response Body Should Include:**
  - `candidates`: array of matched product objects (non-empty)
  - Each candidate should contain:
    - `name` or `title`: product name/description extracted from URL
    - `similarity` or `confidence`: numeric score for ranking (e.g., 0.0–1.0)
    - Additional product metadata as per API specification

### Example Expected Response
```json
{
  "matchedItems": [
    {
      "id": "12345",
      "productName": "Choice 24\" x 18\" x 1/2\" Green Polyethylene Cutting Board",
      "similarity": 0.95,
      "confidence": 0.92,
      "vendor": "WebstaurantStore",
      "price": 24.99,
      "sourceUrl": "https://www.webstaurantstore.com/choice-24-x-18-x-1-2-green-polyethylene-cutting-board/40724185GN.html"
    },
    {
      "id": "67890",
      "productName": "24\" x 18\" Commercial Green Cutting Board",
      "similarity": 0.87,
      "confidence": 0.84,
      "vendor": "Restaurant Supply",
      "price": 22.50
    }
  ]
}
```

### Authorization Behavior
- **If token is valid:** Return matching candidates extracted from the provided URL (as described above)
- **If token is invalid/expired:** Return HTTP `401 Unauthorized` or `403 Forbidden` with clear error message in response body, NOT `200 OK` with business envelope error

---

## Severity & Priority

### Severity: **High**
- The defect completely blocks positive-path testing of the RFQ URL-based matching feature.
- No product candidates are returned from valid product URLs, preventing validation of:
  - URL parsing and product extraction accuracy
  - AI matching accuracy for URL-sourced products
  - Similarity/confidence scoring
  - Product ranking logic
  - Data completeness
- Core business functionality for URL-based RFQ cannot be demonstrated or tested in dev environment.

### Priority: **High**
- Must be resolved before:
  - Functional testing of RFQ URL matching can proceed
  - Integration testing with e-commerce platforms
  - UAT preparation and client demos
  - Performance/load testing of URL parsing and matching algorithms
  - Validation of web scraping and product extraction logic
- Blocks multiple test scenarios across positive and data-driven test suites.

---

## Additional Notes

- This issue is identical to the `upload-free-text` endpoint Unauthorized problem.
- Both endpoints use identical authorization mechanism and exhibit the same behavior.
- Both requests has the sam example value.
- Negative test scenarios (empty URLs, invalid URLs, no-product pages) also return `Unauthorized` envelope, masking proper validation behavior.
- Suggests a potential authorization middleware misconfiguration or environment-specific token validation issue in the dev deployment affecting all `/api/rfq/*` endpoints.
- The URL parsing and product extraction logic cannot be tested until authorization is resolved.

---

