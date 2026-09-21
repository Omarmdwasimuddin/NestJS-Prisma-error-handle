## Prisma error handle (prisma v8)

#### `common/filters/prisma-exception.filter.ts`
```bash
import { ExceptionFilter, Catch, ArgumentsHost } from '@nestjs/common';

interface SqlQueryError {
  kind: string;
  sqlState?: string;
  constraint?: string;
  table?: string;
  column?: string;
  detail?: string;
  message: string;
}

function isSqlQueryError(error: unknown): error is SqlQueryError {
  return (
    typeof error === 'object' &&
    error !== null &&
    (error as any).kind === 'sql_query'
  );
}

@Catch()
export class PrismaExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();

    if (!isSqlQueryError(exception)) {
      throw exception; // Prisma/DB error না হলে অন্য handler-এ যেতে দাও
    }

    let status = 500;
    let message = 'Database error occurred';

    switch (exception.sqlState) {
      case '23505': // unique_violation
        status = 409;
        message = `Duplicate value for field: ${exception.constraint}`;
        break;
      case '23503': // foreign_key_violation
        status = 400;
        message = 'Invalid reference — related record does not exist';
        break;
      case '23502': // not_null_violation
        status = 400;
        message = `Missing required field: ${exception.column}`;
        break;
      default:
        status = 500;
    }

    response.status(status).json({
      success: false,
      error: { code: exception.sqlState, message, detail: exception.detail },
      timestamp: new Date().toISOString(),
    });
  }
}
```
---


#### main.ts e register:
```bash
import { PrismaExceptionFilter } from './common/filters/prisma-exception.filter.js';

app.useGlobalFilters(new PrismaExceptionFilter());
```
---

>## OUTPUT
><img width="1305" height="308" alt="image" src="https://github.com/user-attachments/assets/05311259-10f2-40ea-9ba6-81b0cade6f25" />
