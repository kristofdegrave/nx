`workspace.json`:

```json
//...
"frontend": {
    "architect": {
        //...
        "ls-project-root": {
            "builder": "@nrwl/workspace:run-commands",
            "options": {
                "commands": [
                    {
                    "command": "ls apps/frontend/src"
                    }
                ]
            }
        }
    }
}
```

```bash
<%= cli %> run frontend:ls-project-root
```

##### Chaining commands, interpolating args and setting the cwd

Let's say each of our workspace projects has some custom bash scripts in a `scripts` folder.
We want a simple way to create empty bash script files for a given project, that have the execute permissions already set.

Given that Nx knows our workspace structure, we should be able to give it a project and the name of our script, and it should take care of the rest.

The `commands` option accepts as many commands as you want. By default, they all run in parallel.
You can run them sequentially by setting `parallel: false`:

```json
"create-script": {
    "builder": "@nrwl/workspace:run-commands",
    "options": {
        "commands": [
            {
            "command": "mkdir -p scripts"
            },
            {
            "command": "touch scripts/{args.name}.sh"
            },
            {
            "command": "chmod +x scripts/{args.name}.sh"
            }
        ],
        "cwd": "apps/frontend",
        "parallel": false
    }
}
```

By setting the `cwd` option, each command will run in the `apps/frontend` folder.

We run the above with:

```bash
<%= cli %> run frontend:create-script --args="--name=example"
```

Notice the `--args="--name=example"` syntax: we can send custom arguments that will be interpolated into our commands via `{args.name}`

##### Custom **done** conditions

Normally, `run-commands` considers the commands done when all of them have finished running. If you don't need to wait until they're all done, you can set a special string, that considers the command finished the moment the string appears in `stdout` or `stderr`:

```json
"finish-when-ready": {
    "builder": "@nrwl/workspace:run-commands",
    "options": {
        "commands": [
            { "command": "echo 'READY' && sleep 5 && echo 'FINISHED'" }
        ],
        "readyWhen": "READY"
    }
}
```

```bash
<%= cli %> run frontend:finish-when-ready
```

The above command will finish immediately, instead of waiting for 5 seconds.

##### Nx Affected

The true power of `run-commands` comes from the fact that it runs through `nx`, which knows about your dependency graph. So you can run **custom commands** only for the projects that have been affected by a change.

We can create some configurations to generate docs, and if run using `nx affected`, it will only generate documentation for the projects that have been changed:

```bash
nx affected --target=generate-docs
```

```json
//...
"frontend": {
    "architect": {
        //...
        "generate-docs": {
            "builder": "@nrwl/workspace:run-commands",
            "options": {
                "commands": [
                    {
                    "command": "npx compodoc -p apps/frontend/tsconfig.app.json"
                    }
                ]
            }
        }
    }
},
"api": {
    "architect": {
        //...
        "generate-docs": {
            "builder": "@nrwl/workspace:run-commands",
            "options": {
                "commands": [
                    {
                    "command": "npx compodoc -p apps/api/tsconfig.app.json"
                    }
                ]
            }
        }
    }
}
```
