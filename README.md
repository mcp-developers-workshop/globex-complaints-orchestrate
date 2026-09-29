# globex-complaints-orchestrate

Starter workspace for Module 5 of the MCP in Practice for Developers workshop.

It holds the three watsonx Orchestrate artefacts that put the Globex complaints MCP
server behind an Orchestrate agent: a **connection** that authenticates to Keycloak, an
**MCP toolkit** that registers the remote server, and a **native agent** that uses its
tools.

## Layout

```
├── connections/
│   └── globex-complaints-connection.yaml
├── toolkits/
│   └── globex-complaints.yaml
├── agents/
│   └── complaints_analyser.yaml
└── workspace_config.yaml
```

## Before you start

Replace `<OPENSHIFT_SUBDOMAIN>` with your cluster's subdomain. It appears in the
connection and toolkit files:

```sh
grep -rl '<OPENSHIFT_SUBDOMAIN>' . | xargs sed -i '' 's|<OPENSHIFT_SUBDOMAIN>|apps.your-cluster.example.com|g'
```

Drop the `''` after `-i` on Linux.

Check nothing was missed:

```sh
grep -rn '<OPENSHIFT_SUBDOMAIN>' .
```

## Import order

The order matters. The toolkit's `app_id` refers to the connection, and the agent's
`tools` refer to the toolkit's tools.

```sh
orchestrate connections import -f connections/globex-complaints-connection.yaml

orchestrate connections configure \
  -a globex-complaints-connection \
  --kind oauth_auth_client_credentials_flow \
  --type team \
  --env draft

orchestrate connections set-credentials \
  -a globex-complaints-connection \
  --env draft \
  --client-id "<YOUR_CLIENT_ID>" \
  --client-secret "<YOUR_CLIENT_SECRET>" \
  --token-url "https://sso.<OPENSHIFT_SUBDOMAIN>/realms/internal/protocol/openid-connect/token"

orchestrate toolkits import -f toolkits/globex-complaints.yaml
orchestrate tools list

orchestrate agents import -f agents/complaints_analyser.yaml
orchestrate agents deploy --name complaints_analyser
```

The client ID and secret come from the `internal` Keycloak realm. You pass them as
flags to `set-credentials` and Orchestrate stores them against the connection. They are
never written to a file here, which is why the YAML is safe to commit.

## Notes

- `llm: groq/openai/gpt-oss-120b` in the agent is a model Orchestrate hosts itself. It
  needs no credentials of your own.
- `workspace_config.yaml` is a placeholder. See the comments in the file.

The full walkthrough is Module 5 of the workshop guide.
