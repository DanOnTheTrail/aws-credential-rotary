<h1 align="center">
  🔄
  <br/>
  AWS Credential Rotary
</h1>

<p align="center">
   A GitHub action for rotate AWS credentials stored in GitHub secrets
</p>

<div align="center">
  <a href="https://github.com/softprops/aws-credential-rotary/actions">
		<img src="https://github.com/softprops/aws-credential-rotary/workflows/Main/badge.svg"/>
	</a>
</div>

<br />

## 🤔 why bother

AWS assumes a shared security responsibility model with you and it's services.

It goes to great lengths to secure your privacy and access to services which your users depend on.
It also assumes that you are doing the same with the credentials that permit access to thoae services and data.
AWS [documents some helpful best practices](https://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html) to manage that.

One of those practices is ensuring you are periodically rotating your credentials. The longer lived your credentials are, the greater the opportunity of inviting unwanted and unintential breach of your aws managed systems and data is.

In short, it is much easier to rotate your credentials than to cope with the aftermath of a data access breach. 

## 🤸 usage

This action depends on the ability to update repository secrets. As such it requires an GitHub api token with `repo` permissions. Create a personal access token with `repo` permissions,  store that in GitHub secrets, and provide that as `GITHUB_TOKEN` environment variable.

This action also depends on having the ability to list, create, and delete iam access keys.

You will need to provide at a minimum an `iam-user-name` to for the action to fetch the members access keys.

The example below rotates credentials just before they are used

```diff
name: Main

on: push

jobs:
  main:
    runs-on: ubuntu-latest
    steps:
+     - name: Rotate credentials
+       uses: softprops/aws-credential-rotary@master
+       with:
+           iam-user-name: 'name-of-iam-user-associated-with-credentials'
+       env:
+         GITHUB_TOKEN: ${{ secrets.REPO_GITHUB_TOKEN }}
+         AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
+         AWS_SECRET_ACCESS_TOKEN: ${{ secrets.AWS_SECRET_ACCESS_TOKEN }}
      - name: Print Create Date
        run: aws iam list-access-keys --user name-of-iam-user-associated-with-credentials --query 'AccessKeyMetadata[0].CreateDate' --output text
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Rotating on a schedule

It is recommended to rotate credentials on a [schedule](https://docs.github.com/en/actions/reference/events-that-trigger-workflows#scheduled-events). You can find some [example schedules here](https://crontab.guru/examples.html)

```diff
name: Rotate AWS Credentials

+ on:
+  schedule:
+    # At 00:00 on Sunday.
+    - cron:  '0 0 * * 0'

jobs:
  main:
    runs-on: ubuntu-latest
    steps:
     - name: Rotate credentials
       uses: softprops/aws-credential-rotary@master
       with:
           iam-user-name: name-of-iam-user-associated-with-credentials'
       env:
         GITHUB_TOKEN: ${{ secrets.REPO_GITHUB_TOKEN }}
         AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
         AWS_SECRET_ACCESS_TOKEN: ${{ secrets.AWS_SECRET_ACCESS_TOKEN }}
      - name: Print Create Date
        run: aws iam list-access-keys --user name-of-iam-user-associated-with-credentials --query 'AccessKeyMetadata[0].CreateDate' --output text
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Custom secret names

By default, this action will assume the credentials to be rotated exist as secrets named `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_TOKEN`. You can override these 


```diff
name: Main

on: push

jobs:
  main:
    runs-on: ubuntu-latest
    steps:
      - name: Rotate credentials
        uses: softprops/aws-credential-rotary@master
        with:
            iam-user-name: 'name-of-iam-user-associated-with-credentials'
+           github-access-key-id-name: 'CUSTOM_ACCESS_KEY_ID_NAME'
+           github-secret-access-key-name: 'CUSTOM_SECRET_ACCESS_KEY_NAME'
        env:
          GITHUB_TOKEN: ${{ secrets.REPO_GITHUB_TOKEN }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_TOKEN: ${{ secrets.AWS_SECRET_ACCESS_TOKEN }}
      - name: Print Create Date
        run: aws iam list-access-keys --user name-of-iam-user-associated-with-credentials --query 'AccessKeyMetadata[0].CreateDate' --output text
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

#### inputs

| Name        | Type    | Description                                                     |
|-------------|---------|-----------------------------------------------------------------|
| `iam-user-name`   | string  | AWS IAM username associated with credentials to be rotated                         |
| `github-access-key-id-name`      | string  | GitHub secret name used to store AWS access key id. Defaults to AWS_ACCESS_KEY_ID                |
| `github-secret-access-key-name`      | string  | GitHub secret name used to store AWS access key secret. Defaults to AWS_SECRET_ACCESS_KEY                |



Doug Tangren (softprops) 2020.
