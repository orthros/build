# Copyright 2017 The Go Authors. All rights reserved.
# Use of this source code is governed by a BSD-style
# license that can be found in the LICENSE file.

FROM golang:1.12 AS build
LABEL maintainer "golang-dev@googlegroups.com"

ENV GO111MODULE=on
ENV GOPROXY=https://proxy.golang.org

RUN mkdir /gocache
ENV GOCACHE /gocache

COPY go.mod /go/src/golang.org/x/build/go.mod
COPY go.sum /go/src/golang.org/x/build/go.sum

WORKDIR /go/src/golang.org/x/build

# Optimization for iterative docker build speed, not necessary for correctness:
# TODO: write a tool to make writing Go module-friendly Dockerfiles easier.
RUN go install cloud.google.com/go/compute/metadata
COPY gerrit /go/src/golang.org/x/build/gerrit
RUN go install golang.org/x/build/gerrit
COPY buildenv /go/src/golang.org/x/build/buildenv
RUN go install golang.org/x/build/buildenv
COPY internal/foreach /go/src/golang.org/x/build/internal/foreach
COPY maintner /go/src/golang.org/x/build/maintner
COPY cmd/pubsubhelper /go/src/golang.org/x/build/cmd/pubsubhelper
RUN go install golang.org/x/build/maintner/maintnerd/apipb
RUN go install golang.org/x/build/maintner/godata

COPY . /go/src/golang.org/x/build/

RUN go install golang.org/x/build/cmd/gitmirror

FROM debian:buster
LABEL maintainer "golang-dev@googlegroups.com"

# For interacting with the Go source & subrepos
RUN apt-get update && apt-get install -y \
	--no-install-recommends \
	ca-certificates \
	git-core \
	openssh-client \
	gnupg dirmngr \
	&& rm -rf /var/lib/apt/lists/*

# See https://github.com/golang/go/issues/23705
ENV TINI_VERSION v0.16.1
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini.asc /tini.asc
RUN gpg --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 595E85A6B1B4779EA4DAAEC70B588DFF0527A9B7 \
 && gpg --verify /tini.asc
RUN chmod +x /tini

# Add github.com's known_hosts entries, so git push calls later don't
# prompt, and don't need to have their strict host key checking
# disabled.
RUN mkdir -p ~/.ssh/ \
	&& chmod 0700 ~/.ssh/ \
	&& echo "|1|SFEvEAqYsJ18JCr+0iV4GtlwS4w=|P6oCZUUd/5t9pH4Om7ShlfltRyE= ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAq2A7hRGmdnm9tUDbO9IDSwBK6TbQa+PXYPCPy6rbTrTtw7PHkccKrpp0yVhp5HdEIcKr6pLlVDBfOLX9QUsyCOV0wzfjIJNlGEYsdlLJizHhbn2mUjvSAHQqZETYP81eFzLQNnPHt4EVVUh7VfDESU84KezmD5QlWpXLmvU31/yMf+Se8xhHTvKSCZIFImWwoG6mbUoWf9nzpIoaSjB+weqqUUmpaaasXVal72J+UX2B+2RPW3RcT0eOzQgqlJL3RKrTJvdsjE3JEAvGq3lGHSZXy28G3skua2SmVi/w4yCE6gbODqnTWlg7+wC604ydGXA8VJiS5ap43JXiUFFAaQ==" > ~/.ssh/known_hosts \
	&& echo "|1|HygGkfOGLovavKfixjXWFJ7Yk1I=|lb/724row8KDTMC1dZiJlHyjxWM= ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAq2A7hRGmdnm9tUDbO9IDSwBK6TbQa+PXYPCPy6rbTrTtw7PHkccKrpp0yVhp5HdEIcKr6pLlVDBfOLX9QUsyCOV0wzfjIJNlGEYsdlLJizHhbn2mUjvSAHQqZETYP81eFzLQNnPHt4EVVUh7VfDESU84KezmD5QlWpXLmvU31/yMf+Se8xhHTvKSCZIFImWwoG6mbUoWf9nzpIoaSjB+weqqUUmpaaasXVal72J+UX2B+2RPW3RcT0eOzQgqlJL3RKrTJvdsjE3JEAvGq3lGHSZXy28G3skua2SmVi/w4yCE6gbODqnTWlg7+wC604ydGXA8VJiS5ap43JXiUFFAaQ==" >> ~/.ssh/known_hosts \
	&& chmod 0600 ~/.ssh/known_hosts

COPY --from=build /go/bin/gitmirror /
ENTRYPOINT ["/tini", "--", "/gitmirror"]
