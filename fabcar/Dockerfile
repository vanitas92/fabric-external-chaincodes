# This image is a microservice in golang for the Degree chaincode
FROM golang:1.14.2-alpine AS build

COPY ./ /go/src/github.com/fabcar
WORKDIR /go/src/github.com/fabcar

# Build application
RUN go build -o chaincode -v .

# Production ready image
# Pass the binary to the prod image
FROM alpine:3.11 as prod

COPY --from=build /go/src/github.com/fabcar/chaincode /app/chaincode

USER 1000

WORKDIR /app
CMD ./chaincode