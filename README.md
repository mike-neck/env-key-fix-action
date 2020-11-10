# env-key-fix-action
A GitHub Action to collect environmental vairables matching given profile and to configure environmental variables.

# Use Cases

Imagine, your application having two stages (for example, `production` and `development`), and using environmental keys/values pair for them.

e.g.

```
├── development
│   ├── BUCKET_DEVELOPMENT
│   ├── AWS_ACCESS_KEY_ID_DEVELOPMENT
│   └── AWS_SECRET_ACCESS_KEY_DEVELOPMENT
└── production
    ├── BUCKET_PRODUCTION
    ├── AWS_ACCESS_KEY_ID_PRODUCTION
    └── AWS_SECRET_ACCESS_KEY_PRODUCTION
```

If you want to deploy application, you may using a separate yamls for each stages.

##### `deploy-development.yml`

```yaml
- name: Put S3
  run: aws s3 sync ./build "s3://${ BUCKET }"
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID_DEVELOPMENT }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY_DEVELOPMENT }}
    BUCKET: ${{ secrets.BUCKET_DEVELOPMENT }}
```

##### `deploy-production.yml`

```yaml
- name: Put S3
  run: aws s3 sync ./build "s3://${ BUCKET }"
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID_PRODUCTION }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY_PRODUCTION }}
    BUCKET: ${{ secrets.BUCKET_PRODUCTION }}
```

This way is redundant, if some changes is to be added to the workflow yaml.

Using this action, the workflows file may become into single file.


##### `deploy.yml`

```yaml
- name: Determine stage
  run: echo "STAGE=$(echo "${REF}" | sed -e "s/refs\///g" -e "s/heads\///g")" >> $GITHUB_ENV
  env:
    REF: ${{ github.ref }}

- name: Determine environmental variables
  uses: mike-neck/fix-profile-action
  with:
    profile: ${{ env.STAGE }}
    ignore-case: true
  env:
    AWS_ACCESS_KEY_ID_PRODUCTION: ${{ secrets.AWS_ACCESS_KEY_ID_PRODUCTION }}
    AWS_SECRET_ACCESS_KEY_PRODUCTION: ${{ secrets.AWS_SECRET_ACCESS_KEY_PRODUCTION }}
    BUCKET_PRODUCTION: ${{ secrets.BUCKET_PRODUCTION }}
    AWS_ACCESS_KEY_ID_DEVELOPMENT: ${{ secrets.AWS_ACCESS_KEY_ID_DEVELOPMENT }}
    AWS_SECRET_ACCESS_KEY_DEVELOPMENT: ${{ secrets.AWS_SECRET_ACCESS_KEY_DEVELOPMENT }}
    BUCKET_DEVELOPMENT: ${{ secrets.BUCKET_DEVELOPMENT }}

- name: Put S3
  run: aws s3 sync ./build "s3://${ BUCKET }"
```

In the example above, when running at `development` environment, this action picks up a three environmental variables, and adjust the names of them.

- `AWS_ACCESS_KEY_ID_DEVELOPMENT` --> `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY_DEVELOPMENT` --> `AWS_SECRET_ACCESS_KEY`
- `BUCKET_PDEVELOPMENT` --> `BUCKET`
