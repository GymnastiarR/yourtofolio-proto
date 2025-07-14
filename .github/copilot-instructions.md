# YourTofolio Proto - AI Coding Instructions

## Project Architecture

This is a **Protocol Buffers (protobuf) definition project** for the YourTofolio application that generates TypeScript types and service interfaces. The project uses a **layered service architecture** with clear boundaries between public and private APIs.

### Directory Structure & Service Boundaries
- `protoprot/public/` - Public-facing APIs (e.g., auth services accessible to clients)
- `protoprot/private/` - Internal microservice APIs (e.g., user management between services)  
- `protoprot/shared/` - Common data models and status types shared across all services
- `types/` - Generated TypeScript code (never edit manually)

### Naming Conventions
- **Proto packages**: Follow `{visibility}.{domain}.v{version}` pattern (e.g., `private.user.v1`)
- **Service names**: Use `{Domain}Service` format (e.g., `UserService`, `AuthService`)
- **Message fields**: Use snake_case in proto, converts to camelCase in TypeScript
- **Versioning**: Always include version (`v1`, `v2`) for backward compatibility

## Development Workflow

### Code Generation
```bash
bun run protoc:gen  # Regenerates all TypeScript types from .proto files
```
**Critical**: Always run this after any `.proto` file changes. Generated files use `@protobuf-ts/plugin` with `server_generic` parameter.

### Project Setup
```bash
bun install        # Install dependencies (uses Bun runtime)
```

### Key Dependencies
- `@protobuf-ts/plugin` - TypeScript code generation from protobuf
- Uses **Bun** as runtime (not Node.js) - ensure Bun-compatible code

## Protobuf Patterns

### Service Definition Pattern
```proto
service UserService {
  rpc CreateUser(CreateUserRequest) returns (CreateUserResponse) {};
  // Always include Status in responses
}
```

### Response Pattern
All service responses include `shared.common.v1.Status` for consistent error handling:
```proto
message CreateUserResponse {
    shared.common.v1.Status status = 1;
    // Optional data fields...
}
```

### Shared Types Usage
- Import shared types: `import "shared/user/v1/user.proto"`
- Use `UserCore` for user data across services
- Include timestamps: `google.protobuf.Timestamp created_at = 6`

## Generated Code Structure

### Service Interfaces
- `*.server.ts` - Server-side interfaces (`IUserService<T = ServerCallContext>`)
- `*.client.ts` - Client-side classes (`UserServiceClient`)
- `*.ts` - Message type definitions and metadata

### Import Patterns
Generated code follows specific import structure:
```typescript
import { UserCore } from "../../../shared/user/v1/user";
import { Status } from "../../../shared/common/v1/status";
```

## CI/CD Integration

- **Trigger**: Changes to `protos/**` (note: path in CI differs from actual `protoprot/`)
- **Process**: Auto-generates types and publishes to GitHub Packages
- **Registry**: Uses GitHub npm registry (`npm.pkg.github.com`)

## Critical Notes

- **Never manually edit** files in `types/` directory - always regenerate
- **Security**: User passwords marked as "should be hashed in practice"
- **Auth tokens**: Uses JWT access/refresh token pattern in `AuthService`
- **Versioning**: Always increment proto versions for breaking changes
- **Path aliases**: `@protoprot/*` available for proto imports (tsconfig)

## Common Tasks

1. **Adding new service**: Create in appropriate `public/` or `private/` directory
2. **Modifying messages**: Update proto → run `bun run protoc:gen` → test generated types
3. **Cross-service communication**: Use shared types from `protoprot/shared/`
4. **API evolution**: Create new version directory (`v2/`) for breaking changes
