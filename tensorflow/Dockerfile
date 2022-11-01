ARG UBUNTU_VERSION=latest
FROM ubuntu:${UBUNTU_VERSION} AS tf-build
RUN apt-get update
RUN apt-get install -y python3-dev python3-pip
RUN pip install -U pip numpy wheel packaging requests opt_einsum
RUN pip install -U keras_preprocessing --no-deps

RUN apt-get install -y apt-transport-https curl gnupg
RUN curl -fsSL https://bazel.build/bazel-release.pub.gpg | gpg --dearmor > bazel-archive-keyring.gpg
RUN mv bazel-archive-keyring.gpg /usr/share/keyrings
RUN echo "deb [arch=amd64 signed-by=/usr/share/keyrings/bazel-archive-keyring.gpg] https://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list
ARG BAZEL_VERSION=5.3.0
RUN apt-get update && apt-get install -y bazel-${BAZEL_VERSION}
RUN ln -s /usr/bin/bazel-${BAZEL_VERSION} /usr/bin/bazel

WORKDIR /
RUN apt-get -y install git

RUN git clone https://github.com/tensorflow/tensorflow.git
WORKDIR /tensorflow
ARG TENSORFLOW_BRANCH=r2.11
RUN git checkout ${TENSORFLOW_BRANCH}

# CPU support configuration
RUN ./configure

# CPU only support
RUN bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package

# GPU support
#RUN bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package

# Build pip whl package
RUN ./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg

FROM ubuntu:${UBUNTU_VERSION} AS tf-install
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update
RUN apt-get install -y python3 python3-pip
COPY --from=tf-build /tmp/tensorflow_pkg/*.whl wheels/
RUN pip install /wheels/tensorflow-*.whl

RUN python3 -c "import tensorflow as tf;print(tf.reduce_sum(tf.random.normal([1000, 1000])))"

CMD ["python3"]