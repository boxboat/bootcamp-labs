FROM golang:1.8

WORKDIR /go/src/myapp
COPY app.go .

RUN go build -o app

CMD [“./app”]


