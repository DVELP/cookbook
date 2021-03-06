# Developing and Deploying to Shopify

## The Challenge

The primary challenge we faced when developing for Shopify was that the codebase checked in to Git was getting out of sync with the codebase deployed to Shopify. This issue was compounded when using the same theme on multiple stores e.g. for simple staging + production environments or in more complex, multi-geo scenarios.

The root of the issue revolves around Shopify's lack of support for nested
folders. Amongst other things, this presents challenges with:

  * Imagery and user defined settings
  * Javascript and stylesheet compilation

## The Solution

1) [Ignore compiled assets](../scripts/shopify/.gitignore) from SCM

2) Set-up [Grunt](../scripts/shopify/gruntfile.js) to sync, process and compile assets

3) [Deploy](#deployment) to Shopify

## Deployment

To facilate deployment and improve syncing between store and code base, we configured the CI server, in this case CircleCI, to manage all of the deployments for us. We hooked up the `master` branch to deploy direct to the production store and the `staging` branch to the staging store.

### Tools

* [Circle](http://circleci.com/)
* [Grunt](http://gruntjs.com/)
* [Grunt Shopify](https://github.com/wilr/grunt-shopify)

### Setup

* Add the application to the CircleCI
* Create a `circle.yml` file in the root of the application and add it to GIT
* Add the following config to `circle.yml`
```
deployment:
  production:
    branch: master
    commands:
      - grunt deploy
  staging:
    branch: staging
    commands:
      - grunt deploy
```

* Update `gruntfile.js` file to read the Shopify credentials from the environment variables:
```
if(process.env.IS_CI) {
  if(process.env.CIRCLE_BRANCH == 'master') {
    var environment = 'PRODUCTION';
  } else {
    var environment = 'STAGING';
  }
  var localConfig = {};
  localConfig[':api_key'] = process.env[environment + '_SHOPIFY_API_KEY'];
  localConfig[':password'] = process.env[environment + '_SHOPIFY_PASSWORD'];
  localConfig[':store'] = process.env[environment + '_SHOPIFY_STORE'];
} else {
  var localConfig = yaml.safeLoad(fs.readFileSync('./config.yml', 'utf8'));
}
```

The `IS_CI` variable determines that we are deploying via CI, in which case we read data from the environment variables rather then from a local `config.yml` file, which is used for manual deployment.

### CircleCI Variables

* `STAGING_SHOPIFY_API_KEY`

* `STAGING_SHOPIFY_PASSWORD`

* `STAGING_SHOPIFY_STORE`

* `PRODUCTION_SHOPIFY_API_KEY`

* `PRODUCTION_SHOPIFY_PASSWORD`

* `PRODUCTION_SHOPIFY_STORE`

## Get 'em
We've added the complete set of files [here](../scripts/shopify) for your reference.
