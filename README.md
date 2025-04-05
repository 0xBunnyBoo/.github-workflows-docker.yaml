name: build docker images

on:
  release:
    types: [created]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-go@v3
        with:
          go-version: "1.21.0"
          check-latest: true
      - uses: actions/checkout@v2
      - name: Prepare Release Variables
        id: vars
        uses: ignite/cli/actions/release/vars@main
      - name: build
        run: |
          GOOS=linux GOARCH=amd64 make
          echo "${{ secrets.GHCR_TOKEN }}" | docker login ghcr.io -u "${{ secrets.GHCR_USER }}" --password-stdin
          docker build . --tag ghcr.io/${{ github.repository }}:${{ steps.vars.outputs.tag_name }} -f docker/Dockerfile
          docker push ghcr.io/${{ github.repository }}:${{ steps.vars.outputs.tag_name }}
