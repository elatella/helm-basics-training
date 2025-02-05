---
title: "Installation for Windows"
weight: 3
type: docs
sectionnumber: 1
---

## Installation for Windows

Install the `helm` CLI binary on your system:

1. [Download the latest release](https://github.com/helm/helm/releases)
1. Unzip it
1. Find the `helm` binary in the unpacked directory and move it to its desired destination
    * The desired destination should be listed in your $PATH environment variable (`echo $PATH`)

{{% alert title="Note" color="primary" %}}
Windows quick hack: Copy the `helm` binary directly into the folder `C:\Windows`.
{{% /alert %}}

If the `$PATH` variable doesn't contain a suitable directory, it can be changed in the advanced system settings:

* [How to set the path and environment variables in Windows](https://www.computerhope.com/issues/ch000549.htm)

{{% onlyWhen mobi %}}


## Proxy configuration

{{% alert title="Note" color="primary" %}}
If you have direct access to the internet from your location, the proxy configuration is not required.
{{% /alert %}}

Set your HTTP proxy environment variables so that a chart repository can be added to your Helm repos in a later lab. It is recommended to set the lowercase and uppercase variables, as the helm command takes them all into account.

In Windows cmd:

```bash
setx HTTP_PROXY="http://<username>:<password>@<proxy>:<port>"
setx HTTPS_PROXY="http://<username>:<password>@<proxy>:<port>"
setx NO_PROXY="<noproxy-list>"
setx http_proxy="http://<username>:<password>@<proxy>:<port>"
setx https_proxy="http://<username>:<password>@<proxy>:<port>"
setx no_proxy="<noproxy-list>"
```

In Windows Powershell:

```bash
$env:HTTP_PROXY="http://<username>:<password>@<proxy>:<port>"
$env:HTTPS_PROXY="http://<username>:<password>@<proxy>:<port>"
$env:NO_PROXY="<noproxy-list>"
$env:http_proxy="http://<username>:<password>@<proxy>:<port>"
$env:https_proxy="http://<username>:<password>@<proxy>:<port>"
$env:no_proxy="<noproxy-list>"
```

In Git Bash:

```bash
export HTTP_PROXY="http://<username>:<password>@<proxy>:<port>"
export HTTPS_PROXY="http://<username>:<password>@<proxy>:<port>"
export NO_PROXY="<noproxy-list>"
export http_proxy="http://<username>:<password>@<proxy>:<port>"
export https_proxy="http://<username>:<password>@<proxy>:<port>"
export no_proxy="<noproxy-list>"
```

Replace `<username`> and `<password>` with your credentials. If you have special characters in your password, escape them with their corresponding hexadecimal values according to [this article](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters).
{{% /onlyWhen %}}


## Verification

Now, [verify your installation](../04/).
