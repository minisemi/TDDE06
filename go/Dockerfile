# Our base image
FROM alpine:3.5

RUN apk add --no-cache ca-certificates

# Define Golang language
ENV GOLANG_VERSION 1.8rc3
ENV GOLANG_SRC_URL https://golang.org/dl/go$GOLANG_VERSION.src.tar.gz
ENV GOLANG_SRC_SHA256 38b1c1738f111f7bccdd372efca2aa98a7bad1ca2cb21767ba69f34ae007499c

# https://golang.org/issue/14851
COPY no-pic.patch /

# Download and install Golang in the machine
RUN set -ex \
	&& apk add --no-cache --virtual .build-deps \
		bash \
		gcc \
		musl-dev \
		openssl \
		go \
	\
	&& export GOROOT_BOOTSTRAP="$(go env GOROOT)" \
	\
	&& wget -q "$GOLANG_SRC_URL" -O golang.tar.gz \
	&& echo "$GOLANG_SRC_SHA256  golang.tar.gz" | sha256sum -c - \
	&& tar -C /usr/local -xzf golang.tar.gz \
	&& rm golang.tar.gz \
	&& cd /usr/local/go/src \
	&& patch -p2 -i /no-pic.patch \
	&& ./make.bash \
	\
	&& rm -rf /*.patch \
	&& apk del .build-deps

# Set environment variables
ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH
ENV GOBIN $GOPATH/bin

# Create directories in our /go folder
RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"
WORKDIR $GOPATH

COPY go-wrapper /usr/local/bin/

# Install git in order to run "go get"
RUN apk update && \
    apk upgrade && \
    apk add git

# Copy our files to the image
COPY todo.go src/
COPY todo_test.go src/
COPY schema.sql src/

WORKDIR $GOPATH/src

RUN go get
RUN go build 
EXPOSE 8080
CMD go run todo.go