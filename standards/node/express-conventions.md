# Express.js Conventions

## Project Structure

### Standard Layout
```
src/
├── controllers/
│   ├── user-controller.ts
│   └── order-controller.ts
├── services/
│   ├── user-service.ts
│   └── order-service.ts
├── repositories/
│   ├── user-repository.ts
│   └── order-repository.ts
├── models/
│   ├── user.model.ts
│   └── order.model.ts
├── routes/
│   ├── user-routes.ts
│   └── order-routes.ts
├── middleware/
│   ├── auth.ts
│   ├── validation.ts
│   └── error-handler.ts
├── config/
│   ├── database.ts
│   └── env.ts
├── utils/
│   ├── logger.ts
│   └── errors.ts
├── types/
│   └── index.ts
└── app.ts
```

## Application Setup

### Basic Express App
```typescript
// app.ts
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import { userRoutes } from './routes/user-routes';
import { orderRoutes } from './routes/order-routes';
import { errorHandler } from './middleware/error-handler';
import { logger } from './utils/logger';

const app = express();

// Middleware
app.use(helmet()); // Security headers
app.use(cors()); // CORS
app.use(express.json()); // JSON body parser
app.use(express.urlencoded({ extended: true })); // URL-encoded body parser

// Request logging
app.use((req, res, next) => {
  logger.info(`${req.method} ${req.path}`);
  next();
});

// Routes
app.use('/api/v1/users', userRoutes);
app.use('/api/v1/orders', orderRoutes);

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Error handler (must be last)
app.use(errorHandler);

export default app;
```

### Server Startup
```typescript
// server.ts
import app from './app';
import { config } from './config/env';
import { logger } from './utils/logger';

const PORT = config.port || 3000;

app.listen(PORT, () => {
  logger.info(`Server running on port ${PORT}`);
});
```

## Routes

### Route Definition
```typescript
// routes/user-routes.ts
import { Router } from 'express';
import { userController } from '../controllers/user-controller';
import { authenticate } from '../middleware/auth';
import { validate } from '../middleware/validation';
import { createUserSchema, updateUserSchema } from '../validators/user.validator';

const router = Router();

// Public routes
router.get('/', userController.getUsers);
router.get('/:id', userController.getUserById);

// Protected routes
router.post(
  '/',
  authenticate,
  validate(createUserSchema),
  userController.createUser
);

router.put(
  '/:id',
  authenticate,
  validate(updateUserSchema),
  userController.updateUser
);

router.delete('/:id', authenticate, userController.deleteUser);

export { router as userRoutes };
```

### Route Organization
- Group related routes in single file
- Use descriptive route names
- Follow RESTful conventions
  ```typescript
  // ✅ Good
  GET    /api/v1/users          - List users
  GET    /api/v1/users/:id      - Get user by ID
  POST   /api/v1/users          - Create user
  PUT    /api/v1/users/:id      - Update user
  DELETE /api/v1/users/:id      - Delete user
  
  // ❌ Bad
  GET    /api/v1/getUsers
  POST   /api/v1/createUser
  GET    /api/v1/user/:id
  ```

## Controllers

### Controller Structure
```typescript
// controllers/user-controller.ts
import { Request, Response, NextFunction } from 'express';
import { userService } from '../services/user-service';
import { logger } from '../utils/logger';

export const userController = {
  async getUsers(req: Request, res: Response, next: NextFunction) {
    try {
      const { page = 1, pageSize = 20 } = req.query;
      const result = await userService.getUsers(
        Number(page),
        Number(pageSize)
      );
      
      res.json({
        success: true,
        data: result.items,
        meta: {
          page: result.page,
          pageSize: result.pageSize,
          total: result.total,
          totalPages: Math.ceil(result.total / result.pageSize)
        }
      });
    } catch (error) {
      logger.error('Error in getUsers', error);
      next(error);
    }
  },

  async getUserById(req: Request, res: Response, next: NextFunction) {
    try {
      const { id } = req.params;
      const user = await userService.getUserById(Number(id));
      
      res.json({
        success: true,
        data: user
      });
    } catch (error) {
      logger.error('Error in getUserById', error);
      next(error);
    }
  },

  async createUser(req: Request, res: Response, next: NextFunction) {
    try {
      const user = await userService.createUser(req.body);
      
      res.status(201).json({
        success: true,
        data: user
      });
    } catch (error) {
      logger.error('Error in createUser', error);
      next(error);
    }
  },

  async updateUser(req: Request, res: Response, next: NextFunction) {
    try {
      const { id } = req.params;
      const user = await userService.updateUser(Number(id), req.body);
      
      res.json({
        success: true,
        data: user
      });
    } catch (error) {
      logger.error('Error in updateUser', error);
      next(error);
    }
  },

  async deleteUser(req: Request, res: Response, next: NextFunction) {
    try {
      const { id } = req.params;
      await userService.deleteUser(Number(id));
      
      res.status(204).send();
    } catch (error) {
      logger.error('Error in deleteUser', error);
      next(error);
    }
  }
};
```

## Middleware

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
  logger.error('Error:', {
    message: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method
  });

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

  // Default error
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
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      res.status(401).json({
        success: false,
        error: {
          code: 'UNAUTHORIZED',
          message: 'Authentication required'
        }
      });
      return;
    }

    const token = authHeader.split(' ')[1];
    const decoded = verifyToken(token);
    
    const user = await userRepository.findById(decoded.userId);
    
    if (!user) {
      res.status(401).json({
        success: false,
        error: {
          code: 'UNAUTHORIZED',
          message: 'Invalid token'
        }
      });
      return;
    }

    // Attach user to request
    (req as any).user = user;
    next();
  } catch (error) {
    res.status(401).json({
      success: false,
      error: {
        code: 'UNAUTHORIZED',
        message: 'Invalid token'
      }
    });
  }
};
```

### Authorization Middleware
```typescript
// middleware/auth.ts
export const authorize = (...roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const user = (req as any).user;
    
    if (!user) {
      res.status(401).json({
        success: false,
        error: {
          code: 'UNAUTHORIZED',
          message: 'Authentication required'
        }
      });
      return;
    }

    if (!roles.includes(user.role)) {
      res.status(403).json({
        success: false,
        error: {
          code: 'FORBIDDEN',
          message: 'Insufficient permissions'
        }
      });
      return;
    }

    next();
  };
};

// Usage
router.delete('/:id', authenticate, authorize('admin'), userController.deleteUser);
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
      if (error instanceof z.ZodError) {
        res.status(400).json({
          success: false,
          error: {
            code: 'VALIDATION_ERROR',
            message: 'Validation failed',
            details: error.errors.map(err => ({
              field: err.path.join('.'),
              message: err.message
            }))
          }
        });
        return;
      }
      next(error);
    }
  };
};
```

### Request Logging Middleware
```typescript
// middleware/logger.ts
import { Request, Response, NextFunction } from 'express';
import { logger } from '../utils/logger';

export const requestLogger = (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    logger.info(`${req.method} ${req.path} ${res.statusCode} ${duration}ms`);
  });
  
  next();
};
```

## Validation

### Using Zod
```typescript
// validators/user.validator.ts
import { z } from 'zod';

export const createUserSchema = z.object({
  email: z.string().email('Invalid email format'),
  name: z.string().min(2, 'Name must be at least 2 characters').max(100),
  password: z.string().min(8, 'Password must be at least 8 characters')
});

export const updateUserSchema = z.object({
  email: z.string().email().optional(),
  name: z.string().min(2).max(100).optional(),
  password: z.string().min(8).optional()
});

export type CreateUserDto = z.infer<typeof createUserSchema>;
export type UpdateUserDto = z.infer<typeof updateUserSchema>;
```

### Using Joi
```typescript
// validators/user.validator.ts
import Joi from 'joi';

export const createUserSchema = Joi.object({
  email: Joi.string().email().required(),
  name: Joi.string().min(2).max(100).required(),
  password: Joi.string().min(8).required()
});

export const updateUserSchema = Joi.object({
  email: Joi.string().email(),
  name: Joi.string().min(2).max(100),
  password: Joi.string().min(8)
}).min(1); // At least one field required
```

## Configuration

### Environment Variables
```typescript
// config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  PORT: z.string().default('3000'),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  JWT_EXPIRES_IN: z.string().default('1h')
});

const parsed = envSchema.safeParse(process.env);

if (!parsed.success) {
  console.error('Invalid environment variables:', parsed.error.flatten().fieldErrors);
  process.exit(1);
}

export const config = {
  env: parsed.data.NODE_ENV,
  port: parseInt(parsed.data.PORT),
  databaseUrl: parsed.data.DATABASE_URL,
  jwt: {
    secret: parsed.data.JWT_SECRET,
    expiresIn: parsed.data.JWT_EXPIRES_IN
  }
};
```

## Best Practices

### Do
- Use TypeScript for type safety
- Organize code into layers (controllers, services, repositories)
- Use middleware for cross-cutting concerns (auth, validation, logging)
- Validate all input data with Zod or Joi
- Handle errors globally with error handler middleware
- Use environment variables for configuration
- Use async/await consistently
- Log important events and errors
- Use helmet for security headers
- Use CORS properly
- Rate limit your APIs
- Use HTTPS in production

### Don't
- Don't put business logic in controllers
- Don't use synchronous code (blocking)
- Don't expose stack traces in production
- Don't hardcode configuration values
- Don't use `console.log` (use a logger)
- Don't forget to handle errors
- Don't trust client input
- Don't store secrets in code
- Don't use `req.body` without validation
- Don't forget to sanitize input
- Don't use `app.listen()` without error handling
- Don't expose sensitive data in error responses

## Example: Complete User Endpoint

```typescript
// routes/user-routes.ts
import { Router } from 'express';
import { userController } from '../controllers/user-controller';
import { authenticate, authorize } from '../middleware/auth';
import { validate } from '../middleware/validation';
import { createUserSchema, updateUserSchema } from '../validators/user.validator';

const router = Router();

router.get('/', userController.getUsers);
router.get('/:id', userController.getUserById);
router.post('/', authenticate, validate(createUserSchema), userController.createUser);
router.put('/:id', authenticate, validate(updateUserSchema), userController.updateUser);
router.delete('/:id', authenticate, authorize('admin'), userController.deleteUser);

export { router as userRoutes };

// app.ts
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import { userRoutes } from './routes/user-routes';
import { errorHandler } from './middleware/error-handler';
import { requestLogger } from './middleware/logger';

const app = express();

app.use(helmet());
app.use(cors());
app.use(express.json());
app.use(requestLogger);

app.use('/api/v1/users', userRoutes);

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.use(errorHandler);

export default app;
```
