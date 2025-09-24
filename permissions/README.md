# Permise

Jaké permise dostává jaký uživatel:

```ts
export const groupPermissions: Record<groups, Permissions[]> = {
    global_admin: [ // Název role
        Permissions.program_create, // Název permisse
        Permissions.program_edit,
        Permissions.program_delete,
        Permissions.sensitive_data_access,
        Permissions.event_create,
        Permissions.event_edit,
        Permissions.event_delete,
        Permissions.manage_users,
        Permissions.manage_admins
    ],
    event_admin: [
        Permissions.program_create,
        Permissions.program_edit,
        Permissions.program_delete,
        Permissions.sensitive_data_access,
        Permissions.event_edit,
        Permissions.manage_users,
        Permissions.manage_admins
    ],
    program_manager: [
        Permissions.program_create,
        Permissions.program_edit,
        Permissions.program_delete
    ],
    lector: [
        Permissions.program_edit_own
    ],
    sensitive_data_viewer: [
        Permissions.sensitive_data_access
    ]
};```

Role lector je přidělována automaticky když lektor nějakého programu