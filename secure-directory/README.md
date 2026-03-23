# action-secure-checkout/secure-directory

An action that runs [Tetragon](https://github.com/cilium/tetragon) and applies a tracing policy to disallow writes any files in specified directory.

## What this action does

- Starts/runs Tetragon for runtime enforcement/observation.
- Applies a tracing policy that prevents write operations to files in specified directory.

## Why

This helps protect checked-out source code from in-job edition

## Behavior

- Read operations on secured directory remain allowed.
- Write attempts to secured files files are blocked by policy.
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
        uses: action/checkout
        with:
          dir: secure-dir
          
      - name: Guard checked out code with Tetragon
        uses: cilium/action-secure-checkout/secure-directory
        with:
          path: secure-dir
        
      - name: use securely checked out code
        run: |
           # This will succeed
           cat secure-dir/README.md
           
           # This will fail due to the policy
           echo "malicious edit" >> secure=dir/README.md
      
      - name: Normal checkout (untrusted)
        uses: actions/checkout@v3
        path: untrusted
        
      - name: use securely checked out code
        run: |
          # This will not fail because `untrusted` is not protected by the policy
          echo "malicious edit" >> untrusted/README.md
```


