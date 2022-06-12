[Distributed systems](/engineering/distributed-systems)
# Kubernetes configuration - ConfigMap and Secret

## Problem to solve - configuration change per environment

Regardless of whether application runs in container or directly on webserver, in most cases there's a need to tweak it configuration depending which environment or server it's running on.<br>
It might be database (or any other storage) connection string, it might be application's identity for logging or service discovery or it might be any other setting that might differ between non-production and production environments.

## Configuration - Kubernetes way

To solve configuration change problem `ConfigMap` and `Secret` have been introduced. They provide a way for setting up applications's (or even cross-application) configuration parts independently from their container images or pod configurations. 

ConfigMap and Secret are distinct Kubenernetes resource that has it's own decriptor present on k8s API server.<br>
They can be created from:
- file using `--from-file` where whole file becomes single variable in configuration
- file using `--from-env-file` where each key-value in file becomes a variable in configuration
- explicit value using `--from-literal`

### Usage

Both ConfigMap and Secret are commonly referenced in deployed applications through either:
- **enviroment variables** (`spec.containers[].env[].valueFrom.configMapKeyRef`)
- **files representing Secret/ConfigMap keys** in pod's volume (`spec.volumes[].configMap.name`) that are mounted on container level (`spec.containers[].volumeMounts[]`). 

Note that only mounted ConfigMap/Secret will update pods automatically (for .NET it's however broken until .NET 6, since FileWatcher of config files do not follow LastUpdateDate of symlinked file, but only of symlink itself).<br> Environment variables will not update until pod restarts. ConfigMap and Secret are both kept in `etcd`.

### When `ConfigMap` and when `Secret`?

For ConfigMap and Secret, the main difference is - obviously - the fact, that `Secret` is meant to hold confidential data like username and password, API keys etc. Fun fact is that... they are mostly the same on implementation level - for now. <br>Secret actually holds values as Base64-encoded, but that's not any protection at all. From my understanding, k8s creators meant this to be intent-based API semantics, that let users know, that secrets might eventually be encrypted somehow indepently from `etcd` (this seems to be an ambiguous case already, since k8s docs state that secrets are only encrypted, whereas docs of platforms like OpenShift state that they encrypt ConfigMaps too...).

One of the pitfalls on going "the right way" with splitting confidential and non-confidential configuration into Secret and ConfigMap is that they cannot be referenced from one to another. That might be an issue, if we want to have a single source of truth - like part of some node in config (provided it's JSON, XML, YAML etc.) that comes from ConfigMap (database address for example) and the other part comes from Secret (database login and password).<br> We cannot do this right now (at least if application cannot merge such disjoint configs itself). Current solution is to simply throw everything together into Secret.

Discussion on this is still being held active on GitHub when I'm writing the note:
https://github.com/kubernetes/kubernetes/issues/79224