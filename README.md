# action-secure-checkout

A secure wrapper around `actions/checkout` that runs [Tetragon](https://github.com/cilium/tetragon) and applies a tracing policy to disallow writes to checked-out code.

## What this action does

- Executes a standard repository checkout (via `actions/checkout`).
- Starts/runs Tetragon for runtime enforcement/observation.
- Applies a tracing policy that prevents write operations to files in the checked-out workspace.

## Why

This helps protect checked-out source code from in-job edition

## Behavior

- Read operations on checked-out code remain allowed.
- Write attempts to checked-out files are blocked by policy.
- Policy is intended to reduce supply-chain and CI tampering risk during workflow execution.

## Usage

```yaml
name: CI

on:
  pull_request_target: {}

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Secure checkout
        uses: mkwiek/action-secure-checkout@v1
        
      - name: use securely checked out code
        run: |
           # This will succeed
           cat README.md
           
           # This will fail due to the policy
           echo "malicious edit" >> README.md
      
      - name: Normal checkout (untrusted)
        uses: actions/checkout@v3
        path: untrusted
        
      - name: use securely checked out code
        run: |
          # This will not fail because `untrusted` is not protected by the policy
          echo "malicious edit" >> untrusted/README.md
```


