---
allowed-tools: Read, Edit, MultiEdit, Bash, Grep, Glob, TodoWrite, WebFetch
argument-hint: [error message or build log]
description: Fix Vercel deployment build errors
---

Tried to deploy to Vercel but got this build error: $ARGUMENTS

## Systematic Debugging Approach:

1. **Analyze the error message**
   - Parse the full error output carefully
   - Identify the specific error type (build, dependency, configuration, etc.)
   - Note the exact file and line number if provided
   - Check for common Vercel-specific issues

2. **Common Vercel build issues to check**
   - **Node version mismatch**: Check package.json engines field
   - **Missing dependencies**: Ensure all deps are in package.json (not devDependencies for production deps)
   - **Build command issues**: Verify build script in package.json
   - **Environment variables**: Check if required env vars are set in Vercel dashboard
   - **File path case sensitivity**: Linux is case-sensitive (Vercel runs on Linux)
   - **Module resolution**: Check for incorrect import paths
   - **Memory limits**: Build might exceed Vercel's limits

3. **Investigate project configuration**
   - Check `vercel.json` configuration if exists
   - Review `package.json` scripts and dependencies
   - Verify Next.js config (`next.config.js`) if applicable
   - Check for `.vercelignore` file issues
   - Review build and output directories

4. **Fix based on error type**
   - **Dependency errors**: 
     * Run `npm install` or `yarn install` locally
     * Check package-lock.json or yarn.lock is committed
     * Move necessary devDependencies to dependencies
   - **Build script errors**:
     * Test build locally with `npm run build`
     * Check for missing build tools or scripts
   - **TypeScript errors**:
     * Fix type errors
     * Consider adding `// @ts-ignore` temporarily if needed
   - **ESLint errors**:
     * Fix linting issues or adjust rules
     * Add `.eslintignore` if needed

5. **Vercel-specific solutions**
   - Add or update `vercel.json`:
     ```json
     {
       "builds": [{"src": "package.json", "use": "@vercel/node"}],
       "functions": {"api/*.js": {"maxDuration": 10}}
     }
     ```
   - Set Node version in package.json:
     ```json
     "engines": {"node": "18.x"}
     ```
   - Configure build command and output directory
   - Adjust framework preset if needed

6. **Test the fix**
   - Run build command locally first
   - Check if all files are properly committed
   - Ensure no large files exceed Vercel limits
   - Verify environment variables match production needs

7. **Document the solution**
   - Update README.md with deployment instructions
   - Add troubleshooting section for future reference
   - Document any Vercel-specific configurations
   - Note environment variables needed for deployment