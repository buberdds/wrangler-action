# Wrangler action with publish URL 

This action extends core [Wrangler GitHub Action](https://github.com/cloudflare/wrangler-action) by adding publish URL to `GITHUB_OUTPUT`

## Usage
- add id to your Wrangler deploy task
- grab publish URL via `steps.wrangler.outputs.url`

### Example action with multiple deploys and comment

```
name: CloudFlare

on:
  push:
    branches: [master]
  pull_request:
    branches: [master]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 16.x
          cache: 'yarn'
      - run: yarn install --frozen-lockfile

      - run: yarn build-preview && yarn build-storybook
      - name: Deploy App to Cloudflare Workers
        id: wrangler
        uses: buberdds/wrangler-action@1.0.2
        with:
          apiToken: ${{ secrets.CLOUDFRONT_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFRONT_ACCOUNT_ID }}
          command: pages publish "./build" --project-name=<YOUR_CLOUDFLARE_PROJECT_NAME>

      - name: Deploy Storybook to Cloudflare Workers
        id: wrangler-storybook
        uses: buberdds/wrangler-action@2.0.0
        with:
          apiToken: ${{ secrets.CLOUDFRONT_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFRONT_ACCOUNT_ID }}
          command: pages publish "./storybook-static" --project-name=<YOUR_CLOUDFLARE_PROJECT_NAME>

      - name: Add Comment to PR
        uses: peter-evans/create-or-update-comment@v1
        with:
          issue-number: ${{ github.event.pull_request.number }}
          body: |
            App has been deployed to: ${{ steps.wrangler.outputs.url }}
            Storybook has been deployed to: ${{ steps.wrangler-storybook.outputs.url }}
```



