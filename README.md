pip install "paddleocr==2.7.3" "paddlex==2.1.0" "paddlepaddle==2.6.2"



   pip uninstall paddlepaddle
   git clone https://github.com/PaddlePaddle/Paddle.git && cd Paddle
   git checkout release/3.2  # Match your required version
   mkdir build && cd build
   cmake .. -DWITH_GPU=OFF -DWITH_AVX=OFF -DWITH_TESTING=OFF -DCMAKE_BUILD_TYPE=Release -DPY_VERSION=3.10
   make -j$(nproc)
   pip install python/dist/paddlepaddle-*.whl


stages:
  - test
  - build
  - deploy

variables:
  DOCKER_IMAGE: <your-registry>.cr.de-fra.ionos.com/sample-python-project
  DOCKER_TAG: latest

test:
  stage: test
  script:
    - pip install -r requirements.txt
    - pytest tests/

build:
  stage: build
  script:
    - docker build -t $DOCKER_IMAGE:$DOCKER_TAG .

deploy:
  stage: deploy
  script:
    - echo "$IONOS_REGISTRY_PASSWORD" | docker login <your-registry>.cr.de-fra.ionos.com -u "$IONOS_REGISTRY_USERNAME" --password-stdin
    - docker push $DOCKER_IMAGE:$DOCKER_TAG
  only:
    - main




stages:
  - test
  - build
  - deploy

variables:
  DOCKER_REGISTRY: "abc123.cr.de-fra.ionos.com"
  DOCKER_IMAGE: "$DOCKER_REGISTRY/sample-python-project"

test:
  stage: test
  image: python:3.12

  script:
    - pip install -r requirements.txt
    - pytest tests/


build:
  stage: build

  script:
    # Get the latest version from Git tags
    - |
      LATEST_TAG=$(git tag --sort=-v:refname | head -n 1)

      if [ -z "$LATEST_TAG" ]; then
        VERSION="1.0.0"
      else
        VERSION=$(echo "$LATEST_TAG" | sed 's/^v//')

        MAJOR=$(echo "$VERSION" | cut -d. -f1)
        MINOR=$(echo "$VERSION" | cut -d. -f2)
        PATCH=$(echo "$VERSION" | cut -d. -f3)

        PATCH=$((PATCH + 1))

        VERSION="$MAJOR.$MINOR.$PATCH"
      fi

      echo "VERSION=$VERSION" > version.env
      echo "Building version: $VERSION"

    - source version.env

    # Build versioned image
    - docker build -t "$DOCKER_IMAGE:$VERSION" .

    # Also tag as latest
    - docker tag "$DOCKER_IMAGE:$VERSION" "$DOCKER_IMAGE:latest"

  artifacts:
    reports:
      dotenv: version.env


deploy:
  stage: deploy

  script:
    # Login to IONOS Container Registry
    - echo "$IONOS_REGISTRY_PASSWORD" | docker login "$DOCKER_REGISTRY" -u "$IONOS_REGISTRY_USERNAME" --password-stdin

    # Push versioned image
    - docker push "$DOCKER_IMAGE:$VERSION"

    # Push latest
    - docker push "$DOCKER_IMAGE:latest"

  only:
    - main
