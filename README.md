<div align="center">
  <h1>🔄 figma-token-sync</h1>h1>
  <p>Automated pipeline: Figma Variables → Style Dictionary → production tokens.<br/>
    Keep design and code in perfect sync on every Figma publish.</p>p>

  [![License: MIT](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)](./LICENSE)
    [![Node.js](https://img.shields.io/badge/Node.js-18+-339933?style=for-the-badge&logo=node.js)](https://nodejs.org)
      [![Figma API](https://img.shields.io/badge/Figma_API-v1-F24E1E?style=for-the-badge&logo=figma)](https://www.figma.com/developers/api)
      </div>
      
      ---
      
      ## What This Does
      
      ```
      Figma Variables (design source of truth)
               │
                        │  Figma REST API (on publish webhook)
                                 ▼
                                 Raw token extraction and normalization
                                          │
                                                   │  Style Dictionary transforms
                                                            ▼
                                                            CSS Custom Properties · SCSS · JSON · TypeScript
                                                                     │
                                                                              │  GitHub Actions
                                                                                       ▼
                                                                                       Pull Request opened with token diff
                                                                                                │
                                                                                                         │  Team review and merge
                                                                                                                  ▼
                                                                                                                  npm publish → @nithishkd/tokens (new patch version)
                                                                                                                  ```
                                                                                                                  
                                                                                                                  ---
                                                                                                                  
                                                                                                                  ## Why This Exists
                                                                                                                  
                                                                                                                  Manual token exports break. Designers update Figma, forget to notify engineers, and production drifts from design. This pipeline makes drift **impossible** — the moment a Figma variable changes, a PR appears in GitHub with a precise diff.
                                                                                                                  
                                                                                                                  ---
                                                                                                                  
                                                                                                                  ## Setup
                                                                                                                  
                                                                                                                  ### 1. Install
                                                                                                                  
                                                                                                                  ```bash
                                                                                                                  npm install --save-dev figma-token-sync
                                                                                                                  ```
                                                                                                                  
                                                                                                                  ### 2. Configure
                                                                                                                  
                                                                                                                  Create `figma-sync.config.js`:
                                                                                                                  
                                                                                                                  ```javascript
                                                                                                                  module.exports = {
                                                                                                                    figmaFileId: 'YOUR_FIGMA_FILE_ID',
                                                                                                                      figmaToken: process.env.FIGMA_ACCESS_TOKEN,
                                                                                                                        outputDir: './packages/tokens/src',
                                                                                                                          transforms: ['css', 'scss', 'json', 'typescript'],
                                                                                                                            tiers: {
                                                                                                                                primitive: 'Primitives',
                                                                                                                                    semantic: 'Semantic',
                                                                                                                                        component: 'Component'
                                                                                                                                          }
                                                                                                                                          };
                                                                                                                                          ```
                                                                                                                                          
                                                                                                                                          ### 3. Add GitHub Action
                                                                                                                                          
                                                                                                                                          ```yaml
                                                                                                                                          # .github/workflows/figma-sync.yml
                                                                                                                                          name: Sync Figma Tokens
                                                                                                                                          on:
                                                                                                                                            repository_dispatch:
                                                                                                                                                types: [figma-publish]
                                                                                                                                                
                                                                                                                                                jobs:
                                                                                                                                                  sync:
                                                                                                                                                      runs-on: ubuntu-latest
                                                                                                                                                          steps:
                                                                                                                                                                - uses: actions/checkout@v4
                                                                                                                                                                      - uses: actions/setup-node@v4
                                                                                                                                                                              with:
                                                                                                                                                                                        node-version: 18
                                                                                                                                                                                              - run: npm ci
                                                                                                                                                                                                    - run: npx figma-token-sync
                                                                                                                                                                                                            env:
                                                                                                                                                                                                                      FIGMA_ACCESS_TOKEN: ${{ secrets.FIGMA_ACCESS_TOKEN }}
                                                                                                                                                                                                                            - name: Open PR with token changes
                                                                                                                                                                                                                                    uses: peter-evans/create-pull-request@v6
                                                                                                                                                                                                                                            with:
                                                                                                                                                                                                                                                      title: 'chore(tokens): sync from Figma'
                                                                                                                                                                                                                                                                branch: figma-sync/tokens-update
                                                                                                                                                                                                                                                                          commit-message: 'chore(tokens): automated sync from Figma Variables'
                                                                                                                                                                                                                                                                          ```
                                                                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                          ---
                                                                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                          ## How It Works
                                                                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                          1. Designer publishes Figma file with updated Variables
                                                                                                                                                                                                                                                                          2. Figma webhook fires → GitHub `repository_dispatch` event
                                                                                                                                                                                                                                                                          3. GitHub Action clones repo and runs `figma-token-sync`
                                                                                                                                                                                                                                                                          4. Tool calls Figma REST API, extracts all Variables from all collections
                                                                                                                                                                                                                                                                          5. Maps Variables to three-tier token structure
                                                                                                                                                                                                                                                                          6. Runs Style Dictionary transforms for all output formats
                                                                                                                                                                                                                                                                          7. Commits changed files and opens a PR with the diff
                                                                                                                                                                                                                                                                          8. Design/engineering team reviews — only real changes appear in diff
                                                                                                                                                                                                                                                                          9. Merge → CI runs → npm patch version published automatically
                                                                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                          ---
                                                                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                          ## Token Collection Mapping
                                                                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                          | Figma Collection | Token Tier | Output prefix |
                                                                                                                                                                                                                                                                          |---|---|---|
                                                                                                                                                                                                                                                                          | `Primitives` | Primitive | `color-blue-500` |
                                                                                                                                                                                                                                                                          | `Semantic` | Semantic | `color-action-primary` |
                                                                                                                                                                                                                                                                          | `Component` | Component | `button-background-default` |
                                                                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                          ---
                                                                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                          ## License
                                                                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                          MIT © [Nithish Kumar](https://github.com/nithish-kumar-design)</p></h1>
