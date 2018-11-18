# build stage
FROM golang:alpine AS builder
ADD . /src
RUN cd /src && go build -o goapp

# production stage
FROM alpine
WORKDIR /app
COPY --from=builder /src/goapp /app/
ENTRYPOINT ./goapp
