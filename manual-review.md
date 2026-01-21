| Type | Path | Short Reasoning |
| :--- | :--- | :--- |
| Broken Access Control | `server/trpc/router/match/getById.ts` | No ownership check; any user can fetch any match data. |
| Broken Access Control | `server/trpc/router/match/update.ts` | No ownership check; any user can update match status/dates. |
| Broken Access Control | `server/trpc/router/file/index.ts` | Missing check for AGENT role; agents can delete any file. |
| Broken Access Control | `server/trpc/router/internal/getById.ts` | Unrestricted access; any user can query internal employee data. |
| Broken Access Control | `server/trpc/router/pdf/index.ts` | `generateStudentProfile` checks visibility but not application relationship. |
| Broken Access Control | `apps/web/src/server/trpc/router/mail/toCompany/sentNoExtPortal.ts` | Missing ownership check for `applicationId`; potential IDOR. |
| Auth Bypass | `apps/web/src/server/trpc/router/company/getById.ts` | Critical: Query uses raw `input.id` instead of validated `id` variable. |
| Data Exposure | `apps/web/src/server/trpc/router/application/getById.ts` | Fetches all applications for a vacancy, potentially exposing excess data. |
| Missing Authentication | `actions.ts` | `invalidateSession` function lacks authentication checks. |
| Missing Authentication | `actions.ts` | Action logic (`auth check`) lacks authentication checks. |
| Missing Authentication | `layout.tsx` | `serverFunction` lacks authentication checks. |
| Input Validation | `actions.ts` | `updateStudentVisibility` lacks input validation. |
| Input Validation | `actions.ts` | `updateUserLanguage` lacks input validation. |
| Input Validation | `actions.ts` | General action input lacks validation. |
| Input Validation | `companies.ts` | Handler function lacks input validation. |
