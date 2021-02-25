<p align="center">
  <a href="https://console.amplify.aws">
    <img alt="Amplify" src="https://github.com/aws-amplify/community/blob/master/src/assets/images/logo-dark.png" width="60" />
  </a>
</p>
<h1 align="center">
  Amplify Console FAQ
</h1>

### Table of contents

- [Table of contents](#table-of-contents)
  - [Build fails with cannot find module aws-exports](#build-fails-with-cannot-find-module-aws-exports)
  - [How do I override a build timeout](#how-do-i-override-a-build-timeout)
  - [How do I pull private packages during a build.](#how-do-i-pull-private-packages-during-a-build)
  - [How do I run Amplify functions with python runtime](#how-do-i-run-amplify-functions-with-python-runtime)
  - [How to migrate domains to Amplify with minimal downtime](#how-to-migrate-domains-to-amplify-with-minimal-downtime)



#### Build fails with cannot find module aws-exports

The following error is generated when your app cannot find the `aws-exports.js` file.
```tsx
TS2307: Cannot find module 'aws-exports'
```

The aws-exports.js file is generated by the Amplify CLI during your backend build. Add the following code snippet to your build spec to resolve the error. This will create the aws-exports.js file for use in the build.

```tsx
backend:
  phases:
    build:
      commands:
        - '# Execute Amplify CLI with the helper script'
        - amplifyPush --simple
```

#### How do I override a build timeout
The default build timeout is 30 minutes, if your build takes more than 30 minutes, you can override the default build timeout using an env variable: _BUILD_TIMEOUT (App settings > Environment variables).

#### How do I pull private packages during a build.

Create your own key pair and add the private key as an environment variable in the Amplify app. Add the public key to the repository you would like to clone. You could then add the key to the ssh-agent on the build instance during build and git clone the second repository.

1. create the keypair without a password:

```
ssh-keygen -f deploy_key -N ""
```

2. encode it and copy the output into an Environment Variable in the Amplify Console (eg DEPLOY_KEY)

```
cat deploy_key | base64 | tr -d n

```

3. add the contents of deploy_key.pub to the access keys of the private repo you want to access

4. then in amplify.yml

```
commands:
        - eval "$(ssh-agent -s)"
        - ssh-add <(echo "$DEPLOY_KEY" | base64 -d)
```

#### How do I run Amplify functions with python runtime

[Amplify lambda functions need python 3.8.x or above](https://docs.amplify.aws/cli/function#function-templates). Our build image supports both python 3.7.9 which is aliased under `python3`, and python 3.8 which is aliased under `python3.8`. In order to you use `python3` with python 3.8 version, use the following commands (*create symlink to link python3 to python3.8 and install pipenv using pip3.8*).

```
version: 1
backend:
  phases:
    build:
      commands:
        - update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.8 11
        - /usr/local/bin/pip3.8 install --user pipenv
        - amplifyPush --simple
```

#### How to migrate domains to Amplify with minimal downtime

[The best process to follow to minimize downtime here would be](#how-to-migrate-domains-to-amplify-with-minimal-downtime): 


In your Amplify Console app, open the domain management screen

 1. Type in your root domain (yourdomain.com) and click configure
 2. Click “Exclude root” button
 3. Click “Remove” button next to the “www” sub domain that was automatically added
 4. Add a sub domain that you don't use elsewhere for testing purposes
 5. Click save
  
What we have now done, is started the process of creating and verifying a domain in Amplify Console, without adding any `CNAMEs` to the Amplify Console CloudFront Distribution.

Now follow the instructions for setting up the verification `CNAME` so that your new custom domain will receive a SSL certificate.

While this process is completing, please check the TTL on the DNS records you are moving to Amplify Console. You want them to be as small as possible so that the change propagates quickly. If you have to change this value, please be sure to wait out at minimum, the original TTL before continuing, to make sure that your new TTL is in effect.

Once that is done and the domain is marked as available, and you've tested with the extra sub domain you created in step 5 above, you’re now ready to do the migration.

You will need to do the following in quick succession:

Change the DNS record (the records will all have the same destination as the sub domain you created for testing in step 5 above)
Remove the CNAME from your Cloudfront distribution.
Add the CNAME to Amplify Console by going to the domain management page, and clicking “Manage subdomains”
Doing it following this method you should see very little downtime, and will mainly depend on the TTL of the DNS record.
