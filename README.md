## Prisma error handle

#### `common/filters/prisma-exception.filter.ts`
```bash
import { ExceptionFilter, Catch, ArgumentsHost } from '@nestjs/common';
import { Prisma } from '@prisma/client';

@Catch(Prisma.PrismaClientKnownRequestError)
export class PrismaExceptionFilter implements ExceptionFilter {
  catch(exception: Prisma.PrismaClientKnownRequestError, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();

    let status = 500;
    let message = 'Database error occurred';

    switch (exception.code) {
      case 'P2002': // Unique constraint violation
        status = 409;
        message = `Duplicate value for field: ${exception.meta?.target}`;
        break;
      case 'P2025': // Record not found
        status = 404;
        message = 'Record not found';
        break;
      case 'P2003': // Foreign key constraint failed
        status = 400;
        message = 'Invalid reference — related record does not exist';
        break;
      default:
        status = 500;
        message = 'Unexpected database error';
    }

    response.status(status).json({
      success: false,
      error: { code: exception.code, message },
      timestamp: new Date().toISOString(),
    });
  }
}
```
---


#### main.ts e register:
```bash

```
---
