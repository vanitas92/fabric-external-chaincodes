FROM alpine:3.13 as base

# Add CA certificates and timezone data files
RUN apk add -U --no-cache ca-certificates tzdata

# Add unprivileged user
# Since the proxy needs to bind to a port, only root user has enough permissions to do that, 
# otherwise applications not requiring this privilege should use a non admin user
RUN adduser -s /bin/true -u 1000 -D -h /app app \
    && sed -i -r "/^(app|root)/!d" /etc/group /etc/passwd \
    && sed -i -r 's#^(.*):[^:]*$#\1:/sbin/nologin#' /etc/passwd

# This image is a microservice in golang for the marbles chaincode
FROM golang:1.15.8-alpine AS build

COPY ./ /go/src/github.com/marbles
WORKDIR /go/src/github.com/marbles

# Build application
RUN CGO_ENABLED=0 GOARCH=amd64 GOOS=linux go build -trimpath -ldflags '-extldflags "-static" -w -s' -o chaincode -v .

# Production ready image
# Pass the binary to the prod image
FROM scratch

# Add the timezone data files
COPY --from=base /usr/share/zoneinfo /usr/share/zoneinfo

# Add the CA certificates
COPY --from=base /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Add-in our unprivileged user
# As explained above, the user needs admin right to bind the 53 port
COPY --from=base /etc/passwd /etc/group /etc/shadow /etc/

COPY --from=build /go/src/github.com/marbles/chaincode /chaincode

USER app

CMD /chaincode