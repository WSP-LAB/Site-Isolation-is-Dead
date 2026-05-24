# Site Isolation is Dead: How Site Isolation is Broken in Agentic Browsers and Extensions

This repository provides the official artifacts for the IEEE S&P 2026 paper,
**"Site Isolation is Dead: How Site Isolation is Broken in Agentic Browsers and
Extensions"**.

It includes (1) a **demo video** illustrating an end-to-end paper content
exfiltration scenario via our Malicious Prompt Injection (MPI) attack, and (2)
**patches** that implement our proposed guardrail system to detect and mitigate
Indirect Prompt Injection (IPI) attacks. These patches are provided for two
open-source agentic systems evaluated in our study:
[Nanobrowser](https://github.com/nanobrowser/nanobrowser) and [BrowserOS
Agent](https://github.com/browseros-ai/old-browseros-agent).

## Background: MPI and SLI Attacks

Our research demonstrates that modern agentic browsers and extensions inherently
trust Inter-Process Communication (IPC) channels, allowing a compromised
renderer process to bypass site isolation. We introduce two novel attacks:

* **Malicious Prompt Injection (MPI):** Exploits forged IPC messages to coerce
the agent into executing attacker-chosen cross-site actions.

* **Stealing LLM-accessible Information (SLI):** Exploits IPC requests to
exfiltrate sensitive data (e.g., conversation history, API keys) from
extension-scoped local storage.

## Demo Video

[![Demo](/thumbnail.jpg)](https://www.youtube.com/watch?v=QWLxxc_n7yo)

This video is a demo of a paper content exfiltration scenario utilizing our MPI
attack. The video depicts a victim user visiting an attacker-controlled webpage
using BrowserOS. Through an MPI attack, the injected prompt instructs the agent
to automatically:

1. Navigate to a HotCRP submission site.
2. Open the victim's submitted paper.
3. Read the paper's abstract.
4. Exfiltrate the abstract to an attacker-controlled server via a URL query
parameter.

## Full Renderer-to-Agent Exploit PoC

Our end-to-end attack chain initiates with arbitrary code execution within the
renderer by exploiting memory corruption vulnerabilities. In adherence to
responsible disclosure practices, we do not release concrete Proof-of-Concept
(PoC) exploits in this repository, as several evaluated systems have not yet
deployed patches.

Our PoC attack framework was developed on top of the following resource:

> [https://github.com/Petitoto/chromium-exploit-dev](https://github.com/Petitoto/chromium-exploit-dev)

Researchers seeking to validate the renderer-to-agent site isolation bypass path
may utilize this framework as a starting point.

## Repository Layout
```text
.
├── browseros.patch       # Guardrail patch for BrowserOS Agent
├── nanobrowser.patch     # Guardrail patch for Nanobrowser
├── browseros-agent/      # BrowserOS Agent submodule (pinned)
└── nanobrowser/          # Nanobrowser submodule (pinned)
```

## Guardrail Design

To mitigate IPI attacks, our patches integrate a **Verifier LLM** into the
agent's execution pipeline, positioned directly between the reasoning components
(planner/navigator) and the final action executor. Before each agent action is
executed, the verifier evaluates the proposed action against a trusted
context---comprising the original user prompt, the high-level plan, and the
history of previous actions---to identify adversarial deviations. If an IPI
attempt is detected, the verifier immediately halts the task and displays a
security warning on the task panel.

## Installation

### Step 1: Applying the Patches

1. Load the original versions of Nanobrowser and BrowserOS Agent:

```bash
git submodule update --init --recursive
```

2. Apply the patch for Nanobrowser:

```bash
cd nanobrowser
git apply ../nanobrowser.patch
```

3. Apply the patch for BrowserOS Agent:

```bash
cd browseros-agent
git apply ../browseros.patch
```

### Step 2: Building the Patched Extensions

1. Building the Nanobrowser extension (*Note: It requires Node.js >= 22.12.0 and
pnpm 9.15.1*)

```bash
cd nanobrowser

# Install dependencies
pnpm install

# Build the extension (output: dist/)
pnpm build
```

2. Building the BrowserOS agent extension (*Note: It requires Yarn*)

```bash
cd browseros-agent

# Install dependencies
yarn install

# Build the extension (output: dist/)
yarn build
```

### Step 3: Loading the Extensions into the Browser

For Nanobrowser:

1. Open Chrome and navigate to `chrome://extensions`
2. Enable **Developer mode** (toggle in the top-right corner)
3. Click **Load unpacked**
4. Select the `nanobrowser/dist/` directory
5. The extension will appear in your toolbar

For BrowserOS Agent:

1. Download and install [BrowserOS 0.28.1](https://github.com/browseros-ai/BrowserOS/releases/tag/v0.28.1)
2. Navigate to `chrome://extensions` within BrowserOS
3. Enable **Developer mode**
4. Click **Load unpacked**
5. Select the `browseros-agent/dist/` directory
6. The patched agent will replace the built-in version

## Evaluation Setup

For the evaluation detailed in the paper, we utilized **`gemini-2.5-flash`** as
the backend LLM for both Nanobrowser and BrowserOS Agent, as well as for the
Verifier LLM. Both extensions support configurable LLM backends via their
respective settings UIs. To replicate our setup, set the model to
`gemini-2.5-flash` and supply a valid Gemini API key prior to execution.

## License

This repository contains patches and derivative works combining submodules under
different licenses. The original submodules retain their respective licenses:

- Nanobrowser: Apache-2.0 License
- BrowserOS Agent: AGPL-3.0 License

In compliance with the strong copyleft requirements of the AGPL-3.0 license, the
modified patches and the combined project framework provided in this repository
are released under the GNU Affero General Public License v3.0 (AGPL-3.0) for
research purposes.

## Citation

If you build upon this work or utilize these artifacts in your research, please
cite our paper:

```bibtex
@INPROCEEDINGS{lee:oakland:2026,
  author = {Suyoung Lee and Seongho Keum and Changoo Lee and Dongwon Shin and Sanghyun Hong and Byoungyoung Lee and Sooel Son},
  title = {Site Isolation is Dead: How Site Isolation is Broken in Agentic Browsers and Extensions},
  booktitle = {Proceedings of the {IEEE} Symposium on Security and Privacy},
  pages = {1804--1821},
  year = 2026
}
```
