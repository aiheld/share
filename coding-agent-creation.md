You can turn your “fix Snyk vulnerability” prompt into a reusable custom agent, then select that agent in the GitHub.com coding agent UI for issues/PRs on that repo.[1][2]

## 1. Decide where the agent should live

- If this agent is only for one project, put it in that repository (repo‑level agent).[2]
- If you want to share it across many repos in your org/enterprise, use the configured `.github` / `.github-private` “agents” repo (org/enterprise‑level agent).[3][2]

## 2. Create the custom agent in GitHub UI

1. Go to `https://github.com/copilot/agents` while signed in.[4][1]
2. In the prompt box at the top, use the dropdowns to select:
   - The **repository** where you want this agent to be available.
   - Optionally the **branch** (usually `main`).[1][4]
3. Click the **+** button (Create) near the prompt box, then choose **Create an agent**.[5][4][1]
4. GitHub opens a new file like `.github/agents/my-agent.agent.md` in that repo with a template.[1]

## 3. Configure the agent profile for your Snyk flow

1. In the `my-agent.agent.md` file, edit the YAML front‑matter at the top: set a clear `name` (for example, `Snyk Vulnerability Fixer`) and `description` explaining its job.[6][1]
2. Under `tools`, ensure it can:
   - Read and search code (`read`, `search`).
   - Edit files / create branches / open PRs (`edit`, `pull-requests`, etc.), as needed for your workflow.[7][8]
3. Below the YAML, paste or adapt the prompt you are already using in VS Code chat, but write it as instructions to the agent, for example:
   - What vulnerabilities to look for (from Snyk reports).
   - How to analyze and suggest fixes.
   - How to modify files and propose a PR (style, commit messages, etc.).[6][1]
4. Commit the file to the repo (and open a PR if needed), then merge it into the default branch so Copilot can use it.[4][1]

## 4. Use the agent in GitHub.com coding agent

1. Open the **Code** tab of the target repository on GitHub.com.[9][8]
2. Open the Copilot coding agent panel (for example from the **Copilot** button at the top right, or from an issue/PR sidebar, depending on your UI).[8][9]
3. In the agents dropdown (it usually shows “Copilot coding agent” by default), select your new `Snyk Vulnerability Fixer` agent.[5][1]
4. Run your task prompt, for example:
   - “Scan the latest Snyk report in this repo and propose fixes, then open a PR with changes.”  
   The coding agent will execute with the behavior and tools defined in your custom agent profile.[8][1]

## 5. Keep VS Code and GitHub UI in sync

- Because the agent definition lives in `.github/agents/*.agent.md` in the repo, any changes you commit (from VS Code or GitHub UI) update behavior in both environments.[10][2]
- You can refine the prompt, tools, and even restrict the agent to GitHub or VS Code via the `target` property if needed.[7][1]

If you paste your current VS Code prompt (redacting any secrets), a more tailored agent profile structure can be suggested.

[1](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents)
[2](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-custom-agents)
[3](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-agents/prepare-for-custom-agents)
[4](https://docs.github.com/en/copilot/tutorials/customization-library/custom-agents/your-first-custom-agent)
[5](https://docs.github.com/zh/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents)
[6](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
[7](https://docs.github.com/zh/enterprise-cloud@latest/copilot/reference/custom-agents-configuration)
[8](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent)
[9](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/manage-agents)
[10](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
[11](https://github.blog/news-insights/product-news/your-stack-your-rules-introducing-custom-agents-in-github-copilot-for-observability-iac-and-security/)
[12](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment)
[13](https://www.youtube.com/watch?v=XoLH00AJnX4)
[14](https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/agent-builder-build-agents)
[15](https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot)
[16](https://google.github.io/adk-docs/agents/custom-agents/)
[17](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents?search-overlay-input=GitHub+Copilot+workflow&search-overlay-ask-ai=true)
[18](https://docs.github.com/en/copilot/tutorials/customization-library/custom-agents)
[19](https://github.blog/ai-and-ml/github-copilot/how-to-write-a-great-agents-md-lessons-from-over-2500-repositories/)
[20](https://docs.github.com/en/enterprise-cloud@latest/copilot/reference/custom-agents-configuration)