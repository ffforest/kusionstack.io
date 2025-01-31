# kusion resource graph

Display a graph of all the resources' information of the target project and target workspaces

## Synopsis

Display information of all the resources of a project.

This command displays information of all the resources of a project in the current or specified workspaces.

```
kusion resource graph [flags]
```

## Examples

```
  # Display information of all the resources of a project in the current workspace.
  kusion resource graph --project quickstart
  
  # Display information of all the resources of a project in specified workspaces.
  kusion resource graph --project quickstart --workspace=dev,default
  
  # Display information of all the resource of a project in all the workspaces that has been deployed.
  kusion resource graph --project quickstart --all
  kusion resource graph --project quickstart -a
  
  # Display information of all the resource of a project with in specified workspaces with json format result.
  kusion resource graph --project quickstart --workspace dev -o json
```

## Options

```
  -a, --all                 Display all the resources of all the workspaces
      --backend string      The backend to use, supports 'local', 'oss' and 's3'
  -h, --help                help for graph
  -o, --output string       Specify the output format, json only
      --project string      The name of the target project
      --workspace strings   The name of the target workspace
```

## Options inherited from parent commands

```
      --profile string          Name of profile to capture. One of (none|cpu|heap|goroutine|threadcreate|block|mutex) (default "none")
      --profile-output string   Name of the file to write the profile to (default "profile.pprof")
```

## SEE ALSO

* [kusion resource](kusion-resource.md)	 - Observe Kusion resource information

###### Auto generated by spf13/cobra on 21-Jan-2025
