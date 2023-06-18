FROM golang:1.20-alpine AS build

COPY ./ /go/src/github.com/marbles
WORKDIR /go/src/github.com/marbles

# Build application
RUN CGO_ENABLED=0 GOARCH=amd64 GOOS=linux go build -trimpath -ldflags '-extldflags "-static" -w -s' -o chaincode -v .

# Production ready image
# Pass the binary to the prod image
FROM gcr.io/distroless/static

USER nonroot:nonroot

COPY --from=build --chown=nonroot:nonroot /go/src/github.com/marbles/chaincode /chaincode

ENTRYPOINT ["/chaincode"]