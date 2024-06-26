## kusion apply

Apply the operational intent of various resources to multiple runtimes

### Synopsis

Apply a series of resource changes within the stack.

 Create, update or delete resources according to the operational intent within a stack. By default, Kusion will generate an execution preview and prompt for your approval before performing any actions. You can review the preview details and make a decision to proceed with the actions or abort them.

```
kusion apply [flags]
```

### Examples

```
  # Apply with specified work directory
  kusion apply -w /path/to/workdir
  
  # Apply with specified arguments
  kusion apply -D name=test -D age=18
  
  # Apply with specified intent file
  kusion apply --intent-file intent.yaml
  
  # Apply with specifying intent file
  kusion apply --intent-file intent.yaml
  
  # Skip interactive approval of preview details before applying
  kusion apply --yes
  
  # Apply without output style and color
  kusion apply --no-style=true
```

### Options

```
  -a, --all --detail              Automatically show all preview details, combined use with flag --detail
  -D, --argument stringToString   Specify the top-level argument (default [])
      --backend string            the backend name
  -d, --detail                    Automatically show preview details with interactive options (default true)
      --dry-run                   Preview the execution effect (always successful) without actually applying the changes
  -h, --help                      help for apply
      --ignore-fields strings     Ignore differences of target fields
      --intent-file string        Specify the intent file path as input, and the intent file must be located in the working directory or its subdirectories
      --no-style                  no-style sets to RawOutput mode and disables all of styling
      --operator string           Specify the operator
  -o, --output string             Specify the output format
  -Y, --setting strings           Specify the command line setting files
      --watch                     After creating/updating/deleting the requested object, watch for changes
  -w, --workdir string            Specify the work directory
      --workspace string          the workspace name
  -y, --yes                       Automatically approve and perform the update after previewing it
```

### Options inherited from parent commands

```
      --profile string          Name of profile to capture. One of (none|cpu|heap|goroutine|threadcreate|block|mutex) (default "none")
      --profile-output string   Name of the file to write the profile to (default "profile.pprof")
```

### SEE ALSO

* [kusion](index.md)	 - Kusion is the Platform Orchestrator of KusionStack

###### Auto generated by spf13/cobra on 29-Mar-2024
