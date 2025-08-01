# Vercel Function Limit Optimization

## Problem

Vercel's Hobby plan has a limit of 12 Serverless Functions per deployment. Our original implementation created a separate function for each API endpoint, which exceeded this limit.

## Solution

We've consolidated multiple API endpoints into fewer serverless functions to stay within the limit. This approach is more efficient while maintaining the same functionality:

### 1. Consolidated API Handlers

We've created two main handler files:

- **consolidated-api.js**: Handles multiple API endpoints:
  - `/api/auth/login`
  - `/api/auth/logout`
  - `/api/users/search`
  - `/api/chats`

- **consolidated-auth.js**: Handles authentication endpoints:
  - `/api/auth/register`
  - `/api/auth/test`

### 2. Routing Logic

Each consolidated file contains a main handler function that:
1. Extracts the requested path from the URL
2. Routes the request to the appropriate sub-handler based on the path
3. Shares common functionality like database connection and error handling

### 3. Updated vercel.json

We've updated the vercel.json file in two important ways:

1. **Route configuration**: Routes multiple API paths to the same handler files
    ```json
    {
      "src": "/api/auth/login",
      "dest": "/api/consolidated-api.js"
    },
    {
      "src": "/api/auth/logout",
      "dest": "/api/consolidated-api.js"
    }
    ```

2. **Build configuration**: Specifically lists only the necessary files instead of using wildcards
    ```json
    "builds": [
      { "src": "server.js", "use": "@vercel/node" },
      { "src": "api/consolidated-*.js", "use": "@vercel/node" },
      { "src": "api/allowCors.js", "use": "@vercel/node" },
      { "src": "api/auth-middleware-new.js", "use": "@vercel/node" },
      { "src": "api/cors-test.js", "use": "@vercel/node" }
    ]
    ```

## Important: Before Each Deployment

To ensure your deployment stays within the function limits, run one of these scripts before pushing to GitHub:

- **Windows**: `.\cleanup-vercel-functions.ps1`
- **Linux/Mac**: `./cleanup-vercel-functions.sh`

These scripts will:
1. Move unused API files to an `_archived` folder
2. Update your vercel.json if needed
3. Ensure only the necessary files are included in the build

## Benefits

1. **Reduced Function Count**: Stays within Vercel's Hobby plan limit of 12 functions
2. **Shared Resources**: Common code like database connections is shared between endpoints
3. **Simplified Deployment**: Fewer files to manage and deploy
4. **Consistent Error Handling**: Standardized error responses across all endpoints

## Limitations

1. **Cold Start Penalties**: A single cold start affects multiple endpoints in the same file
2. **Error Isolation**: An error in one handler might affect other handlers in the same file
3. **Debugging Complexity**: Logs from multiple endpoints are combined

## Monitoring and Scaling

If your application grows beyond the Hobby plan limits, consider:

1. Upgrading to Vercel's Pro plan which allows more functions
2. Splitting high-traffic endpoints into separate functions
3. Using edge functions for simpler, high-volume requests

## Testing

Use the included test scripts to verify all endpoints still work as expected after consolidation:
- `test-api.sh` (Linux/Mac)
- `test-api.ps1` (Windows)
