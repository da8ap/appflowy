# Troubleshooting Guide

## Build Errors

### Error: "cp -r /tmp/appflowy/backend /app/backend" failed

This error indicates that the AppFlowy repository structure doesn't match what we expected. The updated Dockerfiles now include debugging that will show the actual repository structure.

**What to do:**

1. **Check the build logs** - The updated Dockerfiles will output the repository structure before failing. Look for lines like:
   ```
   === AppFlowy repository structure ===
   === Top-level directories ===
   === Looking for backend/server/Cargo.toml ===
   ```

2. **Verify AppFlowy repository structure** - The AppFlowy repository structure may have changed. Check the [official AppFlowy repository](https://github.com/AppFlowy-IO/appflowy) to see the current structure.

3. **Alternative: Use AppFlowy Collaboration Server**
   - AppFlowy might use a separate collaboration server repository
   - Check: https://github.com/AppFlowy-IO/appflowy-collaboration
   - Or: https://github.com/AppFlowy-IO/appflowy-server

4. **Use a specific release tag** - Instead of `main` branch, try using a specific release tag:
   ```dockerfile
   ARG APPFLOWY_VERSION=v0.3.0  # or latest stable release
   ```

## Common Issues

### Issue: Backend directory not found

**Solution:** The Dockerfile now tries multiple locations:
- `/tmp/appflowy/backend`
- `/tmp/appflowy/appflowy-backend`
- `/tmp/appflowy/server`
- Root directory if Cargo.toml is there

If none work, check the build logs to see the actual structure.

### Issue: Cargo.toml not found

**Solution:** The build will show where Cargo.toml files are located. You may need to adjust the Dockerfile to use a different path.

### Issue: Frontend build fails

**Solution:** Similar to backend - check the logs for the actual frontend structure. AppFlowy frontend might be in:
- `/tmp/appflowy/frontend`
- `/tmp/appflowy/appflowy` (the Flutter app itself)

## Alternative Approaches

### Option 1: Use Pre-built Images (if available)

If AppFlowy provides official Docker images, update `docker-compose.yml`:

```yaml
backend:
  image: appflowyio/appflowy-backend:latest  # if available
  # Remove build section
```

### Option 2: Use AppFlowy Collaboration Server

AppFlowy might have a separate server component. Check:
- https://github.com/AppFlowy-IO/appflowy-collaboration
- https://github.com/AppFlowy-IO/appflowy-server

### Option 3: Build from Local Source

If you have AppFlowy source locally:

1. Copy the source into your repository
2. Update Dockerfiles to use `COPY` instead of `git clone`
3. Build from local files

### Option 4: Use AppFlowy Desktop with Custom Backend

Consider using AppFlowy desktop application and connecting it to a custom backend API.

## Getting Help

1. Check AppFlowy documentation: https://docs.appflowy.io
2. Check AppFlowy GitHub issues
3. Review the build logs carefully - they now include detailed structure information
4. Verify the AppFlowy repository structure matches what the Dockerfile expects

## Next Steps After Fixing

Once the build succeeds:

1. Verify all services start: `docker-compose ps`
2. Check logs: `docker-compose logs`
3. Test health endpoint: `curl http://localhost:6633/health`
4. Access the application: `http://localhost:6633`

