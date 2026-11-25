> **Obsolete:** This repository is obsolete and no longer actively maintained.

# A2A Go FMSG Transport

[![Go Reference](https://pkg.go.dev/badge/github.com/markmnl/a2a-go-fmsg.svg)](https://pkg.go.dev/github.com/markmnl/a2a-go-fmsg)
[![test](https://github.com/markmnl/a2a-go-fmsg/actions/workflows/test.yml/badge.svg)](https://github.com/markmnl/a2a-go-fmsg/actions/workflows/test.yml)

`a2afmsg` is an FMSG transport plugin for the official
[A2A Go SDK](https://github.com/a2aproject/a2a-go). It implements
[FMSG-004 v0.2.0](https://github.com/markmnl/fmsg/blob/main/standards/fmsg-004-a2a-binding.md)
over an [FMSG-003 Web API](https://github.com/markmnl/fmsg/blob/main/standards/fmsg-003-webapi.md).

The binding is currently experimental. Its protocol identifier must match the
`protocolBinding` value in the remote Agent Card exactly:

```text
https://github.com/markmnl/fmsg/blob/main/standards/fmsg-004-a2a-binding.md
```

## Install

```sh
go get github.com/markmnl/a2a-go-fmsg@v0.1.0
```

The module requires Go 1.25 or newer and A2A Go SDK v2.

## Use

```go
package main

import (
    "context"
    "os"

    "github.com/a2aproject/a2a-go/v2/a2a"
    "github.com/a2aproject/a2a-go/v2/a2aclient"
    "github.com/a2aproject/a2a-go/v2/a2aclient/agentcard"
    a2afmsg "github.com/markmnl/a2a-go-fmsg"
)

func main() {
    ctx := context.Background()
    card, err := agentcard.DefaultResolver.Resolve(ctx, "https://fmsg.io")
    if err != nil {
        panic(err)
    }

    client, err := a2aclient.NewFromCard(ctx, card,
        a2afmsg.WithTransport(a2afmsg.Config{
            WebAPIURL: os.Getenv("FMSG_API_URL"),
            APIKey:    os.Getenv("FMSG_API_KEY"),
        }),
    )
    if err != nil {
        panic(err)
    }
    defer client.Destroy()

    _, err = client.SendMessage(ctx, &a2a.SendMessageRequest{
        Message: a2a.NewMessage(a2a.MessageRoleUser, a2a.NewTextPart("Hello Groot")),
    })
    if err != nil {
        panic(err)
    }
}
```

`WebAPIURL` and `APIKey` belong to the calling FMSG address, not the remote
agent. The transport authenticates through `POST /fmsg/token` and uses the
token subject as the caller's verified FMSG address.

The plugin supports every A2A 1.0 client transport operation, native FMSG
mapping for `Part.raw`, request/result correlation, task and context thread
topology, detached recovery after FMSG response code 6, streaming event
ordering, bounded resource limits, and distinct A2A and FMSG delivery errors.

## End-to-end CLI

```sh
go install github.com/markmnl/a2a-go-fmsg/cmd/a2a-fmsg@v0.1.0
export FMSG_API_URL=https://api.example.com
export FMSG_API_KEY=fmsgk_callers_key
a2a-fmsg -card https://fmsg.io "Hello Groot"
```

Using a caller on a different FMSG domain also exercises federated delivery.

## Credentials and retention

FMSG messages are immutable and may be retained. Values supplied through
`Config.Credentials` therefore become durable message content. Never send
passwords, refresh tokens, or long-lived API keys; use narrowly scoped,
short-lived credentials where application-level authentication is required.

## Versioning

Module releases and binding revisions are versioned independently. Module
`v0.1.x` implements FMSG-004 binding version `0.2` and A2A protocol version
`1.0` using `github.com/a2aproject/a2a-go/v2`.

## License

Apache License 2.0.
