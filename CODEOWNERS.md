# CODEOWNERS

This file dictates who is automatically requested for review
based on the file extension or directory modified.
Also defines who owns which parts of the codebase.
Must add this file to every repo to function.
This can be used as a starter/template for repos.
Place this file in the root, .github/, or docs/ directory.

## 1. The "Safety Net" or Global owners

If no other rule matches, these people are requested

* @K-Group-Companies/team-leads

## 2. Frontend Team

Any changes to UI code automatically tag the frontend squad

*.js                @K-Group-Companies/frontend-team
*.jsx               @K-Group-Companies/frontend-team
*.ts                @K-Group-Companies/frontend-team
*.tsx               @K-Group-Companies/frontend-team
*.css               @K-Group-Companies/frontend-team
*.scss              @K-Group-Companies/frontend-team
*.html              @K-Group-Companies/frontend-team

## 3. Backend / API Team

Any changes to C# or Python files tag the backend squad

*.cs                @K-Group-Companies/backend-team
*.py                @K-Group-Companies/backend-team

## 4. DevOps & Infrastructure

Critical infrastructure changes tag the DevOps team

/infra/             @K-Group-Companies/devops-team
/terraform/         @K-Group-Companies/devops-team
.github/workflows/  @K-Group-Companies/devops-team
*.yaml              @K-Group-Companies/devops-team

## 5. Security Critical

Changes to the payment gateway require a sign-off from the Tech Lead

src/payments/       @cklenk

## 6. Ignore Ownership

Ignored for generated files in /dist/ to prevent unnecessary review requests

dist/*              @K-Group-Companies/build-bot
