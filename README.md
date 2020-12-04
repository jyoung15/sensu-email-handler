[![Bonsai Asset Badge](https://img.shields.io/badge/Sensu%20Go%20Email%20Handler-Download%20Me-brightgreen.svg?colorB=89C967&logo=sensu)](https://bonsai.sensu.io/assets/sensu/sensu-email-handler)

# Sensu Email Handler
- [Overview](#overview)
- [Usage examples](#usage-examples)
- [Configuration](#configuration)
  - [Asset registration](#asset-registration)
  - [Asset definition](#asset-definition)
  - [Handler definition](#handler-definition)
- [Annotations](#annotations)
- [Templates](#templates)
  - [Formatting Timestamps in Templates](#formatting-timestamps-in-templates)
  - [Formatting the Event ID in Templates](#formatting-the-event-id-in-templates)
- [Installing from source and contributing](#installing-from-source-and-contributing)

## Overview

The Sensu Go Email Handler is a [Sensu Event Handler][2] for sending
incident notification emails.

## Usage Examples

### Help

```
The Sensu Email handler sends an email notifications

Usage:
  sensu-email-handler [flags]

Flags:
  -a, --authMethod string         The SMTP authentication method, one of 'none', 'plain', or 'login' (default "plain")
  -T, --bodyTemplateFile string   A template file to use for the body
  -l, --enableLoginAuth           [deprecated] Use "login auth" mechanisim
  -f, --fromEmail string          The 'from' email address
  -h, --help                      help for sensu-email-handler
  -H, --hookout                   Include output from check hook(s)
  -i, --insecure                  [deprecated] Use an insecure connection (unauthenticated on port 25)
  -s, --smtpHost string           The SMTP host to use to send to send email
  -p, --smtpPassword string       The SMTP password, if not in env SMTP_PASSWORD
  -P, --smtpPort uint             The SMTP server port (default 587)
  -u, --smtpUsername string       The SMTP username, if not in env SMTP_USERNAME
  -S, --subjectTemplate string    A template to use for the subject (default "Sensu Alert - {{.Entity.Name}}/{{.Check.Name}}: {{.Check.State}}")
  -k, --tlsSkipVerify             Do not verify TLS certificates
  -t, --toEmail string            The 'to' email address (accepts comma delimited and/or multiple flags)
```
## Configuration

### Asset registration

Assets are the best way to make use of this handler. If you're not using an asset, please consider doing so! If you're using sensuctl 5.13 or later, you can use the following command to add the asset:

`sensuctl asset add sensu/sensu-email-handler`

If you're using an earlier version of sensuctl, you can download the asset definition from [this project's Bonsai Asset Index page](https://bonsai.sensu.io/assets/sensu/sensu-email-handler).

### Asset definition

```yml
---
type: Asset
api_version: core/v2
metadata:
  name: sensu-email-handler_linux_amd64
  labels:
  annotations:
    io.sensu.bonsai.url: https://bonsai.sensu.io/assets/sensu/sensu-email-handler
    io.sensu.bonsai.api_url: https://bonsai.sensu.io/api/v1/assets/sensu/sensu-email-handler
    io.sensu.bonsai.tier: Supported
    io.sensu.bonsai.version: 0.2.0
    io.sensu.bonsai.namespace: sensu
    io.sensu.bonsai.name: sensu-email-handler
    io.sensu.bonsai.tags: handler
spec:
  url: https://assets.bonsai.sensu.io/45eaac0851501a19475a94016a4f8f9688a280f6/sensu-email-handler_0.2.0_linux_amd64.tar.gz
  sha512: d69df76612b74acd64aef8eed2ae10d985f6073f9b014c8115b7896ed86786128c20249fd370f30672bf9a11b041a99adb05e3a23342d3ad80d0c346ec23a946
  filters:
  - entity.system.os == 'linux'
  - entity.system.arch == 'amd64'
```

### Handler definition

```yml
---
api_version: core/v2
type: Handler
metadata:
  namespace: default
  name: email
spec:
  type: pipe
  command: sensu-email-handler -f from@example.com -t to@example.com -t "to2@example.com, to3@example.com" -s smtp.example.com
    -u user -p password
  timeout: 10
  filters:
  - is_incident
  - not_silenced
  runtime_assets:
  - sensu/sensu-email-handler
```

### Annotations
All of the above command line arguments can be overridden by check or entity annotations.
The annotation consists of the key formed by appending the "long" argument specification
to the string sensu.io/plugins/email/config (e.g. sensu.io/plugins/email/config/toEmail).

For example, having the following in an agent.yml file will create an entity annotation
such that emails generated by events on this entity will go to `ops@example.com` instead
of the recipient defined in the handler.  And the subject would be something to the effect
of 'failing - webserver01/check-nginx'.

```
namespace: "default"
subscriptions:
  - linux
backend-url:
  - "ws://127.0.0.1:8081"
annotations:
  sensu.io/plugins/email/config/toEmail: "ops@example.com"
  sensu.io/plugins/email/config/subjectTemplate: "{{.Check.State}} - {{.Entity.Name}}/{{.Check.Name}}",
```

**Note:** When using template expansion for annotations in **checks**, the token delimiters ``{{`` and ``}}`` have to be escaped with ``{{` `` and `` `}}``, respectively.  Yes, this is ugly, but it is because checks are ran through token expansion before being ran.  This is not needed for annotations in entities.

Example:
```
"sensu.io/plugins/email/config/subjectTemplate": "{{`{{ .Check.State }}`}} - {{`{{ .Entity.Name }}`}}/{{`{{ .Check.Name }}`}}",

```

### Templates

The plugin provides an option to use a template file for the body of the email and is capable of using HTML for formatting the email. This template file would need to be available on all backends on which this handler may run. An example is provided below:

```
/etc/sensu/email_template

<html>
Greetings,

<h3>Informational Details</h3>
<b>Check</b>: {{ .Check.Name }}<br>
<b>Entity</b>: {{ .Entity.Name }}<br>
<b>State</b>: {{ .Check.State }}<br>
<b>Occurrences</b>: {{ .Check.Occurrences }}<br>
<b>Playbook</b>: https://example.com/monitoring/wiki/playbook
<h3>Check Output Details</h3>
<b>Check Output</b>: {{.Check.Output}}
<h4>Check Hook(s)</h4>
{{range .Check.Hooks}}<b>Hook Name</b>:  {{.Name}}<br>
<b>Hook Command</b>:  {{.Command}}<br>
<b>Hook Output</b>: {{.Output}}<br>
{{end}}<br>
<br>
<br>
#monitoringlove,<br>
<br>
Sensu<br>
</html>
```

Note that this uses tokens to populate the values provided by the event.  More information on template syntax and format can be found in [the documentation][6]

Also note that line breaks in your template and any text surfaced by token substitution are replaced with the HTML &lt;br&gt; tag.

At the time of this example, check hooks and templates are not able to be used together via the `-H` and `-T` flags. However, you may include the hook output as part of the template via the following:

```
{{range .Check.Hooks}}
Hook Name:  {{.Name}}
Hook Command:  {{.Command}}
{{.Output}}
```
#### Formatting Timestamps in Templates

A Sensu Go event contains multiple timestamps (e.g. .Check.Issued,
.Check.Executed, .Check.LastOk) that are presented in [UNIX timestamp][3]
format.  A function named UnixTime is provided to print these values in a
customizable human readable format as part of a template.  To customize the
output format of the timestamp, use the same format as specified by Golang's
[Time.Format][4].  Additional examples can be found [here][5].

**Note:** the predefined format constants are **not** available.

The example below embellishes the prior example template to include two
timestamps.

```
[...]
<h3>Check Output Details</h3>
<b>Executed</b>: {{(UnixTime .Check.Executed).Format "2 Jan 2006 15:04:05"}}<br>
<b>Last OK</b>: {{(UnixTime .Check.LastOK).Format "2 Jan 2006 15:04:05"}}<br>
<b>Check Output</b>: {{.Check.Output}}
[...]
```

#### Formatting the Event ID in Templates

Each Sensu Go event contains a unique event ID (UUID) that is presented to
this handler in a non-printable format.  To properly print this value as
part of a template, the function UUIDFromBytes is provided.  The example
below shows its use:

```
[...]
<b>Event ID</b>: {{UUIDFromBytes .ID}}
<h3>Check Output Details</h3>
<b>Check Output</b>: {{.Check.Output}}
```

## Installing from source and contributing

Download the latest version of the sensu-email-handler from [releases][1],
or create an executable from this source.

From the local path of the sensu-email-handler repository:

```
go build -o /usr/local/bin/sensu-email-handler main.go
```
For additional instructions, see [CONTRIBUTING](https://github.com/sensu/sensu-go/blob/master/CONTRIBUTING.md)

[1]: https://github.com/sensu/sensu-email-handler/releases
[2]: https://docs.sensu.io/sensu-go/latest/reference/handlers/#how-do-sensu-handlers-work
[3]: https://en.wikipedia.org/wiki/Unix_time
[4]: https://golang.org/pkg/time/#Time.Format
[5]: https://yourbasic.org/golang/format-parse-string-time-date-example/
[6]: https://docs.sensu.io/sensu-go/latest/observability-pipeline/observe-process/handler-templates/
