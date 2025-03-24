ARG debian_buster_image_tag=11-jre-slim
FROM openjdk:${debian_buster_image_tag}

ARG scala_version="2.13.16"
ARG build_date
ARG shared_workspace=/opt/workspace

RUN mkdir -p ${shared_workspace}/data
RUN mkdir -p /usr/share/man/man1
RUN apt-get update -y
RUN apt-get install -y curl python3 procps r-base
RUN ln -s /usr/bin/python3 /usr/bin/python
RUN curl https://downloads.lightbend.com/scala/2.13.16/scala-${scala_version}.deb -k -o scala.deb
RUN apt install -y ./scala.deb
RUN rm -rf scala.deb /var/lib/apt/lists/*

ENV SCALA_HOME="/usr/bin/scala"
ENV PATH=${PATH}:${SCALA_HOME}/bin
ENV SHARED_WORKSPACE=${shared_workspace}

VOLUME ${shared_workspace}
CMD ["bash"]

ARG build_date
ARG spark_version="3.5.5"
ARG scala_version="2.13.16"
ARG jupyterlab_version="4.3.0"
ARG scala_kernel_version="0.14.1"

#JupyterLab + Python kernel for PySpark
COPY workspace/ /workspace/

RUN apt-get update -y
RUN apt-get install -y python3-pip python3-dev
RUN pip3 install --upgrade pip
RUN pip3 install wget==3.2 pyspark==${spark_version} jupyterlab==${jupyterlab_version}
RUN pip3 install catboost numpy pandas lightgbm matplotlib optuna pycaret plotly scipy
RUN pip3 install scikit-learn statsmodels nltk

#Scala kernel for Spark
RUN apt-get install -y ca-certificates-java --no-install-recommends
RUN curl -Lo coursier https://git.io/coursier-cli
RUN chmod +x coursier
RUN ./coursier launch --fork almond:${scala_kernel_version} --scala ${scala_version} -- --display-name "Scala ${scala_version}" --install
RUN rm -f coursier

#R kernel for SparkR
RUN apt-get install -y r-base-dev
RUN R -e "install.packages('IRkernel')"
RUN R -e "IRkernel::installspec(displayname = 'R 3.5', user = FALSE)"
RUN curl https://archive.apache.org/dist/spark/spark-${spark_version}/SparkR_${spark_version}.tar.gz -k -o sparkr.tar.gz
RUN R CMD INSTALL sparkr.tar.gz
RUN rm -f sparkr.tar.gz

EXPOSE 8888

WORKDIR /workspace
CMD jupyter lab --ip=0.0.0.0 --port=8888 --no-browser --allow-root --NotebookApp.token=
