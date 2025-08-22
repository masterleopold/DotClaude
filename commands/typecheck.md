---
allowed-tools: Bash, Read, Grep, Glob, TodoWrite
argument-hint: [optional: specific file or directory to check]
description: Check codebase for TypeScript type errors and report findings
---

Check the codebase for TypeScript type errors: $ARGUMENTS

## Quick Commands

### ⭐ **Recommended for Next.js/React Projects**

```bash
# MOST ACCURATE: Next.js build (handles JSX correctly)
npm run build

# Development type checking (if available)
npm run typecheck || npx tsc --noEmit

# Alternative for large codebases with timeout issues
npx tsc --noEmit --skipLibCheck --incremental
```

### 🔧 **Targeted Type Checking**

```bash
# Check specific directories (faster for large projects)
npx tsc --noEmit src/lib/**/*.ts --skipLibCheck
npx tsc --noEmit src/utils/**/*.ts --skipLibCheck

# Non-JSX files (avoid JSX compilation issues)
find src -name "*.ts" -not -path "*/node_modules/*" | xargs npx tsc --noEmit --skipLibCheck

# Check with JSX handling for React
npx tsc --noEmit --skipLibCheck --jsx react-jsx
```

### 📊 **Diagnostic Commands**

```bash
# Count TypeScript files
find . -name "*.ts" -o -name "*.tsx" | grep -v node_modules | wc -l

# Check tsconfig.json configuration
cat tsconfig.json | jq '.compilerOptions | {jsx, moduleResolution, paths}'
```

## ⚠️ **Critical: JSX Compilation in Next.js/React**

### **Expected False Positives:**
- ❌ **Direct `npx tsc`**: Shows hundreds/thousands of JSX errors
- ✅ **Next.js `npm run build`**: Handles JSX correctly via build pipeline
- ⚠️ **Reason**: TypeScript CLI doesn't use framework-specific JSX transforms

### **Error Categories:**

#### 1. **JSX Errors (FALSE POSITIVES)**
```
Cannot use JSX unless the '--jsx' flag is provided
```
**Status**: ✅ Expected in Next.js projects when using `npx tsc` directly

#### 2. **Module Resolution (REAL ISSUES)**
```
Cannot find module '@/utils/api'
Cannot find module '@/components/ui/button'
```
**Status**: 🔴 Requires investigation - check path aliases and file structure

#### 3. **React Import Issues (CONFIG ISSUE)**
```
Module can only be default-imported using the 'esModuleInterop' flag
```
**Status**: 🟡 Check tsconfig.json for `esModuleInterop: true`

### **Resolution Strategy:**
```bash
# 1. FIRST: Test actual build (most reliable)
npm run build

# 2. IF build fails: Check environment
npm run validate:env  # (Emergen-specific)

# 3. IF build succeeds but tsc fails: JSX issues are expected
echo "JSX errors in direct tsc are normal for Next.js projects"

# 4. Focus on non-JSX files for pure TypeScript checking  
npx tsc --noEmit src/lib src/utils src/types --skipLibCheck
```

## Approach
Systematically check the codebase for TypeScript compilation errors with framework-aware strategy.

## Updated Workflow:

### 1. **Project Detection & Strategy**
```bash
# Detect framework type
ls next.config.* package.json tsconfig.json

# Count codebase size for timeout considerations
find . -name "*.ts" -o -name "*.tsx" | grep -v node_modules | wc -l
```

### 2. **Framework-Appropriate Type Checking**
- **Next.js/React**: Prioritize `npm run build` over direct `tsc`
- **Node.js/Library**: Use `npx tsc --noEmit` directly
- **Large codebases (1000+ files)**: Use targeted/incremental approaches

### 3. **Error Categorization**
- **JSX Issues**: Identify as false positives for React projects
- **Module Resolution**: Flag as real issues requiring investigation
- **Import/Export**: Check configuration issues
- **Type Definitions**: Verify dependency types

### 4. **Intelligent Reporting**
- **For React/Next.js**: Emphasize build results over direct tsc
- **Timeout Handling**: Provide alternative strategies for large codebases
- **False Positive Filtering**: Separate framework-specific issues

### 5. **Actionable Recommendations**
- **Primary**: Framework build process results
- **Secondary**: Targeted type checking for specific areas
- **Configuration**: Suggest tsconfig.json improvements
- **Dependencies**: Identify missing type definitions

### 6. **Performance Considerations**
- **Incremental compilation** for large projects
- **Selective directory checking** to avoid timeouts
- **Skip lib checks** for faster execution
- **Parallel checking** where possible