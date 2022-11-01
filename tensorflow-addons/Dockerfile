ARG UBUNTU_VERSION=latest
FROM ubuntu:${UBUNTU_VERSION} AS tfa-build
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
RUN apt-get -y install git rsync python-is-python3
RUN git clone https://github.com/tensorflow/addons.git
WORKDIR /addons
ARG TF_ADDONS_BRANCH=r0.18
RUN git checkout ${TF_ADDONS_BRANCH}
RUN python ./configure.py
RUN bazel build build_pip_pkg
RUN bazel-bin/build_pip_pkg artifacts

FROM ubuntu:${UBUNTU_VERSION} AS tfa-install
ARG DEBIAN_FRONTEND=noninteractive
COPY --from=tfa-build /addons/artifacts/*.whl /wheels/
RUN pip install /wheels/tensorflow_addons-*.whl

RUN python3 -c "import tensorflow_addons as tfa"

CMD ["python3"]