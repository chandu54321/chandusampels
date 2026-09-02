pip install "paddleocr==2.7.3" "paddlex==2.1.0" "paddlepaddle==2.6.2"



   pip uninstall paddlepaddle
   git clone https://github.com/PaddlePaddle/Paddle.git && cd Paddle
   git checkout release/3.2  # Match your required version
   mkdir build && cd build
   cmake .. -DWITH_GPU=OFF -DWITH_AVX=OFF -DWITH_TESTING=OFF -DCMAKE_BUILD_TYPE=Release -DPY_VERSION=3.10
   make -j$(nproc)
   pip install python/dist/paddlepaddle-*.whl
