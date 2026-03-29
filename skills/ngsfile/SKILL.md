---
name: ngsfile
description: Write and maintain ngsfile — a project-specific CLI script in NGS. Auto-invoked when working with files named "ngsfile" or when the user asks to create/edit project commands via ngsfile.
---

# ngsfile

An `ngsfile` is a script in NGS that lives in a file named `ngsfile` (no extension) at the project root. Running `ngs .` in that directory executes it. It serves as a project-specific CLI — a replacement for Makefiles, npm scripts, or shell script collections.

**IMPORTANT: Immediately load the `ngs:write` skill using the Skill tool before proceeding.**

Always start your response with `[ NGS ngsfile skill active ]`.

## Structure

Every ngsfile is wrapped in a top-level `ns { ... }` block. This creates a namespace where each exported function becomes a subcommand:

```ngs
ns {
    F deploy() { ... }
    F test() { ... }
}
```

Usage: `ngs . deploy`, `ngs . test`

### Passing state into the namespace

Use `ns(name=value) { ... }` to bind external values inside the namespace:

```ngs
ns(t=test) {
    t("my test", { ... })
}
```

This is commonly used to bring `test` in as `t` for writing test suites.

### Nested namespaces for subcommand groups

Use nested `ns { }` blocks assigned to variables for hierarchical commands:

```ngs
ns {
    api = ns {
        F list() { ... }
        F create(name) { ... }
    }

    db = ns {
        F migrate() { ... }
        F seed() { ... }
    }
}
```

Usage: `ngs . api list`, `ngs . db migrate`

### Private/internal functions

Prefix with `_` to keep functions out of the public CLI surface.

## Common Patterns

### Environment variables with defaults

```ngs
ns {
    F url() ENV.get('URL', "http://localhost:3000/api/v1/items")
    F port() ENV.get('PORT', '8080').Int()
}
```

### Setting project-wide AWS environment

Override environment at the top of the namespace:

```ngs
ns {
    global ENV = {
        'AWS_PROFILE': 'myprofile'
        'AWS_REGION': 'eu-central-1'
    } + ENV

    # ... rest of commands
}
```

### Cached expensive lookups

Use `cached()` for values that are expensive to compute and reused across commands:

```ngs
ns {
    account = cached({ ``log: aws sts get-caller-identity``.Account })
    domain = cached({ ``log: aws cloudformation describe-stacks --stack-name my-stack``.the_one().Outputs.Hash('OutputKey', 'OutputValue').Domain })
}
```

### REST API helpers with curl

```ngs
ns {
    F url() ENV.get('URL', "http://localhost:8080")

    F get(path) ``curl -s "${url()}${path}"``

    F post(path, data:Hash) {
        ``curl -s -X POST -H 'Content-type: application/json' "${url()}${path}" -d ${data.encode_json()}``
    }
}
```

### Test suites

Use `ns(t=test)` and `section` blocks to organize tests:

```ngs
ns(t=test) {
    F url() ENV.get('URL', "http://localhost:8080/")

    F test() {
        section "fixtures" {
            id = 'test_01'
            data = {'id': id, 'field1': 'value1'}
        }

        t("Insert data", {
            $(curl --fail-with-body -s -X PUT "${url()}${id}" -H "Content-Type: application/json" -d ${data.encode_json()})
        })

        t("Verify inserted data", {
            ``curl -s "${url()}${id}"``.assert(data)
        })

        t("Delete data", {
            $(curl --fail-with-body -s -X DELETE "${url()}${id}")
        })
    }
}
```

### CloudFormation stack outputs

```ngs
ns {
    F _outputs(stack_name:Str) {
        ``log: aws cloudformation describe-stacks --stack-name ${stack_name}``.the_one().Outputs.Hash('OutputKey', 'OutputValue')
    }
}
```

### Multiple dispatch for flexible commands

Define the same function with different parameter types or patterns:

```ngs
ns {
    F call_grpc(env, op, data:Str='') {
        ``grpcurl -d $data $*{argv} "service.${op}"``
    }

    F call_grpc(env, op, data:Hash) call_grpc(env, op, data.encode_json())
}
```

## Running ngsfile commands

* `ngs .` — runs `main()` if defined (typically shows help)
* `ngs . COMMAND` — runs the named function
* `ngs . NAMESPACE COMMAND` — runs a function inside a nested namespace
* `ngs . COMMAND ARG1 ARG2` — arguments are passed to the function; typed parameters (e.g., `port:Int`) are automatically parsed

## Guidelines

* Keep ngsfile focused on project-specific CLI needs: building, deploying, testing, debugging
* Use `log()` for status messages (timestamped, to stderr)
* Use `$(log: COMMAND)` to show and run external commands
