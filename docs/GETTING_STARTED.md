## Getting Started

In this document we aim to guide users who are new to this project through the
initial setup and demonstrate a couple use cases, helping to get up and running
quickly.

### Installation

MacOS users can install the `newrelic` CLI using [Homebrew][homebrew].

    brew install newrelic-cli

Docker users can grab the latest [Docker image][docker_image] containing the
`newrelic` CLI from docker.

    docker pull newrelic/cli:latest
    docker run -i --rm -e NEW_RELIC_API_KEY newrelic/cli apm application get --name WebPortal --accountId $ACCOUNT_ID

Binary releases are available on [GitHub][releases] as well and installed
anywhere in the environment `$PATH` for the user.

### Environment Setup

You will need a [Personal API Key][api_key] in order to make full use of this
project.  Once the API key has been created, you can export it into the
environment.

    export NEW_RELIC_API_KEY=<your_personal_api_key>

API keys are specific to region, and this project defaults to "US".  If you're
wanting to use this against the EU region, you'll need the following
environment variable.

    export NEW_RELIC_REGION="EU"

### Shell Completion

Frequent users of the shell might appreciate a little assistance from their
shell to help complete the commands.  To install shell completion, the
`completion` command is used, along with a `--shell` flag to note which shell
completions to generate.

You can store the functions yourself and periodically update them as we do
here, or you can do as suggested by running the `newrelic help completion`
command.

    newrelic completion --shell zsh > /usr/local/share/zsh/functions/_newrelic

The above is ZSH specific, but provides an example of how to store the files on
disk rather than executing the command each time you start a shell.

### Example Use cases

In the examples that follow, we'll be using an example application,
`WebPortal`.

To make the examples simpler, we'll export the environment variable
`$ACCOUNT_ID`.

    export ACCOUNT_ID=<your_account_id>

#### Tagging an application

In order to know which application we want to tag, we'll perform an APM search
using the `apm application get` command.

    newrelic apm application get --accountId $ACCOUNT_ID --name WebPortal

This yields the following output about the application we're going to target.

    [
      {
        "accountId": 2508259,
        "applicationId": 204261368,
        "domain": "APM",
        "entityType": "APM_APPLICATION_ENTITY",
        "guid": "MjUwODI1OXxBUE18QVBQTElDQVRJT058MjA0MjYxMzY4",
        "name": "WebPortal",
        "permalink": "https://one.newrelic.com/redirect/entity/MjUwODI1OXxBUE18QVBQTElDQVRJT058MjA0MjYxMzY4",
        "reporting": true,
        "type": "APPLICATION"
      }
    ]

From here, we can see the GUID, which uniquely identifies this specific
application.  We'll use that GUID below using the `entities create-tags`
command.

    newrelic entities create-tags -g MjUwODI1OXxBUE18QVBQTElDQVRJT058MjA0MjYxMzY4 -t devkit:testing

The tags themselves are colon separated key:value pairs, while each tag set is
comma separated.  As such, to set multiple tags at the same time, you could do
something like the following.

    newrelic entities create-tags -g MjUwODI1OXxBUE18QVBQTElDQVRJT058MjA0MjYxMzY4 -t env:test,compliant:true

Now we can query for the tags on the given application.

    newrelic entities describe-tags -g MjUwODI1OXxBUE18QVBQTElDQVRJT058MjA0MjYxMzY4

Which gives the following output.

    [
      ...
      {
        "Key": "compliant",
        "Values": [
          "true"
        ]
      },
      {
        "Key": "env",
        "Values": [
          "test"
        ]
      },
      ...
      {
        "Key": "devkit",
        "Values": [
          "testing"
        ]
      }
    ]

#### Creating a deployment marker

To create a deployment, we use the same ApplicationID from our earlier search,
and pass that to the `apm create-deployment` command.

    newrelic apm create-deployment -a 204261368 -r $(git rev-parse HEAD)

With this we get a response back that looks like the following.

    {
      "id": 37075986,
      "links": {
        "application": 204261368
      },
      "revision": "40af0d07d86363d289e772f4326d23d1717fb765",
      "timestamp": "2020-03-04T15:11:44-08:00",
      "user": "Developer Toolkit Test Account"
    }

You can see here that the revision that was recorded was the SHA from the
current git project.  The revision can be any string, but the intent is to
provide some point in time for the application in question.

This workflow could be built into a CI/CD system to help indicate changes in
application behavior after deployments.

#### Tagging all applications for an account

Another use case, perhaps, is that you need to tag all applications in your
environment.  This could be daunting to do through the web UI, but at the CLI
this is pretty quick.

Here we'll couple the `newrelic` CLI with the `jq`  command to tag each
application returned in our query with a new tag.

    newrelic apm application get --accountId $ACCOUNT_ID | jq -r '.[].GUID' | xargs -I {} newrelic entities create-tags -g {} -t devkit:testing

In the above, we first run `apm application get` to return the applications for
our given account.  This could be filtered further to reduce the set of
results, but for illustration purposes this works well enough.

Next, we pipe the JSON output containing all the results to `jq`.  The
less-than-beautiful syntax there is picking out only the `GUID` field and
printing each results GUID on its own line.

Then we pass that to `xargs` to call the `newrelic` CLI to use the `entities
create-tags` the received GUID strings so that each application gets tagged.

[Jq][jq] can be a powerful way to manipulate JSON data at the command line.  If
you're not familiar with it, it's worth looking at to tie these sorts of
commands together.


### Want more?

If you'd like to engage with other community members, visit our
[discuss][discuss] page.  We welcome feature requests or bug reports on GitHub.
We also love Pull Requests.

[homebrew]: https://brew.sh/

[docker_image]: https://hub.docker.com/r/newrelic/cli

[releases]: https://github.com/newrelic/newrelic-cli/releases

[api_key]: https://docs.newrelic.com/docs/apis/get-started/intro-apis/types-new-relic-api-keys#personal-api-key

[jq]: https://stedolan.github.io/jq/

[discuss]: https://discuss.newrelic.com/c/build-on-new-relic/developer-toolkit