FROM golang:1.21-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./

RUN go mod download

COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -o /boz ./cmd/boz

FROM alpine:latest  

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY --from=builder /boz .

COPY config ./config

COPY db/migration ./db/migration

COPY .env .

EXPOSE 8085

CMD ["./boz"]