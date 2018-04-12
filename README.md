FROM centos

ARG MICROSERVICE=${MICROSERVICE}
ENV MICROSERVICE ${MICROSERVICE}

RUN yum install java-1.8.0-openjdk-devel -y
# Download and Install scala and sbt
RUN yum install -y wget unzip
RUN wget http://downloads.typesafe.com/scala/2.12.3/scala-2.12.3.tgz -O //scala-2.12.3.tgz -q
RUN cd / & tar -xzf scala-2.12.3.tgz
RUN echo "export SCALA_HOME=/scala-2.12.3" >> /etc/profile
RUN echo "export PATH=$PATH:/scala-2.12.3/bin" >> /etc/profile
ENV SCALA_HOME /scala-2.12.3/
ENV PATH "$PATH:/scala-2.12.3/bin"

## Install aws cli to download files from s3bucket
RUN curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py" && python get-pip.py &&  pip install awscli --ignore-installed six

RUN mkdir ~/.aws/ && echo -e '[default] \naws_access_key_id = AKIAIAH6YLN5GRZO6FVQ \naws_secret_access_key = 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' >  ~/.aws/credentials && echo '[default] \nregion = us-west-2' > ~/.aws/config && chmod 600 ~/.aws/credentials ~/.aws/config
WORKDIR /opt/
RUN aws s3 cp s3://lifexartifactbucket/${MICROSERVICE}//${MICROSERVICE}.tar /opt/
RUN tar -xvf ${MICROSERVICE}.tar -C /opt/ && mv /opt/pack /opt/${MICROSERVICE}
RUN aws s3 cp s3://lifexartifactbucket/moola.conf /opt/${MICROSERVICE}/

RUN sed -i 's/localhost:24041/mongo:24041/g' /opt/${MICROSERVICE}/moola.conf
WORKDIR /opt/
RUN yum install -y which
CMD /opt/${MICROSERVICE}/bin/${MICROSERVICE}main -Dconfig.file=/opt/${MICROSERVICE}/moola.conf
