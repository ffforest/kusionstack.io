# opensearch

## Index

- [OpenSearch](#opensearch)

## Schemas

### OpenSearch

OpenSearch is a module schema of OpenSearch. Currently, it only supports AWS OpenSearch Service

#### Attributes

| name | type | description | default value |
| --- | --- | --- | --- |
|**domainName** `required`|str|Name of the domain.||
|**engineVersion**|str|Either Elasticsearch_X.Y or OpenSearch_X.Y to specify the engine version for the Amazon OpenSearch Service domain. For example, OpenSearch_1.0 or Elasticsearch_7.9. Defaults to the latest version of OpenSearch.||
#### Examples

```
import opensearch.OpenSearch as o

accessories: {
    "opensearch": o.OpenSearch {
        domainName: "example"
        engineVersion: "OpenSearch_1.0"
    }
}
```

<!-- Auto generated by kcl-doc tool, please do not edit. -->
