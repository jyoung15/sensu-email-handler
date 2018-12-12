# Sensu Go Email Handler Plugin
TravisCI: [![TravisCI Build Status](https://travis-ci.org/sensu/sensu-email-handler.svg?branch=master)](https://travis-ci.org/sensu/sensu-email-handler)

The Sensu Go Email Handler is a [Sensu Event Handler][2] for sending
incident notification emails.

## Installation

Download the latest version of the sensu-email-handler from [releases][1],
or create an executable script from this source.

From the local path of the sensu-email-handler repository:

```
go build -o /usr/local/bin/sensu-email-handler main.go
```

## Configuration

Example Sensu Go definition:

```json
{
    "api_version": "core/v2",
    "type": "Handler",
    "metadata": {
        "namespace": "default",
        "name": "email"
    },
    "spec": {
        "type": "pipe",
        "command": "sensu-email-handler -h",
        "timeout": 10,
        "filters": [
            "is_incident",
            "not_silenced"
        ]
    }
}
```

## Usage Examples

Help:

```
The Sensu Go Email handler for sending an email notification

Usage:
  sensu-email-handler [flags]

Flags:
  -f, --foo string   example
  -h, --help         help for sensu-email-handler
```

## Contributing

See https://github.com/sensu/sensu-go/blob/master/CONTRIBUTING.md

[1]: https://github.com/sensu/sensu-email-handler/releases
[2]: https://docs.sensu.io/sensu-go/5.0/reference/handlers/#how-do-sensu-handlers-work
