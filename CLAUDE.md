# CLAUDE.md - AI Assistant Guide for Exact PHP Client

This document provides a comprehensive guide for AI assistants working with the exact-php-client codebase.

## Project Overview

**Name:** picqer/exact-php-client
**Type:** PHP Library
**Purpose:** PHP client library for the Exact Online API
**License:** MIT
**Minimum PHP Version:** 7.4.0
**Primary Dependencies:**
- guzzlehttp/guzzle ~6.0|~7.0 (HTTP client)
- ext-json (JSON extension)
- phpunit/phpunit ^9.6 (dev, for testing)
- phpstan/phpstan ^2.1 (dev, for static analysis)

**Repository:** https://github.com/picqer/exact-php-client
**Maintainer:** Picqer (Stephan Groen)

## Codebase Structure

```
exact-php-client/
├── src/Picqer/Financials/Exact/    # Main source code
│   ├── Connection.php               # OAuth2 connection manager
│   ├── Model.php                    # Abstract base class for all entities
│   ├── ApiException.php            # Custom exception class
│   ├── Persistance/                # Persistence traits
│   │   ├── Storable.php            # CRUD operations trait
│   │   └── Downloadable.php        # File download trait
│   ├── Query/                      # Query-related traits
│   │   └── Findable.php            # Find/filter operations trait
│   ├── Webhook/                    # Webhook functionality
│   │   └── Authenticatable.php    # Webhook authentication trait
│   └── [Entity Files]              # ~74 entity classes (Account.php, Item.php, etc.)
├── tests/                          # PHPUnit tests
│   └── EntityTest.php              # Entity structure validation tests
├── example/                        # Example implementations
│   └── example.php                 # OAuth flow and API usage example
├── composer.json                   # Composer configuration
├── phpunit.xml.dist               # PHPUnit configuration
├── userscript.js                  # Greasemonkey script for entity generation
└── README.md                      # User-facing documentation
```

### Directory Organization

**Entity Files:** All entity classes reside directly in `src/Picqer/Financials/Exact/` (flat structure, not organized by domain)

**Total Entities:** 74+ entity classes representing Exact Online API resources

## Architecture Patterns

### 1. Entity Model Pattern

All entities follow a consistent structure:

```php
<?php
namespace Picqer\Financials\Exact;

/**
 * Class EntityName
 * @package Picqer\Financials\Exact
 * @see [Exact API Documentation URL]
 *
 * @property Type $PropertyName Description
 * ... (all properties documented in docblock)
 */
class EntityName extends Model
{
    use Query\Findable;      // Provides find(), filter(), get() methods
    use Persistance\Storable; // Provides save(), insert(), update(), delete() methods

    protected $primaryKey = 'ID';  // Can be overridden (default: 'ID')

    protected $fillable = [
        'Property1',
        'Property2',
        // ... all fillable properties
    ];

    protected $url = 'endpoint/path';  // API endpoint (without /api/v1/{division}/)
}
```

### 2. Trait-Based Composition

The library uses PHP traits to add capabilities to entities:

- **Query\Findable**: Provides querying methods (find, filter, get, first, findId)
- **Persistance\Storable**: Provides CRUD operations (save, insert, update, delete)
- **Persistance\Downloadable**: Provides file download capabilities
- **Webhook\Authenticatable**: Provides webhook signature verification

### 3. Connection Management

The `Connection` class handles:
- OAuth2 authentication flow
- Token management (access token, refresh token)
- Automatic token refresh
- HTTP request/response handling via Guzzle
- Division (administration) management
- Middleware support for custom request/response handling

### 4. Key Architectural Decisions

1. **No Active Record Pattern:** Entities don't automatically persist; developers must call `save()` explicitly
2. **Lazy Loading:** Related collections use deferred loading via `__deferred` markers
3. **Division Context:** API calls include division (administration) ID in URL by default
4. **Immutable URLs:** Entity URLs are static strings, not computed
5. **Property Access:** Uses PHP magic methods (`__get`, `__set`) for flexible property access

## Development Workflows

### Entity Creation Workflow

When adding a new entity to the library:

#### Method 1: Using the Userscript (Recommended)

1. Install the `userscript.js` in Greasemonkey or Tampermonkey
2. Navigate to the Exact API documentation page for the entity
3. The script generates the complete PHP class automatically
4. Copy the generated code to a new file in `src/Picqer/Financials/Exact/`
5. Add a corresponding test method to `tests/EntityTest.php`

#### Method 2: Manual Creation

1. Create a new PHP file in `src/Picqer/Financials/Exact/`
2. Follow the entity pattern shown above
3. Reference the Exact API documentation for:
   - Property names and types
   - API endpoint URL
   - Primary key (usually 'ID', sometimes 'Code')
4. Add all properties to:
   - PHPDoc `@property` annotations
   - `$fillable` array
5. Add appropriate traits (`Findable`, `Storable`)
6. Add test method to `tests/EntityTest.php`

**Important:** Entity files should be named in PascalCase singular form (e.g., `Account.php`, not `Accounts.php`)

### Testing Workflow

**Test Framework:** PHPUnit 9.6+

**Running Tests:**
```bash
composer install
./vendor/bin/phpunit
```

**Running Static Analysis:**
```bash
./vendor/bin/phpstan analyse --memory-limit=2G
```

Note: PHPStan is configured but may report errors due to legacy PHPDoc type annotations (Guid, Int32, etc.) from the Exact API documentation.

**Test Philosophy:**
- Tests validate entity structure, not API functionality
- Each entity must have a corresponding test method
- Tests verify: instantiability, required properties (`fillable`, `url`), namespace, parent class

**Adding Tests for New Entities:**

```php
public function testNewEntityEntity()
{
    $this->performEntityTest(\Picqer\Financials\Exact\NewEntity::class);
}
```

### Release Workflow

1. Update `CHANGELOG.md` with changes
2. Follow semantic versioning (v3.x.x currently)
3. Merge to master via pull request
4. Travis CI runs automated tests
5. Tag release on GitHub

## Key Conventions

### Naming Conventions

1. **Entity Classes:** PascalCase, singular (e.g., `SalesInvoice`, `Account`)
2. **Properties:** PascalCase matching Exact API (e.g., `AddressLine1`, `CustomerCode`)
3. **Methods:** camelCase (e.g., `setDivision()`, `findWithSelect()`)
4. **Namespaces:** Follow PSR-4 (`Picqer\Financials\Exact`)

### Code Style Conventions

1. **Indentation:** 4 spaces (not tabs)
2. **Braces:** Opening brace on same line for methods/classes
3. **Visibility:** Always declare property/method visibility
4. **Docblocks:** Required for classes; include `@package`, `@see`, and all `@property` annotations
5. **Type Hints:** Used where appropriate, considering PHP 5.5 compatibility

### Property Conventions

1. **Fillable Properties:** All properties that can be set by users must be in `$fillable` array
2. **Property Types:** Document types in PHPDoc using Exact's type system:
   - `Guid` - UUID strings
   - `String` - Text
   - `Int32`, `Int64` - Integers
   - `Double` - Floating point numbers
   - `DateTime` - Dates
   - `Boolean`, `Byte` - Booleans
   - `Binary` - Binary data
   - Entity names for relationships (e.g., `BankAccounts`)
3. **Primary Keys:** Usually `ID` (Guid), sometimes `Code` (String)
4. **Readonly Properties:** Properties like `Created`, `Modified`, `Creator`, `Modifier` are in fillable but typically set by API

### API Endpoint Conventions

1. **URL Format:** Relative path without `/api/v1/{division}/` prefix
   - Example: `'crm/Accounts'`, `'salesinvoice/SalesInvoices'`
2. **Pluralization:** Follow Exact API's pluralization (usually plural in URL)
3. **GUID Format:** GUIDs in filters/requests use format `guid'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'`

## Common Development Tasks

### Adding a New Entity

**Steps:**

1. Identify the Exact API endpoint from documentation
2. Use userscript.js OR manually create class file
3. Ensure all properties are in both PHPDoc and `$fillable`
4. Add appropriate traits (usually both `Findable` and `Storable`)
5. Set correct `$url` and `$primaryKey`
6. Add test to `tests/EntityTest.php`
7. Run `phpunit` to verify
8. Update CHANGELOG.md if releasing

**Example:**

```php
<?php
namespace Picqer\Financials\Exact;

/**
 * Class ShippingMethod
 * @package Picqer\Financials\Exact
 * @see https://start.exactonline.nl/docs/HlpRestAPIResourcesDetails.aspx?name=LogisticsShippingMethods
 *
 * @property Guid $ID Primary key
 * @property Boolean $Active Is method active
 * @property String $Code Shipping method code
 * @property String $Description Description
 */
class ShippingMethod extends Model
{
    use Query\Findable;
    use Persistance\Storable;

    protected $fillable = [
        'ID',
        'Active',
        'Code',
        'Description',
    ];

    protected $url = 'logistics/ShippingMethods';
}
```

### Extending Existing Entities

When adding properties to existing entities:

1. Check Exact API documentation for exact property name and type
2. Add `@property` annotation to class docblock
3. Add property name to `$fillable` array (alphabetical order not required but appreciated)
4. Test that entity still passes `EntityTest`
5. Update CHANGELOG.md

### Adding New Query Methods

To add specialized query methods:

1. Add to `Findable` trait if applicable to all entities
2. OR add as instance method to specific entity class
3. Use `$this->connection()->get()` with appropriate filters
4. Return `$this->collectionFromResult($result)` for collections
5. Return `new self($this->connection(), $result)` for single entities

### Modifying Connection Behavior

When modifying `Connection.php`:

1. Preserve backward compatibility
2. OAuth2 flow is sensitive - test thoroughly
3. Token refresh happens automatically in `createRequest()`
4. Use `tokenUpdateCallback` for custom token persistence
5. Support middleware via `insertMiddleWare()` for custom behavior

## Important Implementation Details

### OAuth2 Authentication Flow

**Three-Step Process:**

1. **Authorization:** Redirect user to Exact for login (`redirectForAuthorization()`)
2. **Callback:** Receive authorization code, exchange for tokens
3. **Connect:** Use tokens for API requests, auto-refresh as needed

**Token Management:**
- Access tokens expire (default: 600 seconds)
- Refresh tokens are used to get new access tokens
- Library automatically refreshes when `tokenHasExpired()` returns true
- Use `setTokenUpdateCallback()` to persist updated tokens

### Division (Administration) Handling

- By default, uses user's current division
- Can be set explicitly with `setDivision($divisionId)`
- Included in API URLs as `/api/v1/{division}/{endpoint}`
- Some operations (like `Me`) don't use division in URL

### Deferred/Lazy Loading

Collections are lazy-loaded when accessed:

```php
$invoice = $invoices->find($id);
$lines = $invoice->SalesInvoiceLines;  // Triggers lazy load
```

Implementation in `Model::lazyLoad()`:
- Checks for `__deferred` marker in attributes
- Fetches related collection via API
- Caches in `$deferred` array

### Filter Syntax

Uses OData filter syntax:

```php
// Basic filter
$accounts->filter("Name eq 'Example Corp'");

// With GUID
$accounts->filter("ID eq guid'12345678-1234-1234-1234-123456789abc'");

// Comparison operators
$items->filter("UnitPrice gt 100");

// Logical operators
$items->filter("UnitPrice gt 100 and IsSalesItem eq true");

// With select and expand
$accounts->filter("IsSales eq true", "BankAccounts", "Code,Name");

// With system query options
$entries->filter("", "", "EntryID,FinancialYear", ['$top' => 1]);
```

### Error Handling

**ApiException:**
- Thrown for HTTP errors and API errors
- Includes HTTP status code and Exact's error message
- Parse from `error.message.value` in JSON response

**Common Errors:**
- `400 Bad Request`: Missing `$select` or `$top=1` on some endpoints
- `401 Unauthorized`: Token expired or invalid
- `404 Not Found`: Entity doesn't exist or wrong endpoint
- `500 Internal Server Error`: Exact API issue

## Testing Guidelines

### Test Structure

All tests in `tests/EntityTest.php` follow this pattern:

```php
public function testEntityNameEntity()
{
    $this->performEntityTest(\Picqer\Financials\Exact\EntityName::class);
}
```

The `performEntityTest()` method validates:
- ✓ Class is instantiable
- ✓ Has `fillable` property
- ✓ Has `url` property
- ✓ Namespace is correct
- ✓ Extends `Model` class

### Running Tests

```bash
# Install dependencies
composer install

# Run all tests
./vendor/bin/phpunit

# Run with coverage (requires xdebug)
./vendor/bin/phpunit --coverage-html coverage/
```

### CI/CD

- **Platform:** GitHub Actions
- **PHP Versions Tested:** 7.4, 8.0, 8.1, 8.2, 8.3
- **Process:** Validate composer.json → Install dependencies → Run PHPUnit
- **Configuration:** `.github/workflows/test.yml`
- **Legacy:** `.travis.yml` (deprecated, replaced by GitHub Actions)

## Troubleshooting Common Issues

### 1. "Please add a $select or a $top=1 statement"

**Cause:** Some Exact endpoints require limiting results
**Solution:**

```php
// Option 1: Use $select
$test->filter('', '', 'EntryID,FinancialYear');

// Option 2: Use $top
$test->filter('', '', '', ['$top' => 1]);
```

### 2. "Bad Request" on Token Exchange

**Cause:** Authorization code used more than once
**Solution:** Authorization codes are single-use; restart OAuth flow

### 3. Division Not Set

**Cause:** Missing or incorrect division ID
**Solution:**

```php
// Let library auto-detect from user
$connection->connect();

// OR explicitly set
$connection->setDivision(123456);
```

### 4. Property Not Persisting

**Cause:** Property not in `$fillable` array
**Solution:** Add property to `$fillable` in entity class

### 5. Lazy Load Not Working

**Cause:** Collection doesn't have `__deferred` marker
**Solution:** Use `$expand` in query or fetch separately

### 6. GuzzleHttp Version Conflicts

**Cause:** Library requires Guzzle ~6.0
**Solution:** For Guzzle 3.x, use v1.x of this library

## Multi-Country Support

Different Exact Online instances by country:

```php
$connection->setBaseUrl('https://start.exactonline.nl');  // Netherlands (default)
$connection->setBaseUrl('https://start.exactonline.de');  // Germany
$connection->setBaseUrl('https://start.exactonline.uk');  // United Kingdom
$connection->setBaseUrl('https://start.exactonline.be');  // Belgium
```

Reference: https://developers.exactonline.com/#Exact%20Online%20sites.html

## Advanced Features

### Custom Middleware

Add custom Guzzle middleware for logging, monitoring, etc.:

```php
$middleware = function (callable $handler) {
    return function ($request, array $options) use ($handler) {
        // Before request
        error_log('Request: ' . $request->getUri());

        return $handler($request, $options)->then(
            function ($response) {
                // After response
                error_log('Response: ' . $response->getStatusCode());
                return $response;
            }
        );
    };
};

$connection->insertMiddleWare($middleware);
```

### Token Update Callback

Persist tokens when refreshed:

```php
$connection->setTokenUpdateCallback(function($connection) {
    file_put_contents('tokens.json', json_encode([
        'access_token' => $connection->getAccessToken(),
        'refresh_token' => $connection->getRefreshToken(),
        'expires' => $connection->getTokenExpires(),
    ]));
});
```

### Webhooks

Subscribe to webhooks:

```php
$webhook = new WebhookSubscription($connection);
$webhook->CallbackURL = 'https://example.com/webhook';
$webhook->Topic = 'SalesInvoices';
$webhook->save();
```

Authenticate webhook calls:

```php
use Picqer\Financials\Exact\Webhook\Authenticatable;

class WebhookHandler {
    use Authenticatable;

    public function handle($request) {
        if ($this->authenticate($request->getContent(), $secret)) {
            // Process webhook
        }
    }
}
```

## AI Assistant Guidelines

When working with this codebase as an AI assistant:

### DO:

1. **Follow existing patterns:** Match the style of existing entity files exactly
2. **Use the userscript:** Reference userscript.js logic when manually creating entities
3. **Validate against Exact API docs:** Always check official Exact Online API documentation
4. **Add tests:** Every new entity needs a test method in EntityTest.php
5. **Update CHANGELOG:** Document all additions/changes in CHANGELOG.md
6. **Preserve compatibility:** This library supports PHP 7.4+, you can use PHP 7.4 features
7. **Use traits correctly:** Apply `Findable` and `Storable` traits consistently
8. **Document thoroughly:** PHPDoc is critical for IDE autocomplete

### DON'T:

1. **Don't modify core architecture:** Avoid changing Model, Connection, or trait behavior unless necessary
2. **Don't add dependencies:** Keep dependency footprint minimal
3. **Don't break BC:** Existing code relies on current API; maintain backward compatibility
4. **Don't ignore fillable:** All settable properties MUST be in `$fillable` array
5. **Don't guess property names:** Use exact names from Exact API (case-sensitive)
6. **Don't add business logic:** This is a thin API wrapper; keep entities simple
7. **Don't mix concerns:** Separate entities, connection logic, and query logic
8. **Don't skip docblocks:** Every entity needs complete PHPDoc with all properties

### Code Review Checklist:

When reviewing or creating code for this project:

- [ ] Entity extends `Model`
- [ ] Namespace is `Picqer\Financials\Exact`
- [ ] Uses `Query\Findable` and `Persistance\Storable` traits (unless special case)
- [ ] All properties in PHPDoc with correct types
- [ ] All properties in `$fillable` array
- [ ] `$url` is set correctly (matches Exact API endpoint)
- [ ] `$primaryKey` is set if not 'ID'
- [ ] Test method added to `EntityTest.php`
- [ ] CHANGELOG.md updated (if releasing)
- [ ] Code follows existing style (4 spaces, braces, visibility)
- [ ] PHP 7.4+ syntax is allowed (minimum version is 7.4.0)
- [ ] Avoid PHP 8.0+ exclusive features unless necessary (maintain 7.4 compatibility)

## Resources

- **Official API Docs:** https://developers.exactonline.com/
- **API Endpoints Reference:** https://start.exactonline.nl/docs/HlpRestAPIResources.aspx
- **OAuth2 Documentation:** https://developers.exactonline.com/#OAuth.html
- **Repository Issues:** https://github.com/picqer/exact-php-client/issues
- **Example Usage:** See `example/example.php`

## Version Information

**Current Version:** v4.5+ (PHP 7.4+, Guzzle 6/7)
**Previous Version:** v3.x (PHP 5.5+, Guzzle 6.x)
**Legacy Version:** v1.x (Guzzle 3.x)

**Major Changes in v4:**
- Upgraded minimum PHP version to 7.4.0
- Support for both Guzzle 6 and Guzzle 7
- Upgraded PHPUnit to 9.6+
- Added PHPStan static analysis (level 3)
- Migrated CI/CD from Travis CI to GitHub Actions
- Testing on PHP 7.4, 8.0, 8.1, 8.2, 8.3
- Modern PHPUnit configuration with XML schema
- Added autoload-dev for test namespace

**Major Changes in v3:**
- Upgraded to Guzzle 6.x
- Added webhook support
- Added lazy loading for collections
- Added token update callback
- Improved error handling

## Summary

This library is a **thin wrapper** around the Exact Online REST API. It handles OAuth2 authentication, provides a fluent interface for CRUD operations, and represents API resources as PHP objects.

**Core Principles:**
1. Simplicity - minimal abstraction over API
2. Consistency - all entities follow same pattern
3. Flexibility - traits provide optional capabilities
4. Maintainability - auto-generated entities from API docs

When in doubt, check existing entity implementations for patterns to follow.
