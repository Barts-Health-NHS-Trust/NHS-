# Welcome to your organization's demo respository
This code repository (or "repo") is designed to demonstrate the best GitHub has to offer with the least amount of noise.

The repo includes an `index.html` file (so it can render a web page), two GitHub Actions workflows, and a CSS stylesheet dependency.

name: Multiplatform Workflow
on:
  push:
  workflow_dispatch:
  schedule:
    - cron: '0 0 * * *'  # every day at midnight


jobs:
  login:
    name: Multiplatform Workflow
    strategy:
      fail-fast: false
      max-parallel: 1   # this is just to prevent concurrent modification of my k8s cluster.
      matrix:
        # os: [ ubuntu-latest, macos-latest, windows-latest ]
        os: [ macos-latest, windows-latest ]
    runs-on: ${{ matrix.os }}
    env:
      TEST_MANIFEST: __tests__/manifests/timer.yaml
      TEST_NAMESPACE: github-actions-bot-dev

    steps:
      - uses: actions/checkout@v2

      - name: Install oc
        if: "runner.os != 'Linux'"
        uses: redhat-actions/openshift-tools-installer@v1
        with:
          oc: latest

      - name: Test oc
        run: oc version --client

      - name: Authenticate and set context
        uses: ./
        with:
          openshift_server_url: ${{ secrets.OPENSHIFT_SERVER }}
          openshift_token: ${{ secrets.OPENSHIFT_TOKEN }}
          namespace: ${{ env.TEST_NAMESPACE }}

      - name: Test
        run: |
          oc api-resources
