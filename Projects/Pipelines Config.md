```yaml
image: node:16

pipelines:
  branches:
    feature/*:
      - step:
          name: Install Dependencies
          caches:
            - node
          script:
            - echo "Installing dependencies..."
            - npm ci

    dev:
      - step:
          name: Install & Test
          caches:
            - node
          script:
            - echo "Installing dependencies and running tests..."
            - npm ci
            - npm run test:ci

      - step:
          name: Snyk Security Check
          script:
            - echo "Running Snyk Security Scan..."
            - npm install -g snyk
            - snyk test --severity-threshold=high

      - step:
          name: Deploy to Vercel
          deployment: development
          script:
            - echo "Deploying to Vercel..."
            - npm install -g vercel
            - vercel --token $VERCEL_TOKEN --prod

      - step:
          name: Semantic Release (Dev - Alpha)
          script:
            - echo "Running Semantic Release (Alpha)..."
            - npm install -g semantic-release
            - npx semantic-release --branches dev --tag-format "${BRANCH}-alpha"

    staging:
      - step:
          name: Deploy to Staging
          deployment: staging
          script:
            - echo "Deploying to Staging..."
            - npm install -g vercel
            - vercel --token $VERCEL_TOKEN --prod

      - step:
          name: Semantic Release (Staging - Beta)
          script:
            - echo "Running Semantic Release (Beta)..."
            - npm install -g semantic-release
            - npx semantic-release --branches staging --tag-format "${BRANCH}-beta"

    release:
      - step:
          name: Build & Push Docker Image
          image: docker:20.10.16
          services:
            - docker
          script:
            - echo "Building Docker image..."
            - docker build -t your-image-name:rc .

      - step:
          name: Push Docker Image to Registry
          image: docker:20.10.16
          services:
            - docker
          script:
            - echo "Pushing Docker image to Docker Hub/Quay..."
            - docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
            - docker tag your-image-name:rc $DOCKER_REGISTRY_URL/your-repo/your-image-name:rc
            - docker push $DOCKER_REGISTRY_URL/your-repo/your-image-name:rc

    main:
      - step:
          name: Release to Production
          deployment: production
          script:
            - echo "Deploying to Production..."
            - npm install -g vercel
            - vercel --token $VERCEL_TOKEN --prod

      - step:
          name: Semantic Release (Production)
          script:
            - echo "Running Semantic Release (Production)..."
            - npm install -g semantic-release
            - npx semantic-release --branches main

```