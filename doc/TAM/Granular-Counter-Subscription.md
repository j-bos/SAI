#  Telemetry Granular Counter Subscription
-------------------------------------------------------------------------------
 Title       | Telemetry Granular Counter Subscription
-------------|-----------------------------------------------------------------
 Authors     | Jason Bos, Cisco
 Status      | In review
 Type        | Standards track
 Created     | 2023-01-11 - Initial Draft
 SAI-Version | 1.12
-------------------------------------------------------------------------------


## 1.0  Introduction

This spec enhances the existing TAM (Telemetry and Monitoring) spec to add granular counter subscription.

The TAM API previously allowed for predefined telemetry groups, like SAI_TAM_TELEMETRY_TYPE_SWITCH or SAI_TAM_TELEMETRY_TYPE_PORT. This provides extensibility to the API only to define new fixed collections. When collecting counters, all counters for the objects are reported.

However, the user may need enable telemetry for specific counters at runtime. For example, a user may wish to receive only port byte counters, or queue watermarks. Filtering the telemetry permits a device to generate counters samples faster than possible when all counters are collected.

Additionally, to provide a meaningful identifier for a counter. The meaning will depend on the report type.  
An implementation may use this to label the counter in .proto file, or as an enterprise-specific identifer in an IPFIX template.

## 2.0 Configuration


The subscription is represented by an object of SAI_TAM_COUNTER_SUBSCRIPTION. The creation of this object will indicate that a specific counter should be monitored.
```c
typedef enum _sai_tam_counter_subscription_attr_t
{
    /**
     * @brief Start of Attributes
     */
    SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_START,

    /**
     * @brief TAM telemetry type object
     *
     * @type sai_object_id_t
     * @flags MANDATORY_ON_CREATE | CREATE_ONLY
     * @objects SAI_OBJECT_TYPE_TAM
     */
    SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_TEL_TYPE = SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_START,

    /**
     * @brief Subscribed object
     *
     * @type sai_object_id_t
     * @flags MANDATORY_ON_CREATE | CREATE_ONLY
     */
    SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_OBJECT_ID,

    /**
     * @brief Subscribed stat enum
     *
     * @type sai_uint32_t
     * @flags MANDATORY_ON_CREATE | CREATE_ONLY
     */
    SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_STAT_ID,

     /**
     * @brief Telemetry label
     *
     * Label to identify this counter in telemetry reports.
     *
     * @type sai_uint64_t
     * @flags CREATE_ONLY
     * @default 0
     */
    SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_LABEL,

```

The attribute SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_LABEL may be used to provide the NOS-specific counter ID.


## 3 Configuration example

```c
sai_attr_list[0].id = SAI_TAM_TEL_TYPE_ATTR_TAM_TELEMETRY_TYPE;
sai_attr_list[0].value.u32 = SAI_TAM_TELEMETRY_TYPE_OBJECT_STAT;

attr_count = 1;
sai_create_tam_tel_type_fn(
    &sai_tam_tel_type_obj,
    switch_id,
    attr_count,
    sai_attr_list);

// Example: Create counter subscription(s)
sai_attr_list[0].id = SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_TEL_TYPE;
sai_attr_list[0].value.u32 = sai_tam_tel_type_obj;

sai_attr_list[1].id = SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_OBJECT_ID;
sai_attr_list[1].value.oid = sai_ipg1_obj;

sai_attr_list[2].id = SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_STAT_ID;
sai_attr_list[2].value.u32 = SAI_INGRESS_PRIORITY_GROUP_STAT_CURR_OCCUPANCY_BYTES;

sai_attr_list[3].id = SAI_TAM_COUNTER_SUBSCRIPTION_ATTR_LABEL;
sai_attr_list[3].value.u64 = 47;  // NOS-provided label

attr_count = 4;
sai_create_tam_counter_subscription_fn(
    &sai_tam_subscription_obj,
    switch_id,
    attr_count,
    sai_attr_list);

// Repeat as needed for desired counters

// Example: Create Telemetry object
sai_attr_list[0].id = SAI_TAM_TELEMETRY_ATTR_TAM_TYPE_LIST;
sai_attr_list[0].value.oid = sai_tam_tel_type_obj;

sai_attr_list[1].id = SAI_TAM_TELEMETRY_ATTR_COLLECTOR_LIST
sai_attr_list[1].value.objlist.count = 1;
sai_attr_list[1].value.objlist.list[0] = collector_obj;

attr_count = 2;
sai_create_tam_telemetry_fn(
    &sai_tam_telemetry_obj,
    switch_id,
    attr_count,
    sai_attr_list);

// Example: Create TAM object and bind to monitored objects:

sai_attr_list[1].id = SAI_TAM_ATTR_TAM_TELEMETRY_OBJECTS_LIST;
sai_attr_list[1].value.objlist.count = 1;
sai_attr_list[1].value.objlist.list[0] = sai_tam_telemetry_obj;

sai_attr_list[2].id = SAI_TAM_ATTR_TAM_BIND_POINT_TYPE_LIST;
sai_attr_list[2].value.objlist.count = 1;
sai_attr_list[2].value.objlist.list[0] = SAI_TAM_BIND_POINT_TYPE_IPG;

attr_count = 3;
sai_create_tam_fn(
    &sai_tam_obj,
    switch_id,
    attr_count,
    sai_attr_list);

sai_attr.id = SAI_INGRESS_PRIORITY_GROUP_ATTR_TAM;
sai_attr.value.oid = sai_tam_obj;

sai_set_ingress_priority_group_attribute_fn(
    sai_ipg1_obj,
    sai_attr);

sai_set_ingress_priority_group_attribute_fn(
    sai_ipg2_obj,
    sai_attr);
```
