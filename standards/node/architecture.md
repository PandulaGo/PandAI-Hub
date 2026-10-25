# Node.js Architecture

## Layering

### Standard Layered Architecture
```
Controllers → Services → Repositories → Database
```

- **Controllers**: Handle HTTP requests/responses, validation, routing
- **Services**: Business logic, orchestration, data transformation
- **Repositories**: Data access, database queries, external API calls
- **Models**: Data structures, interfaces, types

### Dependency Direction
- Controllers depend on Services
- Services depend on Repositories
- Repositories depend on Models/Database
- Never reverse dependencies

## Project Structure

### Standard Structure
```
project-root/
├── src/
│   ├── controllers/
│   │   ├── user-controller.ts
│   │   └── order-controller.ts
│   ├── services/
│   │   ├── user-service.ts
│   │   └── order-service.ts
│   ├── repositories/
│   │   ├── user-repository.ts
│   │   └── order-repository.ts
│   ├── models/
│   │   ├── user.model.ts
│   │   └── order.model.ts
│   ├── routes/
│   │   ├── user-routes.ts
│   │   └── order-routes.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── validation.ts
│   │   └── error-handler.ts
│   ├── config/
│   │   ├── database.ts
│   │   └── env.ts
│   ├── utils/
│   │   ├── logger.ts
│   │   └── helpers.ts
│   ├── types/
│   │   └── index.ts
│   └── app.ts
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── scripts/
│   ├── seed.ts
│   └── migrate.ts
└── main.ts
```

### Feature-Based Structure (Alternative)
```
src/
├── features/
│   ├── users/
│   │   ├── user-controller.ts
│   │   ├── user-service.ts
│   │   ├── user-repository.ts
│   │   ├── user.model.ts
│   │   └── user-routes.ts
│   └── orders/
│       ├── order-controller.ts
│       ├── order-service.ts
│       ├── order-repository.ts
│       ├── order.model.ts
│       └── order-routes.ts
├── shared/
│   ├── middleware/
│   ├── utils/
│   └── types/
└── app.ts
```

## Controller Pattern

### Express Controller
```typescript
// controllers/user-controller.ts
import { Request, Response, NextFunction } from 'express';
import { userService } from '../services/user-service';
import { logger } from '../utils/logger';

export const getUsers = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { page = 1, pageSize = 20 } = req.query;
    const users = await userService.getUsers(Number(page), Number(pageSize));
    
    res.json({
      success: true,
      data: users,
      meta: {
        page: Number(page),
        pageSize: Number(pageSize),
      }
    });
  } catch (error) {
    logger.error('Error fetching users', error);
    next(error);
  }
};

export const getUserById = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { id } = req.params;
    const user = await userService.getUserById(Number(id));
    
    if (!user) {
      res.status(404).json({
        success: false,
        error: { code: 'NOT_FOUND', message: 'User not found' }
      });
      return;
    }
    
    res.json({
      success: true,
      data: user
    });
  } catch (error) {
    logger.error('Error fetching user', error);
    next(error);
  }
};

export const createUser = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const user = await userService.createUser(req.body);
    
    res.status(201).json({
      success: true,
      data: user
    });
  } catch (error) {
    logger.error('Error creating user', error);
    next(error);
  }
};
```

## Service Pattern

### Business Logic Layer
```typescript
// services/user-service.ts
import { userRepository } from '../repositories/user-repository';
import { User } from '../models/user.model';
import { CreateUserDto, UpdateUserDto } from '../types';
import { AppError } from '../utils/errors';

export const userService = {
  async getUsers(page: number, pageSize: number) {
    return await userRepository.findAll(page, pageSize);
  },

  async getUserById(id: number) {
    const user = await userRepository.findById(id);
    if (!user) {
      throw new AppError('User not found', 404, 'NOT_FOUND');
    }
    return user;
  },

  async getUserByEmail(email: string) {
    return await userRepository.findByEmail(email);
  },

  async createUser(dto: CreateUserDto) {
    // Check if user already exists
    const existingUser = await userRepository.findByEmail(dto.email);
    if (existingUser) {
      throw new AppError('Email already in use', 409, 'CONFLICT');
    }

    // Hash password
    const hashedPassword = await hashPassword(dto.password);

    // Create user
    const user = await userRepository.create({
      ...dto,
      password: hashedPassword
    });

    return user;
  },

  async updateUser(id: number, dto: UpdateUserDto) {
    const user = await userRepository.findById(id);
    if (!user) {
      throw new AppError('User not found', 404, 'NOT_FOUND');
    }

    return await userRepository.update(id, dto);
  },

  async deleteUser(id: number) {
    const user = await userRepository.findById(id);
    if (!user) {
      throw new AppError('User not found', 404, 'NOT_FOUND');
    }

    await userRepository.delete(id);
  }
};
```

## Repository Pattern

### Data Access Layer
```typescript
// repositories/user-repository.ts
import { db } from '../config/database';
import { User } from '../models/user.model';
import { CreateUserDto, UpdateUserDto } from '../types';

export const userRepository = {
  async findAll(page: number, pageSize: number) {
    const offset = (page - 1) * pageSize;
    
    const [users, total] = await Promise.all([
      db.query('SELECT * FROM users LIMIT $1 OFFSET $2', [pageSize, offset]),
      db.query('SELECT COUNT(*) FROM users')
    ]);

    return {
      items: users.rows,
      total: parseInt(total.rows[0].count),
      page,
      pageSize
    };
  },

  async findById(id: number) {
    const result = await db.query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0] || null;
  },

  async findByEmail(email: string) {
    const result = await db.query('SELECT * FROM users WHERE email = $1', [email]);
    return result.rows[0] || null;
  },

  async create(dto: CreateUserDto) {
    const result = await db.query(
      `INSERT INTO users (email, name, password, created_at)
       VALUES ($1, $2, $3, NOW())
       RETURNING *`,
      [dto.email, dto.name, dto.password]
    );
    return result.rows[0];
  },

  async update(id: number, dto: UpdateUserDto) {
    const fields: string[] = [];
    const values: any[] = [];
    let paramCount = 1;

    if (dto.name) {
      fields.push(`name = $${paramCount++}`);
      values.push(dto.name);
    }
    if (dto.email) {
      fields.push(`email = $${paramCount++}`);
      values.push(dto.email);
    }

    if (fields.length === 0) {
      return await this.findById(id);
    }

    values.push(id);
    const result = await db.query(
      `UPDATE users SET ${fields.join(', ')} WHERE id = $${paramCount} RETURNING *`,
      values
    );
    return result.rows[0];
  },

  async delete(id: number) {
    await db.query('DELETE FROM users WHERE id = $1', [id]);
  }
};
```

## Middleware Pattern

### Error Handler Middleware
```typescript
// middleware/error-handler.ts
import { Request, Response, NextFunction } from 'express';
import { AppError } from '../utils/errors';
import { logger } from '../utils/logger';

export const errorHandler = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  logger.error('Error:', err);

  if (err instanceof AppError) {
    res.status(err.statusCode).json({
      success: false,
      error: {
        code: err.code,
        message: err.message,
        details: err.details
      }
    });
    return;
  }

  res.status(500).json({
    success: false,
    error: {
      code: 'INTERNAL_ERROR',
      message: 'An unexpected error occurred'
    }
  });
};
```

### Authentication Middleware
```typescript
// middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import { verifyToken } from '../utils/jwt';
import { userRepository } from '../repositories/user-repository';

export const authenticate = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  try {
    const token = req.headers.authorization?.split(' ')[1];
    
    if (!token) {
      res.status(401).json({
        success: false,
        error: { code: 'UNAUTHORIZED', message: 'Authentication required' }
      });
      return;
    }

    const decoded = verifyToken(token);
    const user = await userRepository.findById(decoded.userId);

    if (!user) {
      res.status(401).json({
        success: false,
        error: { code: 'UNAUTHORIZED', message: 'Invalid token' }
      });
      return;
    }

    req.user = user;
    next();
  } catch (error) {
    res.status(401).json({
      success: false,
      error: { code: 'UNAUTHORIZED', message: 'Invalid token' }
    });
  }
};
```

### Validation Middleware
```typescript
// middleware/validation.ts
import { Request, Response, NextFunction } from 'express';
import { ZodSchema } from 'zod';

export const validate = (schema: ZodSchema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      schema.parse(req.body);
      next();
    } catch (error) {
      res.status(400).json({
        success: false,
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Validation failed',
          details: error.errors
        }
      });
    }
  };
};
```

## Error Handling

### Custom Error Class
```typescript
// utils/errors.ts
export class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code: string = 'INTERNAL_ERROR',
    public details?: any
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

export class ValidationError extends AppError {
  constructor(details: any) {
    super('Validation failed', 400, 'VALIDATION_ERROR', details);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401, 'UNAUTHORIZED');
  }
}

export class ConflictError extends AppError {
  constructor(message: string) {
    super(message, 409, 'CONFLICT');
  }
}
```

## Best Practices

### Do
- Use TypeScript for type safety
- Separate concerns (controllers, services, repositories)
- Use dependency injection
- Validate all input data
- Handle errors globally
- Use async/await consistently
- Log important events and errors
- Use environment variables for configuration
- Write unit and integration tests
- Use a linter and formatter (ESLint, Prettier)

### Don't
- Don't put business logic in controllers
- Don't use synchronous database calls
- Don't expose sensitive data in error responses
- Don't hardcode configuration values
- Don't ignore errors (always handle or log them)
- Don't use `any` type (use proper TypeScript types)
- Don't mix callbacks and promises
- Don't forget to validate and sanitize input
- Don't use `console.log` in production (use a logger)
- Don't commit `.env` files or secrets
