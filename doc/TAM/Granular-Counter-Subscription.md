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

The TAM API previously allowed for predefined telemetry groups, like SAI_TAM_TELEMETRY_TYPE_SWITCH or SAI_TAM_TELEMETRY_TYPE_PORT. This provides extensibility to the API only to define new fixed collections. When collecting counters, all counters for the objects were reported.

However, the user may need enable telemetry for specific counters at runtime. For example, a user may wish to receive only port byte counters, or queue watermarks. Filtering the telemetry may allow a device to generate counters samples faster than possible when all counters are collected.

## 2.0 Configuration

The telemetry type object is defined with a new SAI_TAM_TEL_TYPE_ATTR_OBJECT_STATS to identify the desired counters. 

```
/**
 * @brief List of stats to collect
 *
 * @type sai_object_stat_list_t
 * @flags CREATE_AND_SET
 * @default empty
 */
 SAI_TAM_TEL_TYPE_ATTR_OBJECT_STATS,
```

This is a list of entries identifying the desired counters.

```c
typedef struct _sai_object_stat_id_t
{
    sai_object_id_t oid;
    sai_stat_id_t stat_enum;
    uint64_t telemetry_id;
} sai_object_stat_id_t;

typedef struct _sai_object_stat_list_t
{
    uint32_t count;
    sai_object_stat_id_t *list;
} sai_object_stat_list_t;    
```

In addition, the user may attach a NOS-assigned ID to the counter to allow for that counter in telemetry packets. For example, an IPFix export format may use this as the type of the enterprise-specific information element.

## 3 Configuration example

```c
sai_attr_list[0].id = SAI_TAM_TEL_TYPE_ATTR_TELEMETRY_TYPE;
sai_attr_list[0].value.u32 = SAI_TAM_TEL_TYPE_ATTR_OBJECT_STAT

sai_attr_list[1].id = SAI_TAM_TEL_TYPE_ATTR_OBJECT_STATS;
sai_attr_list[1].value.objectstatlist.count = 2;
sai_attr_list[1].value.objectstatlist.list[0].oid = sai_ipg1_obj;
sai_attr_list[1].value.objectstatlist.list[0].stat_enum = SAI_INGRESS_PRIORITY_GROUP_STAT_CURR_OCCUPANCY_BYTES;
sai_attr_list[1].value.objectstatlist.list[0].telemetry_id = 42;  /* NOS assigned ID */

sai_attr_list[1].value.objectstatlist.list[1].oid = sai_ipg2_obj;
sai_attr_list[1].value.objectstatlist.list[1].stat_enum = SAI_INGRESS_PRIORITY_GROUP_STAT_CURR_OCCUPANCY_BYTES;
sai_attr_list[1].value.objectstatlist.list[1].telemetry_id = 43;  /* NOS assigned ID */

sai_attr_list[2].id = SAI_TAM_TEL_TYPE_ATTR_REPORT_ID;
sai_attr_list[2].value.oid = sai_tam_report_obj; /* Report object created earlier and reused */

attr_count = 3;
sai_create_tam_tel_type_fn(
    &sai_tam_flow_tel_type_obj,
    switch_id,
    attr_count,
    sai_attr_list);

// Example: Create TAM object and attach to monitored objects:
sai_attr_list[0].value.objlist.count = 1;
sai_attr_list[0].value.objlist.list[0] = sai_tam_telemetry_obj;

sai_attr_list[1].id = SAI_TAM_ATTR_TAM_BIND_POINT_TYPE_LIST;
sai_attr_list[1].value.objlist.count = 1;
sai_attr_list[1].value.objlist.list[0] = SAI_TAM_BIND_POINT_TYPE_IPG;

attr_count = 2;
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
