# Backend Code Style Guide (Express + SQLite)

## Table of Contents
1. [Project Structure](#project-structure)
2. [Code Organization](#code-organization)
3. [API Design](#api-design)
4. [Database Operations](#database-operations)
5. [Error Handling](#error-handling)
6. [Logging](#logging)
7. [Security](#security)
8. [Code Conventions](#code-conventions)
9. [Best Practices](#best-practices)

## Project Structure

```
backend/
├── server.js              # Main Express application
├── database.db            # SQLite database file
├── logger.js              # Logging utility
├── docs.js                # Swagger documentation
├── package.json
└── README.md
```

## Code Organization

### Module Structure
Organize code by feature and concern separation.

```javascript
// ✓ Good: Clear separation of concerns
const express = require('express')
const cors = require('cors')
const Database = require('better-sqlite3')
const logger = require('./logger')

const app = express()
const PORT = process.env.PORT || 3000

// Middleware setup
app.use(cors())
app.use(express.json())

// Route definitions
app.get('/api/health', (req, res) => { })
```

### Constants and Configuration
Centralize constants at the top of the file or in separate config files.

```javascript
// ✓ Good: Organized constants
const PORT = 3000
const API_BASE_URL = '/api'

// Error codes enum
const ERROR_CODES = {
  SUCCESS: 0,
  VALIDATION_ERROR: 1,
  NOT_FOUND: 2,
  DATABASE_ERROR: 3,
  NETWORK_ERROR: -1,
}

// Error messages configuration
const ERROR_MESSAGES = {
  REQUIRED_FIELD_MISSING: 'Required field is missing',
  CONTACT_NOT_FOUND: 'Contact not found',
  DATABASE_OPERATION_FAILED: 'Database operation failed',
}
```

## API Design

### RESTful Principles
Follow REST principles for API design.

```javascript
// ✓ Good: RESTful routes
GET    /api/contacts       # List all contacts
GET    /api/contacts/:id   # Get single contact
POST   /api/contacts       # Create new contact
PUT    /api/contacts/:id   # Update contact
DELETE /api/contacts/:id   # Delete contact
```

### Response Format
Use a consistent response format for all API endpoints.

```javascript
// Standard response format
{
  code: 0,        // 0 = success, non-zero = error
  msg: "Success", // Human-readable message
  data: {...}     // Optional payload
}
```

### Route Handler Pattern
Structure route handlers consistently.

```javascript
// ✓ Good: Consistent handler structure
app.get('/api/contacts', (req, res) => {
  try {
    // 1. Extract parameters
    const { id } = req.params

    // 2. Validate input
    if (!id) {
      return sendResponse(res, ERROR_CODES.VALIDATION_ERROR, ERROR_MESSAGES.REQUIRED_FIELD_MISSING)
    }

    // 3. Perform operation
    const result = db.prepare('SELECT * FROM contacts WHERE id = ?').get(id)

    // 4. Send response
    if (result) {
      sendResponse(res, ERROR_CODES.SUCCESS, ERROR_MESSAGES.SUCCESS, result)
    } else {
      sendResponse(res, ERROR_CODES.NOT_FOUND, ERROR_MESSAGES.CONTACT_NOT_FOUND)
    }
  } catch (error) {
    // 5. Handle errors
    logger.error('Error description', { error: error.message })
    sendResponse(res, ERROR_CODES.DATABASE_ERROR, ERROR_MESSAGES.DATABASE_OPERATION_FAILED)
  }
})
```

## Database Operations

### Prepared Statements
Always use prepared statements to prevent SQL injection.

```javascript
// ✓ Good: Prepared statement
const stmt = db.prepare('SELECT * FROM contacts WHERE id = ?')
const result = stmt.get(contactId)

// ✓ Good: Multiple parameters
const stmt = db.prepare('INSERT INTO contacts (name, phone, email) VALUES (?, ?, ?)')
const result = stmt.run(name, phone, email)
```

### Transaction Handling
Use transactions for multiple related operations.

```javascript
// ✓ Good: Transaction example
const transaction = db.transaction((contacts) => {
  const stmt = db.prepare('INSERT INTO contacts (name, phone) VALUES (?, ?)')
  contacts.forEach(contact => {
    stmt.run(contact.name, contact.phone)
  })
})

try {
  transaction(contactList)
  sendResponse(res, ERROR_CODES.SUCCESS, 'Contacts imported successfully')
} catch (error) {
  logger.error('Import failed', { error: error.message })
  sendResponse(res, ERROR_CODES.DATABASE_ERROR, 'Import failed')
}
```

### Query Helpers
Create reusable query helpers.

```javascript
// ✓ Good: Helper functions
const getContactById = (id) => {
  return db.prepare('SELECT * FROM contacts WHERE id = ?').get(id)
}

const updateContact = (id, data) => {
  const stmt = db.prepare('UPDATE contacts SET name = ?, phone = ?, email = ?, address = ? WHERE id = ?')
  return stmt.run(data.name, data.phone, data.email, data.address, id)
}
```

## Error Handling

### Centralized Error Handler
Use a centralized error handling pattern.

```javascript
// ✓ Good: Helper function for responses
const sendResponse = (res, code, msg, data = null) => {
  const response = { code, msg }
  if (data !== null) {
    response.data = data
  }
  res.json(response)
}

// Usage in route handlers
app.get('/api/contacts', (req, res) => {
  try {
    const contacts = db.prepare('SELECT * FROM contacts').all()
    sendResponse(res, ERROR_CODES.SUCCESS, 'Success', contacts)
  } catch (error) {
    logger.error('Database error', { error: error.message })
    sendResponse(res, ERROR_CODES.DATABASE_ERROR, ERROR_MESSAGES.DATABASE_OPERATION_FAILED)
  }
})
```

### Input Validation
Validate all inputs at the endpoint level.

```javascript
// ✓ Good: Input validation
const validateContactInput = (name, phone) => {
  if (!name || !phone) {
    return ERROR_MESSAGES.REQUIRED_FIELD_MISSING
  }
  return null
}

app.post('/api/contacts', (req, res) => {
  const { name, phone, email, address } = req.body

  const validationError = validateContactInput(name, phone)
  if (validationError) {
    return sendResponse(res, ERROR_CODES.VALIDATION_ERROR, validationError)
  }

  // Process valid request
})
```

### Error Logging
Log errors with sufficient context for debugging.

```javascript
// ✓ Good: Detailed error logging
catch (error) {
  logger.error('Error creating contact', {
    error: error.message,
    stack: error.stack,
    contactData: { name, phone, email, address }
  })
  sendResponse(res, ERROR_CODES.DATABASE_ERROR, ERROR_MESSAGES.DATABASE_OPERATION_FAILED)
}
```

## Logging

### Log Levels
Use appropriate log levels for different types of messages.

```javascript
logger.info('User action', { userId: 123, action: 'login' })
logger.warn('Deprecated endpoint', { endpoint: '/old/api' })
logger.error('Database connection failed', { error: error.message })
logger.debug('Query executed', { query: 'SELECT * FROM contacts', params: [] })
```

### Structured Logging
Use structured logging with context objects.

```javascript
// ✓ Good: Structured logging
logger.info('Contact retrieved successfully', {
  id: contactId,
  operation: 'read',
  timestamp: new Date().toISOString()
})

logger.error('Contact update failed', {
  id: contactId,
  attemptedData: updateData,
  error: error.message,
  stack: error.stack
})
```

## Security

### Input Sanitization
Sanitize and validate all inputs.

```javascript
// ✓ Good: Basic sanitization
const sanitizeInput = (str) => {
  if (typeof str !== 'string') return str
  return str.trim().replace(/[<>]/g, '')
}

const name = sanitizeInput(req.body.name)
```

### CORS Configuration
Configure CORS appropriately for your environment.

```javascript
// ✓ Good: Specific CORS configuration
const corsOptions = {
  origin: ['http://localhost:5173', 'https://yourdomain.com'],
  optionsSuccessStatus: 200
}

app.use(cors(corsOptions))
```

### Rate Limiting
Consider implementing rate limiting for production.

```javascript
// ✓ Good: Rate limiting (if needed)
const rateLimit = require('express-rate-limit')

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
})

app.use('/api/', limiter)
```

## Code Conventions

### Naming Conventions
- **Variables and functions**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Database tables**: lowercase with underscores
- **Database columns**: lowercase with underscores

```javascript
// ✓ Good
const databaseConnection = null
const MAX_RETRY_COUNT = 3
const getUserById = (id) => { }

// ✓ Good: Database
-- users table
-- user_profiles table
-- contact_id (foreign key)
```

### Comments and Documentation
Use JSDoc for function documentation and inline comments for complex logic.

```javascript
/**
 * Process contact creation
 * @param {Object} contactData - The contact information
 * @param {string} contactData.name - Contact name (required)
 * @param {string} contactData.phone - Phone number (required)
 * @param {string} contactData.email - Email address (optional)
 * @returns {Object} Created contact object
 */
const createContact = (contactData) => {
  // Implementation
}

// ✗ Avoid: Unclear comments
// Process data
processContact(data)
```

### Module Pattern
Use clear module patterns and exports.

```javascript
// ✓ Good: Named exports for utilities
module.exports = {
  ERROR_CODES,
  ERROR_MESSAGES,
  validateContactInput,
  sendResponse
}

// ✓ Good: Export app for testing
const app = express()
module.exports = app

// Then in a separate file
app.listen(PORT, () => {
  logger.info(`Server running on port ${PORT}`)
})
```

## Best Practices

### 1. Environment Configuration
Use environment variables for configuration.

```javascript
// ✓ Good
const PORT = process.env.PORT || 3000
const NODE_ENV = process.env.NODE_ENV || 'development'

// ✓ Good: Environment-specific logic
if (NODE_ENV === 'production') {
  // Production setup
} else {
  // Development setup
}
```

### 2. Async/Await for Database Operations
Use async/await for asynchronous operations.

```javascript
// ✓ Good: Modern async syntax
app.get('/api/contacts', async (req, res) => {
  try {
    const contacts = await db.prepare('SELECT * FROM contacts').all()
    sendResponse(res, ERROR_CODES.SUCCESS, 'Success', contacts)
  } catch (error) {
    logger.error('Database error', { error: error.message })
    sendResponse(res, ERROR_CODES.DATABASE_ERROR, ERROR_MESSAGES.DATABASE_OPERATION_FAILED)
  }
})
```

### 3. Database Connection Management
Initialize database connections once and reuse them.

```javascript
// ✓ Good: Single database instance
const db = new Database('database.db')

// Configure database
db.pragma('journal_mode = WAL')
db.pragma('foreign_keys = ON')
```

### 4. Resource Cleanup
Ensure proper cleanup of database connections.

```javascript
// ✓ Good: Graceful shutdown
process.on('SIGINT', () => {
  logger.info('Shutting down gracefully...')
  db.close()
  process.exit(0)
})
```

### 5. HTTP Status Codes
Use appropriate HTTP status codes.

```javascript
// ✓ Good: Proper status codes
res.status(200).json(response)     // Success
res.status(201).json(response)     // Created
res.status(400).json(response)     // Bad Request
res.status(404).json(response)     // Not Found
res.status(500).json(response)     // Server Error
```

### 6. API Versioning
Plan for API versioning from the start.

```javascript
// ✓ Good: Versioned routes
app.get('/api/v1/contacts', handler)
app.get('/api/v2/contacts', newHandler)
```

### 7. Swagger Documentation
Document APIs using Swagger/OpenAPI.

```javascript
/**
 * @swagger
 * /api/contacts:
 *   get:
 *     summary: Get all contacts
 *     responses:
 *       200:
 *         description: List of contacts
 */
app.get('/api/contacts', (req, res) => {
  // Implementation
})
```

### 8. Middleware Ordering
Place middleware in logical order.

```javascript
// ✓ Good: Ordered middleware
app.use(helmet())              // Security headers first
app.use(cors())                // CORS
app.use(express.json())        // Body parsing
app.use(rateLimiter)           // Rate limiting
app.use('/api', routes)        // Routes last
```

## Code Review Checklist

- [ ] Follows RESTful API design principles
- [ ] Uses prepared statements for all database queries
- [ ] Has proper error handling with try-catch blocks
- [ ] Uses structured logging with context
- [ ] Validates all inputs
- [ ] Uses appropriate HTTP status codes
- [ ] Follows naming conventions
- [ ] Has JSDoc comments for complex functions
- [ ] No hardcoded secrets or configuration
- [ ] Environment variables used for config
- [ ] CORS properly configured
- [ ] Response format is consistent
- [ ] Database transactions used for multi-step operations
- [ ] Resources are properly cleaned up

## Common Patterns

### CRUD Operations Pattern
```javascript
// Create
app.post('/api/resource', (req, res) => {
  // 1. Validate
  // 2. Insert
  // 3. Return created resource
})

// Read All
app.get('/api/resource', (req, res) => {
  // 1. Query
  // 2. Return array
})

// Read One
app.get('/api/resource/:id', (req, res) => {
  // 1. Validate ID
  // 2. Query
  // 3. Return resource or 404
})

// Update
app.put('/api/resource/:id', (req, res) => {
  // 1. Validate ID and body
  // 2. Update
  // 3. Return updated resource
})

// Delete
app.delete('/api/resource/:id', (req, res) => {
  // 1. Validate ID
  // 2. Delete
  // 3. Return success message
})
```

## Resources

- [Express.js Best Practices](https://expressjs.com/en/advanced/best-practice-security.html)
- [REST API Design](https://restfulapi.net/)
- [better-sqlite3 Documentation](https://github.com/JoshuaWise/better-sqlite3)
- [OpenAPI Specification](https://swagger.io/specification/)
- [Node.js Error Handling](https://nodejs.org/api/errors.html)
