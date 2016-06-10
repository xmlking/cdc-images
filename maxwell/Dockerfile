FROM java:8-jre
ENV MAXWELL_VERSION 1.1.1

RUN mkdir /app
WORKDIR /app

RUN curl -sLo - https://github.com/zendesk/maxwell/releases/download/v"$MAXWELL_VERSION"/maxwell-"$MAXWELL_VERSION".tar.gz \
  | tar --strip-components=1 -zxvf -

ADD config.properties /app

RUN echo "$MAXWELL_VERSION" > /REVISION
CMD bin/maxwell
