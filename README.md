# K9s Cheat Sheet

> From first launch to fluent, keyboard-driven Kubernetes cluster operations.

[![K9s](https://img.shields.io/badge/K9s-Kubernetes%20TUI-FF6AC1?style=for-the-badge)](https://k9scli.io/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-cluster%20UI-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docs](https://img.shields.io/badge/Official-Documentation-111827?style=for-the-badge)](https://k9scli.io/)

K9s is a terminal UI for Kubernetes. It watches your cluster in real time and lets you
navigate, inspect, and manage resources without memorizing long `kubectl` pipelines.

This guide follows the workflow you will use day to day:

```text
Install → Connect → Navigate views → Filter & drill down → Inspect/edit/logs/shell
         → Port-forward & scale → Customize aliases/hotkeys/plugins → Master RBAC & skins
```

> [!TIP]
> If you remember only six keys, remember these:
>
> ```text
> ?     Help (always)
> :     Command mode  →  :pod  :deploy  :ns  :ctx
> /     Filter the current view
> d     Describe selection
> l     Logs
> s     Shell into pod
> ```

---

## Table of contents

- [1. Mental model](#1-mental-model)
- [2. Installation](#2-installation)
- [3. Five-minute quick start](#3-five-minute-quick-start)
- [4. Daily workflow](#4-daily-workflow)
- [5. CLI reference](#5-cli-reference)
- [6. Navigation and general keys](#6-navigation-and-general-keys)
- [7. Command mode and filtering](#7-command-mode-and-filtering)
- [8. Resource actions](#8-resource-actions)
- [9. Pods, containers, and logs](#9-pods-containers-and-logs)
- [10. Workloads, nodes, and RBAC views](#10-workloads-nodes-and-rbac-views)
- [11. Pulses, XRay, and cluster overview](#11-pulses-xray-and-cluster-overview)
- [12. Configuration layout](#12-configuration-layout)
- [13. `config.yaml` reference](#13-configyaml-reference)
- [14. Aliases](#14-aliases)
- [15. Hotkeys](#15-hotkeys)
- [16. Plugins](#16-plugins)
- [17. Skins and custom views](#17-skins-and-custom-views)
- [18. Node shell and shell pods](#18-node-shell-and-shell-pods)
- [19. RBAC for K9s users](#19-rbac-for-k9s-users)
- [20. Benchmarking](#20-benchmarking)
- [21. Recipes](#21-recipes)
- [22. Troubleshooting](#22-troubleshooting)
- [23. Mastery checklist](#23-mastery-checklist)
- [24. Official references](#24-official-references)
- [Quick-reference card](#quick-reference-card)

---

## 1. Mental model

K9s sits on top of your normal kubeconfig and Kubernetes API access:

```text
kubeconfig / context
      │
      ▼
k9s process          Terminal UI watching the API
      │
      ├── Views      Tables for pods, deploys, CRDs, …
      ├── Actions    Describe, edit, logs, shell, scale, delete
      └── Config     aliases, hotkeys, plugins, skins, per-context state
```

### What K9s is great at

- Live browsing of standard resources and CRDs.
- Fast drill-down: pod → containers → logs → shell.
- Keyboard-first operations: describe, edit, scale, restart, port-forward.
- Cluster overview via Pulses and dependency traversal via XRay.
- Extending the UI with aliases, hotkeys, plugins, and skins.

### What K9s does not replace

- `kubectl` for scripted automation and CI.
- Your cluster’s RBAC policy. If you cannot list a resource with kubectl, K9s cannot either.
- A secrets manager. Decoded secrets still appear on screen—treat the terminal carefully.
- Helm/GitOps tooling itself (though plugins can call them).

### Where configuration lives

| Location | Purpose |
| --- | --- |
| `$XDG_CONFIG_HOME/k9s/` | Global config, aliases, plugins, skins |
| `$XDG_DATA_HOME/k9s/clusters/<cluster>/<context>/` | Per-context state, favorites, optional overrides |
| `K9S_CONFIG_DIR` | Override the config directory entirely |

Run `k9s info` anytime to print the exact paths K9s is using on your machine.

Default config directories:

| OS | Default config path |
| --- | --- |
| Linux / Unix | `~/.config/k9s` |
| macOS | `~/Library/Application Support/k9s` |
| Windows | `%LOCALAPPDATA%\k9s` |

---

## 2. Installation

K9s targets Linux, macOS, and Windows. Current releases are published on
[GitHub Releases](https://github.com/derailed/k9s/releases) (for example `v0.51.x`).

### macOS

```bash
brew install derailed/k9s/k9s
# or
sudo port install k9s
```

### Linux

```bash
# Homebrew / Linuxbrew
brew install derailed/k9s/k9s

# Arch
pacman -S k9s

# Fedora 42+
dnf install k9s

# OpenSUSE
zypper install k9s

# Snap
snap install k9s --devmode

# Ubuntu / Debian package from latest release
wget https://github.com/derailed/k9s/releases/latest/download/k9s_linux_amd64.deb
sudo apt install ./k9s_linux_amd64.deb
rm k9s_linux_amd64.deb

# Universal installer helpers
curl -sS https://webinstall.dev/k9s | bash
pkgx k9s
```

### Windows

```powershell
winget install k9s
# or
scoop install k9s
# or
choco install k9s
```

### Go, Docker, and source

```bash
# Dev build via Go
go install github.com/derailed/k9s@latest

# Official image with your kubeconfig mounted
docker run --rm -it -v "$HOME/.kube/config:/root/.kube/config" derailed/k9s

# From source (needs a recent Go toolchain)
git clone https://github.com/derailed/k9s.git
cd k9s
make build && ./execs/k9s
```

### Preflight checks

```bash
# 256-color terminal
export TERM=xterm-256color

# Editor used by resource edit (e)
export EDITOR=nvim
export KUBE_EDITOR="$EDITOR"

# Confirm install and paths
k9s version
k9s info
```

K9s uses your normal Kubernetes credentials: `KUBECONFIG`, the current context, and any
cloud/OIDC auth plugins already working with `kubectl`.

---

## 3. Five-minute quick start

```bash
# 1. Confirm cluster access
kubectl get ns
kubectl config current-context

# 2. Launch K9s
k9s

# 3. Inside K9s
?                 # Help for the current view
:ns Enter         # Namespaces
:pod Enter        # Pods
/ frontend Enter  # Filter names matching frontend
d                 # Describe selected pod
l                 # Tail logs
esc               # Back out of logs
s                 # Shell into the pod (if permitted)
:q                # Quit
```

Useful launch variants:

```bash
k9s -n production          # Start in a namespace
k9s -c pod                 # Start directly on the pod view
k9s --context staging      # Use a non-default context
k9s --readonly             # Disable mutating actions
```

---

## 4. Daily workflow

### Start work

```bash
kubectl config get-contexts
k9s --context <ctx> -n <namespace>
```

Inside K9s:

```text
:ctx          Switch context
:ns           Switch namespace
0             Jump to all-namespaces favorite (if configured)
1–9           Jump to favorite namespaces
```

### Investigate a failing workload

```text
:pulse        Cluster health dashboard
:pod          Pod list
/! Running    Inverse filter: hide Running pods (example pattern)
d             Describe
l             Logs
p             Previous container logs (if restarted)
Shift-f       Port-forward
s             Shell
```

### Change something carefully

```text
y             View YAML
e             Edit live resource (opens $EDITOR / $KUBE_EDITOR)
s             Scale (on Deployment/StatefulSet/…)
r             Restart (on restartable workloads)
Ctrl-d        Delete with confirmation
Ctrl-k        Kill / force delete (no friendly confirm dialog)
```

### Prefer this decision flow

```text
Need cluster health first?
├─ Yes → :pulse   or   :xray deploy
└─ No
   Need a specific resource type?
   ├─ Yes → :pod / :deploy / :svc / short-name / alias
   └─ Already on the right view?
      ├─ Narrow results → /filter  or  /-l app=api
      ├─ Understand one object → d / y / Enter
      ├─ Debug runtime → l / s / a / Shift-f
      └─ Mutate → e / scale / restart / Ctrl-d
```

---

## 5. CLI reference

```bash
k9s help
k9s info
k9s version
```

| Flag / form | What it does |
| --- | --- |
| `k9s` | Launch with current kubeconfig context |
| `k9s -n <ns>` | Start in namespace `<ns>` |
| `k9s -c <cmd>` | Start in a view/command (`pod`, `deploy`, …) |
| `k9s --context <ctx>` | Use a specific kubeconfig context |
| `k9s --readonly` | Disable create/update/delete style actions |
| `k9s --kubeconfig <path>` | Use an alternate kubeconfig |
| `k9s --command <cmd>` | Same idea as `-c` on some versions—check `k9s help` |
| `k9s --headless` | Start with header hidden (version-dependent UI flags) |
| `k9s --logoless` | Hide the logo |
| `k9s --crumbsless` | Hide crumbs/history bar |

> [!NOTE]
> Run `k9s help` on your installed version. Flag names can gain aliases across releases;
> the binary help text is authoritative.

Useful environment variables:

| Variable | Purpose |
| --- | --- |
| `KUBECONFIG` | Standard kubeconfig path(s) |
| `K9S_CONFIG_DIR` | Override K9s config directory |
| `K9S_SKIN` | Force a skin name for this shell |
| `K9S_DEFAULT_PF_ADDRESS` | Override port-forward listen address |
| `K9S_FEATURE_GATE_NODE_SHELL` | Force node-shell feature gate on/off |
| `EDITOR` / `KUBE_EDITOR` | Editor for `e` edit actions |
| `TERM` | Prefer a 256-color value such as `xterm-256color` |

---

## 6. Navigation and general keys

Press `?` in any view. Help is contextual: resource actions, general keys, navigation, and
your hotkeys appear together.

### Always available

| Key | Action |
| --- | --- |
| `?` | Help |
| `Ctrl-a` | Alias / resource short-name browser |
| `:` | Command mode |
| `/` | Filter mode |
| `Esc` | Back / clear filter / cancel |
| `q` | Back (when not quitting) |
| `:q` | Quit |
| `Ctrl-c` | Quit (unless `noExitOnCtrlC` is set) |
| `Ctrl-r` | Reload / refresh |
| `Ctrl-u` | Clear the command line |
| `Tab` / `Backtab` | Next / previous field in dialogs |
| `Ctrl-e` | Toggle header |
| `Ctrl-g` | Toggle crumbs |
| `Ctrl-s` | Save / screendump |
| `Space` | Mark resource |
| `Ctrl-Space` | Mark range |
| `Ctrl-\` | Clear marks |
| `Shift-←` / `Shift-→` | Select previous / next column |
| `Ctrl-w` | Toggle wide columns |
| `Ctrl-z` | Toggle faults / toast resources |

### Movement

| Key | Action |
| --- | --- |
| `j` / `k` | Down / up |
| `h` / `l` | Left / right |
| `g` / `Shift-g` | Top / bottom |
| `Ctrl-b` / `Ctrl-f` | Page up / page down |
| `[` / `]` | History back / forward |
| `-` | Last used command / view |

### Sorting

| Key | Action |
| --- | --- |
| `Shift-n` | Sort by name |
| `Shift-a` | Sort by age |
| `Shift-s` | Sort by status |
| `Shift-o` | Sort by the selected column |

Many views add more `Shift-<letter>` sorts for view-specific columns. Check `?`.

---

## 7. Command mode and filtering

### Command mode (`:`)

```text
:pod                 Pods in the active namespace
:pods                Same (plural accepted)
:po                  Short name
:pod kube-system     Pods in kube-system
:pod /api            Pods whose name matches api
:pod app=web,env=dev Pods matching label selector
:pod @staging        Switch to context staging and show pods
:deploy              Deployments
:svc                 Services
:ns                  Namespaces
:ctx                 Contexts
:alias               Aliases
:pulses | :pu        Pulses dashboard
:xray deploy         XRay for deployments
:xray po myns        XRay pods in myns
:screendump | :sd    Saved screendumps / artifacts
:q                   Quit
```

Accepted forms: singular, plural, short-name, CRD name, or a custom alias.

Autosuggestion keys while typing a command:

| Key | Action |
| --- | --- |
| `↑` / `↓` | Move through suggestions |
| `Tab` / `Ctrl-f` / `→` | Accept suggestion |
| `Ctrl-w` / `Ctrl-u` | Clear command text |

### Filter mode (`/`)

| Form | Meaning |
| --- | --- |
| `/api` | Regex filter on the view |
| `/api\|web` | Match `api` or `web` |
| `/! Running` | Inverse regex: keep rows that do **not** match |
| `/-l app=api` | Label selector filter |
| `/-f frontend` | Fuzzy find |
| `Esc` | Clear filter / leave filter mode |

> [!TIP]
> Combine command-mode scoping with filter-mode refinement:
> `:pod kube-system` then `/coredns` is often faster than one giant selector.

---

## 8. Resource actions

These keys appear on most resource tables when RBAC and view support allow them:

| Key | Action |
| --- | --- |
| `Enter` | Drill into related resources / details |
| `d` | Describe |
| `y` | View YAML |
| `e` | Edit |
| `c` | Copy name (common) |
| `Ctrl-d` | Delete (confirm dialog) |
| `n` | Copy namespace (when namespaced) |
| `w` | Warp to the resource’s namespace (when available) |

Bulk work:

1. `Space` to mark rows (or `Ctrl-Space` for a range).
2. Run an action such as delete.
3. `Ctrl-\` to clear marks.

Read-only mode (`k9s --readonly` or `readOnly: true` in config) hides dangerous actions.

---

## 9. Pods, containers, and logs

### Pod view

| Key | Action |
| --- | --- |
| `d` | Describe pod |
| `y` | YAML |
| `e` | Edit |
| `l` | Logs |
| `p` | Previous logs (last terminated container instance) |
| `s` | Shell |
| `a` | Attach |
| `t` | Transfer files |
| `z` | Sanitize / clean completed pods (dangerous) |
| `o` | Jump to the node running the pod |
| `Shift-f` | Port-forward |
| `f` | Show active port-forwards |
| `Ctrl-k` | Kill pod |
| `Ctrl-d` | Delete pod |
| `Shift-j` | Jump to owner (ReplicaSet, Job, …) |

### Container view

| Key | Action |
| --- | --- |
| `l` / `p` | Logs / previous logs |
| `s` | Shell |
| `a` | Attach |
| `Shift-f` | Port-forward |
| `f` | Show port-forwards |

### Log view

| Key | Action |
| --- | --- |
| `0` | Tail (live) |
| `1` | Head |
| `2` | Last 1m |
| `3` | Last 5m |
| `4` | Last 15m |
| `5` | Last 30m |
| `6` | Last 1h |
| `s` | Toggle autoscroll |
| `t` | Toggle timestamps |
| `w` | Toggle wrap |
| `f` | Toggle fullscreen |
| `a` | Toggle all containers (when multi-container) |
| `m` | Mark a line |
| `Shift-c` | Clear the buffer |
| `Shift-l` | Toggle column lock |
| `c` | Copy |
| `Ctrl-s` | Save logs |
| `/` | Filter log lines |
| `Esc` / `q` | Back |

Default logger behavior is controlled in `config.yaml` under `k9s.logger`
(`tail`, `buffer`, `sinceSeconds`, `textWrap`, `showTime`, `fullScreen`).

---

## 10. Workloads, nodes, and RBAC views

### Deployments / scalable workloads

| Key | Action |
| --- | --- |
| `s` | Scale replicas |
| `r` | Restart (rollout restart style) |
| `i` | Set image |
| `z` | Related ReplicaSets (Deployment) |
| `l` / `p` | Logs / previous |
| `Shift-f` | Port-forward into a backing pod |
| `Shift-j` | Jump owner |

### ReplicaSets

| Key | Action |
| --- | --- |
| `Ctrl-l` | Rollback |

### CronJobs

| Key | Action |
| --- | --- |
| `t` | Trigger a Job now |
| `s` | Suspend / resume |

### Services

| Key | Action |
| --- | --- |
| `b` | Run / stop built-in HTTP benchmark |
| `Shift-f` | Port-forward |

### Secrets / ConfigMaps

| Key | Action |
| --- | --- |
| `x` | Decode Secret data (Secret view) |
| `u` | Used-by / references |
| `e` | Edit |

### Nodes

| Key | Action |
| --- | --- |
| `c` | Cordon |
| `u` | Uncordon |
| `r` | Drain |
| `s` | Node shell (only if nodeShell feature gate is enabled) |
| `y` | YAML |
| `Enter` | Pods on that node |

> [!CAUTION]
> Cordon, drain, kill, delete, scale-to-zero, and Secret decode are operationally sensitive.
> Prefer `--readonly` on shared jump hosts and production laptop sessions unless you need to mutate.

### Built-in RBAC exploration

Useful commands:

```text
:cr      ClusterRoles
:crb     ClusterRoleBindings
:role    Roles
:rb      RoleBindings
:sa      ServiceAccounts
```

K9s can help answer “what can this identity do?” by browsing roles and bindings visually.
Exact reverse-lookup flows vary slightly by version—use `Ctrl-a` and `?` inside those views.

---

## 11. Pulses, XRay, and cluster overview

### Pulses

```text
:pulses
:pu
```

Pulses is a high-level dashboard of cluster health and resource activity. Start here when you
do not yet know which namespace or workload is unhealthy.

### XRay

```text
:xray deploy
:xray po
:xray sts staging
:xray ds
:xray svc
:xray rs
```

XRay shows a resource and its related dependents so you can traverse ownership and impact
without manually hopping through `kubectl get` trees.

### Context and namespace switching

```text
:ctx                 List contexts; Enter to switch
:ctx production      Jump directly
:ns                  List namespaces
:pod @other-ctx      Switch context while opening pods
```

Favorite namespaces are remembered per context and mapped to number keys `0–9`.

---

## 12. Configuration layout

```text
$XDG_CONFIG_HOME/k9s/
├── config.yaml          Global K9s settings
├── aliases.yaml         Global command aliases
├── plugins.yaml         Global plugins (or plugins/ directory)
├── plugins/             Drop-in plugin files
└── skins/               Skin YAML files
      └── dracula.yaml

$XDG_DATA_HOME/k9s/
├── hotkeys.yaml         Global hotkeys (path can vary; confirm with k9s info)
├── plugins/             Additional plugin drop-ins
└── clusters/
    └── <cluster>/
        └── <context>/
            ├── config.yaml     Active ns, favorites, skin, feature gates
            ├── aliases.yaml    Context-specific aliases
            └── plugins.yaml    Context-specific plugins
```

> [!TIP]
> When in doubt, run `k9s info`. Paths differ across Linux, macOS, Windows, and custom
> `K9S_CONFIG_DIR` setups.

---

## 13. `config.yaml` reference

Minimal useful global config:

```yaml
# $XDG_CONFIG_HOME/k9s/config.yaml
k9s:
  liveViewAutoRefresh: false
  refreshRate: 2
  apiServerTimeout: 15s
  maxConnRetry: 5
  readOnly: false
  defaultView: ""
  noExitOnCtrlC: false
  skipLatestRevCheck: false
  disablePodCounting: false
  portForwardAddress: localhost

  ui:
    enableMouse: false
    headless: false
    logoless: false
    crumbsless: false
    reactive: false
    noIcons: false
    skin: ""                 # e.g. dracula

  shellPod:
    image: busybox:1.35.0
    namespace: default
    limits:
      cpu: 100m
      memory: 100Mi
    tty: true

  logger:
    tail: 100
    buffer: 5000
    sinceSeconds: -1
    fullScreen: false
    textWrap: false
    showTime: false

  thresholds:
    cpu:
      critical: 90
      warn: 70
    memory:
      critical: 90
      warn: 70

  imageScans:
    enable: false
    exclusions:
      namespaces:
        - kube-system
      labels: {}
```

Per-context example:

```yaml
# $XDG_DATA_HOME/k9s/clusters/<cluster>/<context>/config.yaml
k9s:
  cluster: my-cluster
  readOnly: false
  skin: in_the_navy
  namespace:
    active: default
    lockFavorites: false
    favorites:
      - all
      - default
      - kube-system
      - production
  view:
    active: po
  featureGates:
    nodeShell: false
```

Practical settings to learn first:

| Setting | Why it matters |
| --- | --- |
| `readOnly` | Safety on production contexts |
| `refreshRate` | UI poll interval vs API load |
| `logger.*` | Default log tail experience |
| `ui.skin` | Look and feel |
| `featureGates.nodeShell` | Allow node shell when intentional |
| `thresholds` | CPU/memory warn/critical coloring |
| `portForwardAddress` | Bind address for forwards |

---

## 14. Aliases

Create `$XDG_CONFIG_HOME/k9s/aliases.yaml`:

```yaml
aliases:
  # Short resource aliases (GVR form)
  pp: v1/pods
  dep: apps/v1/deployments
  crb: rbac.authorization.k8s.io/v1/clusterrolebindings

  # Alias can also encode a full command
  pos: pod kube-system app=fred,blee=duh

  # CRD example
  fred: acme.io/v1alpha1/fredericks
```

Then in K9s:

```text
:pp
:dep
:pos
Ctrl-a      # Browse everything available
```

Context-specific aliases can live under the cluster/context data directory as `aliases.yaml`.

As of recent K9s versions the filename and top-level key are plural: `aliases.yaml` / `aliases:`.

---

## 15. Hotkeys

Hotkeys jump to favorite commands without typing `:…` every time.

Example `$XDG_DATA_HOME/k9s/hotkeys.yaml`:

```yaml
hotKeys:
  shift-0:
    shortCut: Shift-0
    description: CoreDNS pods
    command: pods app=coredns

  shift-1:
    shortCut: Shift-1
    description: Deployments
    command: dp

  shift-2:
    shortCut: Shift-2
    description: XRay Deployments
    command: xray deploy

  shift-3:
    shortCut: Shift-3
    description: Pulses
    command: pulses
```

Rules of thumb:

- Do not override built-in K9s shortcuts.
- Keep descriptions short; they show up in `?`.
- Files are reloaded live—edit, save, use.

---

## 16. Plugins

Plugins bind a shortcut to an external command with selected-resource context.

Discovery order includes:

- `$XDG_CONFIG_HOME/k9s/plugins.yaml`
- `$XDG_CONFIG_HOME/k9s/plugins/`
- `$XDG_DATA_HOME/k9s/plugins/`
- `$XDG_DATA_DIRS/k9s/plugins`
- Per-context `plugins.yaml`

### Environment variables available to plugins

| Variable | Meaning |
| --- | --- |
| `$NAME` / `$RESOURCE_NAME` | Selected resource name |
| `$NAMESPACE` | Selected namespace |
| `$CONTEXT` / `$CLUSTER` | Active context / cluster |
| `$CONTAINER` | Container when applicable |
| `$POD` | Pod when in container view |
| `$FILTER` | Active filter |
| `$KUBECONFIG` | Kubeconfig path |
| `$USER` / `$GROUPS` | Auth subject info |
| `$RESOURCE_GROUP` / `$RESOURCE_VERSION` | API group/version |
| `$COL-<COLUMN>` | Value from a named column |

### Example plugin

```yaml
# $XDG_CONFIG_HOME/k9s/plugins.yaml
plugins:
  follow-logs:
    shortCut: Ctrl-L
    description: Follow logs (kubectl)
    scopes:
      - po
    command: kubectl
    background: false
    args:
      - logs
      - -f
      - --context
      - $CONTEXT
      - -n
      - $NAMESPACE
      - $NAME
```

Drop-in snippet style (filename becomes the plugin identity):

```yaml
# $XDG_DATA_HOME/k9s/plugins/misc/stern.yaml
shortCut: Shift-L
description: Stern logs
scopes:
  - pods
command: stern
background: false
args:
  - --context
  - $CONTEXT
  - -n
  - $NAMESPACE
  - $NAME
```

Community plugins ship in the
[derailed/k9s plugins directory](https://github.com/derailed/k9s/tree/master/plugins)
(Helm values, Flux, Argo, dive, debug containers, cert-manager, and more).

> [!WARNING]
> A plugin is arbitrary local command execution with cluster context. Review every plugin
> before installing it, especially ones marked dangerous or running with elevated tools.

---

## 17. Skins and custom views

### Skins

1. Copy or create a skin under `$XDG_CONFIG_HOME/k9s/skins/mytheme.yaml`.
2. Reference it globally:

```yaml
k9s:
  ui:
    skin: mytheme
```

3. Or per context in the context `config.yaml`: `skin: mytheme`.
4. Or for one shell: `export K9S_SKIN=mytheme`.

Stock and community skins live in the
[K9s skins directory](https://github.com/derailed/k9s/tree/master/skins).

Colors may be CSS-like names (`dodgerblue`) or hex (`#1e1e2e`). Use `default` for a
transparent background that keeps your terminal theme.

### Custom column views

Define `$XDG_CONFIG_HOME/k9s/views.yaml` (confirm exact path via docs/`k9s info` for your
version) to control columns and default sort per resource:

```yaml
views:
  v1/pods:
    columns:
      - NAME
      - READY
      - STATUS
      - RESTARTS
      - CPU
      - MEM
      - IP
      - NODE
      - AGE
    sortColumn: AGE:desc

  apps/v1/deployments:
    columns:
      - NAME
      - READY
      - UP-TO-DATE
      - AVAILABLE
      - AGE
    sortColumn: NAME:asc
```

Use `Ctrl-w` anytime to toggle wide columns on the current table.

---

## 18. Node shell and shell pods

Node shell lets you open a shell aligned with a node for host-level debugging. It is gated
because it is powerful.

Enable per context:

```yaml
k9s:
  featureGates:
    nodeShell: true
```

Or force via environment:

```bash
export K9S_FEATURE_GATE_NODE_SHELL=true
```

Global shell-pod defaults (image, namespace, resources) come from `k9s.shellPod` in
`config.yaml`. Prefer a minimal, pinned image and a dedicated debug namespace when you use
this in shared clusters.

---

## 19. RBAC for K9s users

K9s needs at least get/list/watch on the resources you expect to browse. Metrics require
access to `metrics.k8s.io` if you want CPU/memory columns.

### Sample read-only ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: k9s-reader
rules:
  - apiGroups: [""]
    resources: ["nodes", "namespaces", "persistentvolumes"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["rbac.authorization.k8s.io"]
    resources: ["clusterroles", "roles", "clusterrolebindings", "rolebindings"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["apiextensions.k8s.io"]
    resources: ["customresourcedefinitions"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["metrics.k8s.io"]
    resources: ["nodes", "pods"]
    verbs: ["get", "list", "watch"]
```

### Sample namespaced Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: k9s-reader
  namespace: default
rules:
  - apiGroups: ["", "apps", "autoscaling", "batch"]
    resources: ["*"]
    verbs: ["get", "list", "watch"]
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods", "nodes"]
    verbs: ["get", "list", "watch"]
```

Add `create` / `update` / `patch` / `delete` / `exec` / `portforward` only when those K9s
actions are required. Missing verbs usually show up as empty views, denied flashes, or
disabled actions—not as a special K9s error code.

---

## 20. Benchmarking

On Service (and related) views, `b` can start/stop K9s’ built-in HTTP benchmark against a
port-forwarded target. Use it for quick local pressure checks, not as a replacement for
proper load-testing tools.

Typical flow:

```text
:svc
Shift-f          # Port-forward the Service/Pod
b                # Toggle benchmark
```

Tune expectations: results depend on your laptop, network path, and whether you are hitting
a local forward or a real ingress path.

---

## 21. Recipes

### Debug a CrashLoopBackOff pod

```text
:pod
/ CrashLoop
d              # Events + status
l              # Current logs
p              # Previous logs if it already restarted
y              # Spec: probes, mounts, command
s              # Shell only if the container stays up long enough
```

### Watch a rollout

```text
:deploy
/ my-api
r              # Restart if you need a fresh rollout
Enter          # Drill to ReplicaSets/Pods
l              # Follow logs on a new pod
```

### Namespace triage

```text
:ns
Enter on offending ns
:pod
Ctrl-z         # Focus faults if supported in view
:event
/ Warning
```

### Safe production browsing

```bash
k9s --context prod --readonly -n production
```

Or permanently:

```yaml
k9s:
  readOnly: true
```

in that context’s config.

### Port-forward a local UI

```text
:pod
/ frontend
Shift-f        # Choose ports
# Browser → http://localhost:<local-port>
f              # Review active forwards later
```

### Useful personal aliases

```yaml
aliases:
  bad: pod /!Running
  web: pod app=web
  nodes: v1/nodes
  crds: apiextensions.k8s.io/v1/customresourcedefinitions
```

### Install a community plugin safely

1. Read the YAML in the official `plugins/` folder.
2. Copy only what you need into `$XDG_CONFIG_HOME/k9s/plugins/`.
3. Restart K9s or rely on reactive reload if enabled.
4. Press `?` on the target view and confirm the shortcut.

---

## 22. Troubleshooting

### `k9s: command not found`

```bash
command -v k9s
echo "$PATH"
k9s version
```

Reinstall with your package manager or place the binary on `PATH`.

### Empty views or forbidden errors

```bash
kubectl auth can-i list pods -n default
kubectl auth can-i get nodes
kubectl get --raw /apis/metrics.k8s.io/v1beta1/nodes
```

Fix RBAC or metrics-server before blaming K9s.

### Wrong cluster / namespace

```bash
kubectl config current-context
kubectl config get-contexts
k9s info
```

Inside K9s use `:ctx` and `:ns`, or launch with `--context` and `-n`.

### Edit does nothing / opens the wrong editor

```bash
echo "$EDITOR"
echo "$KUBE_EDITOR"
export KUBE_EDITOR=nvim
```

### Colors look wrong

```bash
echo "$TERM"
export TERM=xterm-256color
# Optional: pick a skin that matches your terminal
export K9S_SKIN=dracula
```

### Port-forward fails

- Confirm the container port exists (`y` / `d` on the pod).
- Check `portForwardAddress` / `K9S_DEFAULT_PF_ADDRESS`.
- Ensure RBAC allows `pods/portforward`.
- Look for a local port already in use.

### Shell / attach fails

- Ephemeral containers or distroless images may have no shell.
- Need `pods/exec` permission.
- For nodes, enable `nodeShell` deliberately and understand the blast radius.

### Config changes seem ignored

```bash
k9s info
ls -la "$HOME/.config/k9s"
```

Confirm you edited the directory K9s actually loads. Context-level settings override global
ones for that context.

### High API server load

Increase `refreshRate`, avoid unnecessary all-namespaces views, and disable features you do
not need (pod counting on nodes, image scans, aggressive plugins).

---

## 23. Mastery checklist

### Beginner

- [ ] Install K9s and run `k9s info`.
- [ ] Launch into a known namespace with `k9s -n …`.
- [ ] Use `?`, `:`, `/`, `Esc`, and `:q` confidently.
- [ ] Describe, YAML-view, and log a pod (`d`, `y`, `l`).
- [ ] Switch namespaces and contexts from inside K9s.

### Productive

- [ ] Filter with regex, inverse regex, labels, and fuzzy find.
- [ ] Shell/attach and port-forward without leaving the TUI.
- [ ] Scale and restart Deployments intentionally.
- [ ] Mark multiple resources and operate on the set.
- [ ] Save screendumps/logs with `Ctrl-s`.

### Power user

- [ ] Define personal aliases and hotkeys.
- [ ] Install and vet at least one plugin.
- [ ] Apply a skin and a custom view column set.
- [ ] Use Pulses and XRay during incident triage.
- [ ] Keep production contexts read-only by default.

### Team-ready

- [ ] Document the minimum RBAC for your K9s users.
- [ ] Share a reviewed plugins/aliases pack for the team.
- [ ] Standardize debug recipes (CrashLoop, rollout, node drain).
- [ ] Teach when to drop to `kubectl`/automation instead of the TUI.
- [ ] Review dangerous keystrokes (`Ctrl-k`, drain, decode Secret) in onboarding.

---

## 24. Official references

### Start here

- [K9s home](https://k9scli.io/)
- [Installation](https://k9scli.io/topics/install/)
- [Commands & key bindings](https://k9scli.io/topics/commands/)
- [Configuration](https://k9scli.io/topics/config/)
- [Aliases](https://k9scli.io/topics/aliases/)
- [Hotkeys](https://k9scli.io/topics/hotkeys/)
- [Plugins](https://k9scli.io/topics/plugins/)
- [Skins](https://k9scli.io/topics/skins/)
- [RBAC](https://k9scli.io/topics/rbac/)

### Project and community

- [GitHub repository](https://github.com/derailed/k9s)
- [Releases](https://github.com/derailed/k9s/releases)
- [Community plugins](https://github.com/derailed/k9s/tree/master/plugins)
- [Skins](https://github.com/derailed/k9s/tree/master/skins)
- [K9s Slack](https://k9sers.slack.com/)
- [Slack invite](https://join.slack.com/t/k9sers/shared_invite/zt-3360a389v-ElLHrb0Dp1kAXqYUItSAFA)

### Adjacent tools worth knowing

- [kubectl docs](https://kubernetes.io/docs/reference/kubectl/)
- [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [metrics-server](https://github.com/kubernetes-sigs/metrics-server)

---

## Quick-reference card

```text
# LAUNCH
k9s
k9s -n <ns>
k9s -c pod
k9s --context <ctx>
k9s --readonly
k9s info

# MOVE
?                 Help
:cmd              Command mode
/filter           Filter mode
Esc / q           Back / clear
:q / Ctrl-c       Quit
[ ] -             History back/forward / last
g G               Top / bottom
Ctrl-w            Wide columns
Ctrl-a            Aliases

# COMMANDS
:pod :deploy :svc :ns :ctx
:pulses :xray deploy
:pod kube-system
:pod app=web
:pod /api
:pod @other-ctx

# FILTERS
/api|web          Regex
/! Failed         Inverse
/-l app=api       Labels
/-f front         Fuzzy

# INSPECT
Enter             Drill down
d                 Describe
y                 YAML
l / p             Logs / previous
Ctrl-s            Save

# POD OPS
s                 Shell
a                 Attach
Shift-f           Port-forward
f                 Show forwards
o                 Show node
Shift-j           Jump owner
Ctrl-d            Delete
Ctrl-k            Kill

# WORKLOAD OPS
s                 Scale
r                 Restart
i                 Set image
b                 Bench (Service)

# NODE OPS
c / u             Cordon / uncordon
r                 Drain
s                 Node shell (if gated)

# CONFIG
~/.config/k9s/config.yaml
~/.config/k9s/aliases.yaml
~/.config/k9s/plugins.yaml
skins/ + hotkeys + per-context config
```

> Keep `?` close, keep production read-only until you need to mutate, and make aliases for
> the five views you open every day.

## License

You may reuse and adapt this cheatsheet under the [MIT License](LICENSE).
