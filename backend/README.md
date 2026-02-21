# PayD Backend - Advanced Search & Filtering

This backend implements a powerful search and filtering engine for employee lists and transaction history using PostgreSQL full-text search.

## Features

- Full-text search using PostgreSQL `tsvector` and `ts_rank`
- Multi-criteria filtering (status, date ranges, amount ranges)
- Pagination support
- Performant with proper indexing
- Type-safe with TypeScript and Zod validation

## Setup

1. Install dependencies:

```bash
npm install
```

2. Create a `.env` file:

```bash
cp .env.example .env
```

3. Configure your database connection in `.env`:

```
DATABASE_URL=postgresql://user:password@localhost:5432/payd
PORT=3000
NODE_ENV=development
```

4. Run the database migration:

```bash
psql -d payd -f src/db/migrations/001_create_tables.sql
```

5. Start the development server:

```bash
npm run dev
```

## API Endpoints

### Search Employees

```
GET /api/search/organizations/:organizationId/employees
```

Query Parameters:

- `query` - Full-text search on name, email, position, department
- `status` - Filter by status (comma-separated: active,inactive,pending)
- `dateFrom` - Filter by creation date (ISO 8601)
- `dateTo` - Filter by creation date (ISO 8601)
- `page` - Page number (default: 1)
- `limit` - Items per page (default: 20, max: 100)
- `sortBy` - Sort column (created_at, first_name, last_name, email, status)
- `sortOrder` - Sort order (asc, desc)

Example:

```bash
curl "http://localhost:3000/api/search/organizations/1/employees?query=john&status=active&page=1&limit=20"
```

Response:

```json
{
  "data": [
    {
      "id": 1,
      "organization_id": 1,
      "first_name": "John",
      "last_name": "Doe",
      "email": "john.doe@example.com",
      "wallet_address": "GXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
      "status": "active",
      "position": "Software Engineer",
      "department": "Engineering",
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "totalPages": 3
  }
}
```

### Search Transactions

```
GET /api/search/organizations/:organizationId/transactions
```

Query Parameters:

- `query` - Full-text search on tx_hash, asset_code
- `status` - Filter by status (comma-separated: pending,completed,failed)
- `dateFrom` - Filter by creation date (ISO 8601)
- `dateTo` - Filter by creation date (ISO 8601)
- `amountMin` - Minimum transaction amount
- `amountMax` - Maximum transaction amount
- `page` - Page number (default: 1)
- `limit` - Items per page (default: 20, max: 100)
- `sortBy` - Sort column (created_at, amount, status, tx_hash)
- `sortOrder` - Sort order (asc, desc)

Example:

```bash
curl "http://localhost:3000/api/search/organizations/1/transactions?query=abc123&status=completed&amountMin=100&amountMax=1000&page=1"
```

Response:

```json
{
  "data": [
    {
      "id": 1,
      "organization_id": 1,
      "employee_id": 5,
      "tx_hash": "abc123def456...",
      "amount": "500.0000000",
      "asset_code": "USDC",
      "status": "completed",
      "transaction_type": "payment",
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z",
      "employee_first_name": "Jane",
      "employee_last_name": "Smith"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

## Database Schema

The implementation uses PostgreSQL's full-text search capabilities:

- `tsvector` columns are automatically generated and stored
- GIN indexes on `search_vector` for fast full-text queries
- B-tree indexes on frequently filtered columns (status, dates, amounts)
- `ts_rank()` function for relevance scoring

### Full-Text Search Weights

Employees:

- A (highest): first_name, last_name
- B: email
- C: position, department

Transactions:

- A (highest): tx_hash
- B: asset_code
- C: status

## Performance Considerations

- Pagination limits prevent large result sets
- Indexes on all filterable columns
- Generated `tsvector` columns for instant search
- Connection pooling for database efficiency
- Query parameter validation to prevent SQL injection

## Development

Build for production:

```bash
npm run build
npm start
```

Run tests:

```bash
npm test
```

Run tests with coverage:

```bash
npm test -- --coverage
```

Run tests in watch mode:

```bash
npm run test:watch
```

Lint code:

```bash
npm run lint
```

Format code:

```bash
npm run format
```

## Testing

This project includes comprehensive unit and integration tests covering:

- Full-text search functionality
- Multi-criteria filtering
- Pagination logic
- Error handling
- Edge cases

See [TESTING.md](TESTING.md) for detailed testing documentation.

### Test Coverage

- 60+ test cases
- Unit tests for search service logic
- Integration tests for API endpoints
- Mocked database for fast, reliable tests

Run tests:

```bash
npm test
```

View coverage report:

```bash
npm test -- --coverage
open coverage/lcov-report/index.html
```
