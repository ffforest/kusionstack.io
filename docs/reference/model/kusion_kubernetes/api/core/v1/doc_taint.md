# taint

Source: [base/pkg/kusion_kubernetes/api/core/v1/taint.k](https://github.com/KusionStack/konfig/blob/main/base/pkg/kusion_kubernetes/api/core/v1/taint.k)

This is the taint module in kusion\_kubernetes.api.core.v1 package.<br />This file was generated by the KCL auto-gen tool. DO NOT EDIT.<br />Editing this file might prove futile when you re-run the KCL auto-gen generate command.

## Schema Taint

The node this Taint is attached to has the "effect" on any pod that does not tolerate the Taint.

### Attributes

|Name and Description|Type|Default Value|Required|
|--------------------|----|-------------|--------|
|**effect**<br />Required. The effect of the taint on pods that do not tolerate the taint. Valid effects are NoSchedule, PreferNoSchedule and NoExecute.|str|Undefined|**required**|
|**key**<br />Required. The taint key to be applied to a node.|str|Undefined|**required**|
|**timeAdded**<br />TimeAdded represents the time at which the taint was added. It is only written for NoExecute taints.|str|Undefined|optional|
|**value**<br />The taint value corresponding to the taint key.|str|Undefined|optional|
<!-- Auto generated by kcl-doc tool, please do not edit. -->