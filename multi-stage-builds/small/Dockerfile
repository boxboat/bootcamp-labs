FROM golang:1.8

WORKDIR /go/src/myapp
COPY app.go .

RUN go build -o app .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/

COPY --from=0 /go/src/myapp/app .

CMD [“./app”]
