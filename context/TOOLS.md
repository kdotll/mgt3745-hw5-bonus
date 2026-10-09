## TOOLS.md

The ledger of Trust Boundary crossings. One row per external service this repository depends on. Read by the agent on every task, so keep it short; a service not in use does not belong here.

Never put a credential in this file. A key, token, or password anywhere in the repository is graded as a security failure regardless of the rest.

Each crossing statement answers three questions in one first-person sentence: what crosses, to whom, and who is accountable.

| Service | Trusted with | Credentials live | Crossing statement | Switching cost |
| :--- | :--- | :--- | :--- | :--- |
| Cloudflare Workers + D1 | Evaluated candidate submissions, IP addresses, request timestamps, edge runtime logs | Cloudflare dashboard session; CLI token in environment (`CLOUDFLARE_API_TOKEN`), zero keys in repo | Evaluated candidate records cross from recruiter client browsers over HTTPS to Cloudflare Workers and D1 database under Cloudflare terms of service, and I am accountable. | Medium: `wrangler d1 export` to retrieve data, rewrite one worker fetch router for another Node or serverless host. |
| GitHub + Codespaces | Source code, commit history, devcontainer configuration | GitHub account session, zero tokens stored in repository files | Project source code, configuration files, and git history cross from development machines to GitHub servers under GitHub terms of service, and I am accountable. | Medium: Export git commit history and configure a new remote repository on GitLab or Bitbucket. |
| GitHub Copilot | Everything in the repository, as context for suggestions | Authenticated GitHub account session | Workspace code buffers and context files cross to GitHub Copilot inference servers under GitHub Copilot enterprise terms, and I am accountable. | Low: Disabling the Copilot extension leaves pure vanilla JavaScript, HTML, and CSS requiring no codebase modifications. |
| wrangler (npm) | Local build artifacts, terminal execution commands, local D1 migration scripts | Local developer environment shell session; no secrets stored in package files | Deployment commands and build artifacts cross from local workstation via wrangler CLI to Cloudflare API endpoints under npm open-source license, and I am accountable. | Low: Can be replaced with direct Cloudflare REST API calls or dashboard web UI uploads without changing application code. |
| bolt.new | Client UI component code, CSS tokens, and filter logic | Ephemeral browser session; zero persistent tokens | Frontend specification and UI component requirements cross from developer prompt to bolt.new sandbox under WebContainer terms of service, and I am accountable. | Low: Output is exported as standard vanilla JS/HTML/CSS; easily migrated to another LLM or written manually. |

## Revisit triggers

- A new service is added to the repository.
- A vendor changes pricing, terms, or region.
- A credential moves.