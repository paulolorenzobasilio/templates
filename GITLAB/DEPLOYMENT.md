```
image: node:11.10.1

cache:
  untracked: true
  paths:
  - node_modules/

# create key to access ssh server

before_script:
- 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
- mkdir -p ~/.ssh
- chmod 700 ~/.ssh
- echo "$DEPLOY_STAGING_PRIVATE_KEY" > ~/.ssh/staging.pem
- chmod 400 ~/.ssh/staging.pem
- echo "$DEPLOY_PRODUCTION_PRIVATE_KEY" > ~/.ssh/production.pem
- chmod 400 ~/.ssh/production.pem
- mkdir -p ~/.deploy
- '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
- npm install 

stages:
- test
- deploy

test:
  stage: test
  except:
  - master
  script:
  - cp .env.staging.dist .env
  - npm run clear-cache
  - npm run test

deploy_to_staging:
  stage: deploy
  only:
  - master
  environment:
    name: staging
    url: http://$DEPLOY_STAGING_HOST
  script:
  - echo "Building..."
  - cp .env.staging.dist .env
  - npm run build
  - echo "Packaging..."
  - tar -czf ~/.deploy/package.tar.gz .nuxt/dist/
  - echo "Deploying..."
  - ssh -i ~/.ssh/staging.pem $DEPLOY_STAGING_USERNAME@$DEPLOY_STAGING_HOST "cd $DEPLOY_STAGING_KNIT_PATH && git pull origin master"
  - scp -i ~/.ssh/staging.pem ~/.deploy/package.tar.gz $DEPLOY_STAGING_USERNAME@$DEPLOY_STAGING_HOST:$DEPLOY_STAGING_KNIT_PATH
  - ssh -i ~/.ssh/staging.pem $DEPLOY_STAGING_USERNAME@$DEPLOY_STAGING_HOST "cd $DEPLOY_STAGING_KNIT_PATH && tar -xzf package.tar.gz && rm package.tar.gz && pm2 restart knit"

deploy_to_production:
  stage: deploy
  only:
  - tags
  environment:
    name: production
    url: http://$DEPLOY_PRODUCTION_HOST
  script:
  - echo "Building..."
  - cp .env.production.dist .env
  - npm run generate --verbose
  - echo "Packaging..."
  - cp robots.txt dist/robots.txt
  - tar -czf ~/.deploy/package.tar.gz dist/
  - rm -Rf node_modules/*
  - echo "Deploying..."
  - scp -vvv -i ~/.ssh/staging.pem ~/.deploy/package.tar.gz $DEPLOY_PRODUCTION_USERNAME@$DEPLOY_PRODUCTION_HOST:$DEPLOY_PRODUCTION_PATH
  - ssh -vvv -i ~/.ssh/staging.pem $DEPLOY_PRODUCTION_USERNAME@$DEPLOY_PRODUCTION_HOST "cd $DEPLOY_PRODUCTION_PATH && tar -xzf package.tar.gz && rm package.tar.gz"

```