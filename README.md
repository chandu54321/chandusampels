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
