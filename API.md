1.  [Customer Management](#customer-management)
  1.  [Create](#create-a-customer)
  1.  [List](#list-multiple-customers)
  1.  [Get](#get-a-customer)
  1.  [Update](#update-a-customer)
  1.  [Delete](#delete-a-customer)
1.  [User Management](#user-management)
  1.  [Create](#create-a-user)
  1.  [Verify](#verify-a-user)
  1.  [Validate](#validate-a-user)
  1.  [List](#list-multiple-users)
  1.  [Get](#get-a-user)
  1.  [Update](#update-a-user)
  1.  [Delete](#delete-a-user)
  1.  [Password Reset](#password-reset)
  1.  [Password Clear](#password-clear)
1.  [Role Management](#role-management)
  1.  [Create](#create-a-role)
  1.  [List](#list-multiple-roles)
  1.  [Get](#get-a-role)
  1.  [Update](#update-a-role)
  1.  [Delete](#delete-a-role)
1.  [Access Management](#access-management)
  1.  [Create](#create-an-access)
  1.  [List](#list-multiple-accesses)
  1.  [Get](#get-an-access)
  1.  [Update](#update-an-access)
  1.  [Delete](#delete-an-access)
1.  [Session Management](#session-management)
  1.  [Create](#create-a-session)
  1.  [Create SSO](#create-a-session-using-sso)
  1.  [Verify](#verify-a-session)
  1.  [Choose Domain](#choose-a-domain)
  1.  [Activity](#session-activity)
  1.  [Logout](#session-logout)
  1.  [Single Sign On](#single-sign-on)
  1.  [List](#list-multiple-sessions)
  1.  [Get](#get-a-session)
  1.  [Delete](#delete-a-session)
1.  [SMS Carrier Management](#sms-carrier-management)
  1.  [Create](#create-a-sms-carrier)
  1.  [List](#list-multiple-sms-carriers)
  1.  [Get](#get-a-sms-carrier)
  1.  [Update](#update-a-sms-carrier)
  1.  [Delete](#delete-a-sms-carrier)

The ClearSky Data Authentication server provides **customer**, **user**, **role**, **access**, and **session** management.

Calling any of the API methods below may result in an error. All errors are represented to the user as:

    {
        "status": "error",
        "message": [
            "\n  Postgres error: 23505\n    duplicate key value violates unique constraint \"domain_domain_name_key\"\n  DETAIL: Key (domain_name)=(xyz.com) already exists."
        ],
        "code": 400
    }

# Customer Management
Customers have the following database attributes. There is one preloaded customer - 'clearskydata.com'

Field | Type | Description
-----|------|-----------------------------------------------
`customer_id`  | `integer`| This is the assigned customer id (unique)
`customer_name` | `string`| The name of the customer. It must be a unique domain name like 'foo.com'
`idle_timeout`  | `integer` | The idle timeout in seconds for sessions in this domain. The default is 15 minutes (15*60)
`mgmt_app_ip`  | `inet` | The IPv4 address of the management application
`mgmt_app_port`  | `int` | The IPv4 port of the management application

## Create a customer

    POST /customer-new

This method creates a customer.

Parameter | Type    | Description
-------------|---------|-----------------------------------------------
`customer_name` | `string`| **Required**. The name of the customer to add. *Must be unique*
`customer_id` | `integer`| **Optional**. The customer id. *Must be in the range 0x10000 and 0xfffff* and *must be unique*
`idle_timeout` | `integer` | *Optional*. Session timeout in seconds. Default is 15 minutes (900 seconds)
`mgmt_app_ip`   | `string`| *Optional*. The IPv4 address of the management appliance
`mgmt_app_port`   | `integer`| *Optional*. The IPv4 port of the managment appliance

This method returns a new domain object. For example:
    {
        "mgmt_app_ip": null,
        "idle_timeout": 1000,
        "mgmt_app_port": null,
        "customer_id": 65834,
        "customer_name": "testing.com"
    }

## List mutliple customers

    GET /customers

This method allows you to select all customers (no parameters) or to select specific customers based on search criteria.

Parameter | Type    | Description
-------------|---------|-----------------------------------------------
`name_match` | `string`| *Optional*. This is a posix-style regular expression that is used to find the matching customer

A list of matching customers is returned. Each entry in the list gives a `customer_id` and a `customer_name`.
The following result used a name_match = ^c.*$

    [
        {
            "customer_id": 131072,
            "customer_name": "clearskydata.com"
        },
        {
            "customer_id": 65537,
            "customer_name": "cars.com"
        }
    ]

## Get a customer

    GET /customer-<customer_id>

Get a customer (by `customer_id`)

Parameter  | Type    | Description
-------------|---------|-----------------------------------------------
`customer_id` | `integer`| **Required**. The customer id to get

Returns a customer object. For example:

    {
        "customer_name": "xyz.com",
        "customer_id": 234123,
        "idle_timeout": 1234,
        "mgmt_app_ip": null,
        "mgmt_app_port": null,

    }

## Update a customer

    PUT /customer-<customer-id>

Allows changes to made to the customer.

Parameter   | Type    | Description
-------------|---------|-----------------------------------------------
`customer_id` | `integer`| **Required**. The `customer id` to modify
`mgmt_app_url` | `string` | *Optional*. The url for the managment appliance
`mgmt_app_port` | `string` | *Optional*. IPv4 port for managment appliance
`idle_timeout` | `integer` | *Optional*. Idle timeout for sessions within customer

Returns a customer object. For an example customer return object see [Get a Customer](#get-a-customer)

## Delete a customer

    DELETE /customer-<customer_id>

Delete a customer (by id)

Parameter   | Type    | Description
------------|---------|-----------------------------------------------
`customer_id` | `integer`| **Required**. The customer id to delete

Returns the customer object deleted. For an example customer return object see [Get a customer](#get-a-customer)

# User Management

Users have the following attributes. **All user creation requires a two-factor verfication** except for ClearSky employees
who are validated against LDAP database.

Name | Type | Description
-----|------|-----------------------------------------------
`user_id` | `integer` | The id assigned to the user
`email` | `string` | The user's email address
`password` | `string` | A (strongly) hashed password
`sms_number` | `string` | A telephone number for sms verification
`nickname` | `string` | *Optional*. This must be a unique nickname (across all users in all domains) that the user can employ when logging in rather than their email address
`user_state` | One of: `unverified`, `verified`, `disabled` | The state of the user's record. Only `verified` sessions can log in
`two_factor_type` | One of: `email`, `sms` | The default is `email` *Note: `sms` is currently unsupported*
`verified_on` | `timestamp` | When the user was verified
`verify_code` | `integer` | The verification code for an `unverified` user
`password_expires_at` | `timestamp` | *Optional*. If specified, then the lifetime of a user's password (not currently supported)

## Create a user

    POST /user-new

This method creates a new user. This method must specify the user's email address.

Any new user is created in the `unverified` user_state, has a non-zero `verify_code` and must be subsequently
verified (see [Verify a user](#verify-a-user)) to get to the `verified` user_state. Once `verified` the user can
be used to create new sessions (login)

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`email` | `string` | **Required**. The user's email address
`password` | `string` | **Required**.
`two_factor_type` | One of `email`, `sms` | *Optional*. The default is `email` *Note: `sms` is currently unsupported*
`sms_number` | `string` | *Optional*. A telephone number for sms verification - must also specify `sms_carrier_id`
`sms_carrier_id` | `integer` | *Optional*. The SMS carrier for the telephone number - must also specify `sms_number`
`nickname` | `string` | *Optional*. This must be a unique nickname (across all users in all domains) that the user can employ when logging in rather than their email address
`full_name` | `string` | *Optional*. The name of the individual. This name **does not** have to be *unique across all users*.

This method returns a new user object. For example:

    {
        "sms_number": null,
        "user_id": 3,
        "password_expires_at": null,
        "two_factor_type": "email",
        "nickname": "Foo",
        "verify_code": 552459,
        "user_state": "unverified",
        "sms_carrier_id": null,
        "email": "foo@xyz.com",
        "verified_on": null,
        "full_name": null
    }


## Verify a user

    PUT /user-verify

Verify a user record. This uses the verification code (`verify_code`) generated when the user record was [created](#create-a-user)

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`user_name` | `string`| **Required**. The name of the user to verify - this can be the email or the nickname for the user
`password` | `string`| **Required**. The user's password
`verify_code` | `integer`| **Required**. The verification code

This method returns the verified user object. For example:

    {
        "sms_number": null,
        "user_id": 3,
        "password_expires_at": null,
        "two_factor_type": "email",
        "nickname": "Foo",
        "verify_code": 0,
        "user_state": "unverified",
        "sms_carrier_id": null,
        "email": "foo@xyz.com",
        "full_name": null,
        "verified_on": "2015-03-14 14:31:25.881635"
    }

## Validate a user

    GET /user-validate

Validate a user record. Simply validate the credentials and returns the role within the customer specified. The customer id must exist and there must be a valid access record for the user within the customer.

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`user_name` | `string`| **Required**. The name of the user to verify - this can be the email or the nickname for the user
`password` | `string`| **Required**. The user's password
`customer_id` | `integer`| **Required**. The customer scope to check the user within

This method returns a validated object replete with role information. *NOTE:* no session is established.

    {
        "password_expires_at": null,
        "user_verify_code": 0,
        "was_reset": null,
        "customer_name": "clearskydata.com",
        "sms_number": null,
        "user_id": 2,
        "user_state": "verified",
        "support_cases": "modify",
        "role_id": 1,
        "billing_usage": "read",
        "storage_alerts": "modify",
        "customer_id": 534111,
        "email": "foo@xyz.com",
        "storage_charts": "read",
        "idle_timeout": 900,
        "role_name": "System Admin",
        "support_docs": "read",
        "support_downloads": "read",
        "nickname": "Foo",
        "mgmt_app_url": "mgmt.clearsky-data.net",
        "verified_on": "2015-07-01 12:00:31.093402",
        "storage_l2_config": "modify",
        "admin_center": "modify",
        "two_factor_type": "email",
        "billing_invoices": "read",
        "storage_config": "modify",
        "access_id": 2,
        "mgmt_app_port": 443,
        "sms_carrier_id": null,
        "sfdc_info": "modify",
        "full_name": null
    }


## List mutliple users

    GET /users

This method can view mutliple users from the database. The search can be controlled by search parameters

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`email_match`| `string` | *Optional*. Posix regular expression to match against emails
`nickname_match`| `string` | *Optional*. Posix regular expression to match nickname

Returns a list of matching users:

    [
        {
            "user_id": 1,
            "email": "foo@xyz.com"
        },
        {
            "user_id": 3,
            "email": "foobar@xyz.com"
        }
    ]

## Get a user

    GET /user-<user_id>

Get a user (by `user_id`)

Parameter  | Type    | Description
-------------|---------|-----------------------------------------------
`user_id` | `integer`| **Required**. The user id to get

A user object. For example:

    {
        "sms_number": null,
        "user_id": 1,
        "password_expires_at": null,
        "two_factor_pref": "email",
        "email": "foo@xyz.com",
        "state": "verified",
        "nickname": "Foo",
        "domain_id": 2,
        "verified_on": "2015-03-13 13:33:28.191980",
        "full_name": "Bowser McRuff"
    }

## Update a user

    PUT /user-<user-id>

Allows changes to made to a user.

Changing a password requires that both the `password` and `old_password` parameters must be specified

Parameter    | Type    | Description
-------------|---------|-----------------------------------------------
`user_id` | `integer`| **Required**. The `user_id` to to modify
`password` | `string` | *Optional*. If changing password, then the new password.
`old_password` | `string` | *Optional*. The original password - must match to set the user's new `password`
`sms_number` | `string` | *Optional*. The `sms` number for the user
`two_factor_type` | One of: `email`, `sms` | *Optional*. 
`nickname` | `string` | *Optional*. This must be a unique nickname (across all users in all domains)
`password_expires_at` | `string` | *Optional*. The time/date the password will expire
`full_name` | `string` | *Optional*. The name of the individual. This name **does not** have to be *unique across all users*.

Returns a user object. For an example user return object see [Get a user](#get-a-user)

## Delete a user

    DELETE /user-<user_id>

Delete a user (by id)

Parameter | Type    | Description
-------------|---------|-----------------------------------------------
`user_id` | `integer`| **Required**. The user id to delete

Returns the user object that was deleted. For an example user return object see [Get a user](#get-a-user)

## Password reset

    PUT /user-<user-id>/password-reset

Reset a user's password. The overwritten password is saved to the old password list if the reset password
is itself not a reset password: If two resets are done in a row, the first reset password wlll not be saved
away as a user's password.

The password reset will result in the account being placed back into the 'unverified' state and a verification
code is generated. The user needs to go through the typical verification process.

An email is sent to the user's email address with the user account verification code.

Parameter | Type    | Description
-------------|---------|-----------------------------------------------
`user_id` | `integer`| **Required**. The user id to reset the password of
`password` | `string` | **Required**. The temporary password to set in the re

A user record identical to the record returned from create is sent back - an unverified user record complete
with verification code.

## Password clear

    PUT /users/password-clear

The clears the last N passwords and/or passwords that are in the database since a particular date.

**NOTE:** all old password for all users is changed by this function. This is system-wide policy
enforced by the caller. No old passwords are ever automatically purged by the authentication server.

Parameter | Type    | Description
-------------|---------|-----------------------------------------------
`keep` | `integer`| The maximum number of old_passwords to retain
`keep_since` | `string` | The oldest date to retain passwords since. This is in postgresql acceptable date formats (see [Postgres date input formats](http://www.postgresql.org/docs/9.3/static/datatype-datetime.html#DATATYPE-DATETIME-DATE-TABLE))

Returns {'cleared': 'ok'} on success, an error otherwise


# Role Management

A role is a collection of permissions to access particular areas of the ClearSky product

Name | Type | Description
-----|------|-----------------------------------------------
`role_id` | `integer` | The id assigned to the role
`role_name` | `string` | The name of the role (must be unique)
`admin_center` | `permission_enum` | one of 'no_access', 'read', or 'modify'
`storage_l2_config` | `permission_enum` | one of 'no_access', 'read', or 'modify'
`storage_config` | `permission_enum` | one of 'no_access', 'read', or 'modify'
`storage_charts` | `permission_enum` | one of 'no_access', 'read', or 'modify'
`storage_alerts` | `permission_enum` | one of 'no_access', 'read', or 'modify'
`billing_invoices` | `permission_enum` | one of 'no_access', 'read', or 'modify'
`billing_usage` | `permission_enum` | one of 'no_access', 'read', or 'modify'
`support_docs` | `permission_enum` | one of 'no_access', 'read', or 'modify'
`support_downloads` | `permission_enum` | one of 'no_access', 'read', or 'modify'
`support_cases` | `permission_enum` | one of 'no_access', 'read', or 'modify'
`sfdc_info` | `permission_enum` | one of 'no_access', 'read', or 'modify'

The following table are the databases predefined roles:

role_id |     role_name      | admin_center | storage_l2_config | storage_config | storage_charts | storage_alerts | billing_invoices | billing_usage | support_docs | support_downloads | support_cases | sfdc_info
---------|--------------------|--------------|-------------------|----------------|----------------|----------------|------------------|---------------|--------------|-------------------|---------------|----------- 
1 | System Admin       | modify       | modify            | modify         | read           | modify         | read             | read          | read         | read              | modify        | modify
2 | Storage Admin      | read         | read              | modify         | read           | modify         | no_access        | no_access     | read         | read              | modify        | read 
3 | Finance            | no_access    | no_access         | no_access      | no_access      | no_access      | read             | read          | read         | read              | read          | read
4 | Support Level 1        | read         | read              | read           | read           | read           | read             | read          | read         | read              | modify        | read
6 | Guest | no_access    | no_access         | no_access      | no_access      | no_access      | no_access        | no_access     | read         | read              | modify        | no_access

## Create a role

    POST /role-new

This method creates a new role. Assign the various permissions to the role

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`rolename` | `string` | **Required** The name of the role (must be unique)
`admin_center` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'
`storage_l2_config` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'
`storage_config` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'
`storage_charts` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'
`storage_alerts` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'
`billing_invoices` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'
`billing_usage` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'
`support_docs` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'
`support_downloads` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'
`support_cases` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'
`sfdc_info` | `string` | **Optional** One of 'no_access', 'read', or 'modify'. Defaults to 'no_access'


This method returns a new role object. For example:

    {
        "storage_charts": "no_access",
        "storage_l2_config": "no_access",
        "admin_center": "no_access",
        "support_cases": "modify",
        "billing_invoices": "no_access",
        "role_id": 7,
        "storage_config": "no_access",
        "role_name": "Fake Role",
        "billing_usage": "read",
        "storage_alerts": "no_access",
        "support_docs": "no_access",
        "support_downloads": "no_access",
        "sfdc_info": "no_access"
    }

## List mutliple roles

    GET /roles

This method can retrieve all roles from the database. Specifying search parameters can narrow the list returned.

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`name_match` | `string` | *Optional*.  Limit the return to roles that match `name_match`

Returns a list of matching roles using a name_match of ^.*Admin.*$

    [
        {
            "storage_charts": "read",
            "storage_l2_config": "modify",
            "admin_center": "modify",
            "support_cases": "modify",
            "billing_invoices": "read",
            "role_id": 1,
            "storage_config": "modify",
            "role_name": "System Admin",
            "billing_usage": "read",
            "storage_alerts": "modify",
            "support_docs": "read",
            "support_downloads": "read",
            "sfdc_info": "modify"
        },
        {
            "storage_charts": "read",
            "storage_l2_config": "read",
            "admin_center": "read",
            "support_cases": "modify",
            "billing_invoices": "no_access",
            "role_id": 2,
            "storage_config": "modify",
            "role_name": "Storage Admin",
            "billing_usage": "no_access",
            "storage_alerts": "modify",
            "support_docs": "read",
            "support_downloads": "read",
            "sfdc_info": "read"
        }
    ]
            
## Get a role

    GET /role-<role_id>

Get a role (by `role_id`)

Parameter  | Type    | Description
-------------|---------|-----------------------------------------------
`role_id` | `integer`| **Required**. The role id to get

A role object. For example:

    {
        "storage_charts": "read",
        "storage_l2_config": "read",
        "admin_center": "read",
        "support_cases": "modify",
        "billing_invoices": "read",
        "role_id": 4,
        "storage_config": "read",
        "role_name": "CSD Level 1",
        "billing_usage": "read",
        "storage_alerts": "read",
        "support_docs": "read",
        "support_downloads": "read",
        "sfdc_info": "read"
    }

## Update a role
        
    PUT /role-<role-id>

Allows changes to made to a role. Change the permissions or role name

Parameter    | Type    | Description
-------------|---------|-----------------------------------------------
`role_id` | `integer`| **Required**. The role id to get
`rolename` | `string` | **Optional** The new name for the role (must be unique)
`admin_center` | `string` | **Optional** One of 'no_access', 'read', or 'modify'
`storage_l2_config` | `string` | **Optional** One of 'no_access', 'read', or 'modify'
`storage_config` | `string` | **Optional** One of 'no_access', 'read', or 'modify'
`storage_charts` | `string` | **Optional** One of 'no_access', 'read', or 'modify'
`storage_alerts` | `string` | **Optional** One of 'no_access', 'read', or 'modify'
`billing_invoices` | `string` | **Optional** One of 'no_access', 'read', or 'modify'
`billing_usage` | `string` | **Optional** One of 'no_access', 'read', or 'modify'
`support_docs` | `string` | **Optional** One of 'no_access', 'read', or 'modify'
`support_downloads` | `string` | **Optional** One of 'no_access', 'read', or 'modify'
`support_cases` | `string` | **Optional** One of 'no_access', 'read', or 'modify'
`sfdc_info` | `string` | **Optional** One of 'no_access', 'read', or 'modify'


Returns a role object. For an example role return object see [Get a role](#get-a-role)

## Delete a role

    DELETE /role-<role_id>

Parameter | Type    | Description
-------------|---------|-----------------------------------------------
`role_id` | `integer`| **Required**. The role id to delete

Returns the role object that was deleted. For an example role return object see [Get a role](#get-a-role)

# Access Management

An access record defines a user's role within a customer. This typcially singular, meaning that a
user has only one access record associating them with their customer. However, there is the need
for some users to have access to multiple customers. Take, for example, a ClearSky support engineer
who needs to help a customer address an issue.

An access record is **required** to create a new session. 

Name | Type | Description
-----|------|-----------------------------------------------
`access_id` | `integer` | The id assigned to the access
`user_id` | `integer` | The user id for this access
`customer_id` | `integer` | The customer id assigned to the access
`role_id` | `integer` | The role id for the user at the customer

All accesses must have a unique (`user_id`, `customer_id`).

## Create an access

    POST /access-new

This method creates a new access. A user may have on a single role within the context
of the customer specified. However, a user may be assigned to any number (0 to n)
customers.

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`user_id` | `integer` | **Required** The user for the access
`customer_id`| `integer` | **Required** The customerfor the access
`role_id`| `integer` | **Required** The role for the access

    {
        "role_id": 5,
        "customer_id": 68999,
        "user_id": 1,
        "access_id": 4
    }

## List mutliple accesses

    GET /accesses

This method can retrieve all accesses from the database. Specifying search parameters can narrow the list returned.

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`customer_id` | `integer` | *Optional*. Limit the return to accesses that match the customer
`user_id` | `integer` | *Optional*.  Limit the return to accesses that match user

    [
        {
            "role_id": 5,
            "customer_id": 917504,
            "user_id": 1,
            "access_id": 1
        },
        {
            "role_id": 4,
            "customer_id": 917504,
            "user_id": 2,
            "access_id": 2
        },
        {
            "role_id": 2,
            "customer_id": 589824,
            "user_id": 1,
            "access_id": 4
        }
    ]

## Get an access

    GET /access-<access_id>

Get an access (by `access_id`)

Parameter  | Type    | Description
-------------|---------|-----------------------------------------------
`access_id` | `integer`| **Required**. The access id to get

an access object. For example:
    {
        "role_id": 5,
        "customer_id": 589824,
        "user_id": 2,
        "access_id": 2
    }

## Update an access

    PUT /access-<access-id>

Allows changes to role within an access.

Parameter    | Type    | Description
-------------|---------|-----------------------------------------------
`access_id` | `integer` | **Required** The access to update
`role_id` | `integer` | *Optional* The new role ofr the user within the customer. Not really optional since this is the only thing to change

Returns an access object. For an example access return object see [Get an access](#get-an-access)

## Delete an access

    DELETE /access-<access_id>

The default access (an access who's `domain_id` is the same as the user record's `domain_id`) may not be deleted

Parameter | Type    | Description
-------------|---------|-----------------------------------------------
`access_id` | `integer`| **Required**. The access id to delete

Returns the access object that was deleted. For an example access return object see [Get an access](#get-an-access)

# Session Management

Sessions must be authenticated. They have an expiration time set by the customer chosen.

A session has the states:
* need\_second\_factor - a verification code has been sent the user via their `email` or `sms`
* choose\_customer - a user has multiple access records. The session needs to be associated with a particular customer.
* active - there is an active session: a user has a role within the domain
* expired - the session has timed out and should be denied access to all things
* logged\_out - the session has been logged out and should be denied access to all things

Two factor authentication is controlled by the creator of a new session.

Sessions have the following attributes.

Name | Type | Description
-----|------|-----------------------------------------------
`session_id` | `integer` | The id assigned to the session
`access_id` | `integer` | The access for the session
`session_state` | One of: `need_second_factor`,`choose_customer`, `active`, `expired`, `logged_out` | The state of the session
`last_activity` | `timestamp` | The time of the last detected activity
`times_out_at` | `timestamp` | The session expiry time
`logged_out_at` | `timestamp` | The time the session was logged out
`verify_code` | `integer` | The verification code

Creating a session is the way a user logs in.

## Create a session

    POST /session-new

This method creates a new session == log in. If the user's home domain specifies two-factor authentication, then an email (or sms, when supported) is sent to the user with a randomly generated 6 digit code. That code is then used to [verify the session](#verify-a-session) .

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`user_name` | `string`| **Required**. The name of the user to verify - this can be the email or the nickname for the user
`password` | `string`| **Required**. The user's password
`need_second_factor` | `bool` | *Optional*. If true then send out a second factor 6 digit code to the user's choosen `two_factor_type`

This method returns a new session object. For example, this is a return when `need_second_factor` is true :

    {
        "session_state": "need_second_factor",
        "logged_out_at": null,
        "times_out_at": "2015-04-10 11:23:26.094399",
        "session_user_id": 1,
        "session_id": 3,
        "last_activity": "2015-04-10 11:13:26.094399",
        "verify_code": 633796,
        "access_id": null
    }

For a session that presents multiple customers, you get:
    {
        "customers": [
            {
                "mgmt_app_ip": null,
                "user_id": 1,
                "two_factor": false,
                "role_id": 5,
                "idle_timeout": 3600,
                "access_id": 1,
                "mgmt_app_port": null,
                "customer_id": 589824,
                "customer_name": "clearskydata.com"
            },
            {
                "mgmt_app_ip": null,
                "user_id": 1,
                "two_factor": true,
                "role_id": 5,
                "idle_timeout": 1234,
                "access_id": 4,
                "mgmt_app_port": null,
                "customer_id": 589824,
                "customer_name": "cars.com"
            }
        ],
        "session_state": "choose_customer",
        "logged_out_at": null,
        "times_out_at": "2015-04-10 12:51:30.120373",
        "session_user_id": 1,
        "session_id": 10,
        "last_activity": "2015-04-10 12:41:30.120373",
        "verify_code": null,
        "access_id": null
    }

The session must be verified before `times_out_at` time has been reached, otherwise the session will be marked as `expired` and the user will have to create a new session.

## Create a session using SSO

    POST /session-new-sso

This is the method to call with the token returned from [Single Sign On](#single-sign-on). The token as returned is already URL-encoded and can just be passed back as a POST parameter

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`sso_token` | `string`| **Required**. The SSO token for the signon

This method returns an existing session - exactly like an active session (see [Session Activity](#session-activity)) since this sign on is considered session activity.

    {
        "times_out_at": "2015-04-10 12:55:41.945946",
        "session_user_id": 1,
        "last_activity": "2015-04-10 12:35:07.945946",
        "customer_name": "cars.com",
        "session_state": "active",
        "user_id": 1,
        "user_state": "unverified",
        "support_cases": "modify",
        "sms_number": null,
        "role_id": 5,
        "password_expires_at": null,
        "billing_usage": "read",
        "storage_alerts": "modify",
        "customer_id": 589824,
        "email": "tuser@clearskydata.com",
        "storage_charts": "read",
        "logged_out_at": null,
        "idle_timeout": 1234,
        "role_name": "CSD Level 2",
        "support_docs": "read",
        "support_downloads": "read",
        "nickname": null,
        "verified_on": null,
        "mgmt_app_ip": null,
        "storage_l2_config": "modify",
        "admin_center": "modify",
        "two_factor_type": "email",
        "two_factor": true,
        "billing_invoices": "read",
        "storage_config": "modify",
        "session_id": 5,
        "access_id": 4,
        "mgmt_app_port": null,
        "sms_carrier_id": null,
        "sfdc_info": "read"
    }



## Verify a session

    PUT /session-<session-id>/verify

The session must be in `need_2nd_factor` state, otherwise an error is returned.

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`verify_code` | `integer`| **Required**. The verification code

The result of the verification is a session object. The state will either be set to `active` if the user only has a single role, or `choose_customer` if the user needs to [choose a customer](#choose-a-customer)

    {
        "times_out_at": "2015-04-10 13:04:08.415424",
        "session_user_id": 3,
        "last_activity": "2015-04-10 12:47:28.415424",
        "customer_name": "testing.com",
        "session_state": "active",
        "user_id": 3,
        "user_state": "verified",
        "support_cases": "modify",
        "sms_number": null,
        "role_id": 6,
        "password_expires_at": null,
        "billing_usage": "no_access",
        "storage_alerts": "no_access",
        "customer_id": 589824,
        "email": "foo@xyz.com",
        "storage_charts": "no_access",
        "logged_out_at": null,
        "idle_timeout": 1000,
        "role_name": "Qualified Prospect",
        "support_docs": "read",
        "support_downloads": "read",
        "nickname": "Foo",
        "verified_on": "2015-04-09 23:10:00.491264",
        "mgmt_app_ip": null,
        "storage_l2_config": "no_access",
        "admin_center": "no_access",
        "two_factor_type": "email",
        "two_factor": true,
        "billing_invoices": "no_access",
        "storage_config": "no_access",
        "session_id": 12,
        "access_id": 5,
        "mgmt_app_port": null,
        "sms_carrier_id": null,
        "sfdc_info": "no_access"
    }



    {
        "user_id": 1,
        "times_out_at": "2015-03-14 19:40:54.917120",
        "verify_code": 0,
        "session_id": 7,
        "last_activity": "2015-03-14 17:10:54.917120",
        "state": "active",
        "logged_out_at": null,
        "domain_id": 2
    }

## Choose a customer

    PUT /session-<session-id>/choose

The session is being placed into customer

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`customer_id` | `integer`| **Required**. The customer to choose

Returns a session object.

## Session Activity

    PUT /session-<session-id>/active

This checks to see if the session is active. If the `interactive` parameter is false, the `last_activity` and `times_out_at` for the session are not reset. If `interactive` is true or not present, then `last_activity` is set to the current time and `times_out_at` is set to the `last_activity` + the `domain_id idle_timeout`.

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`interactive` | `bool`| *Optional*. False for no update of timers

The session object is returned. All fields are fetched from the database and thereby may be updated by other users (for example, the role might have changed or a privilege in the role might morph)

    {
        "times_out_at": "2015-04-10 13:05:55.686868",
        "session_user_id": 3,
        "last_activity": "2015-04-10 12:49:15.686868",
        "customer_name": "testing.com",
        "session_state": "active",
        "user_id": 3,
        "user_state": "verified",
        "support_cases": "modify",
        "sms_number": null,
        "role_id": 6,
        "password_expires_at": null,
        "billing_usage": "no_access",
        "storage_alerts": "no_access",
        "customer_id": 589824,
        "email": "foo@xyz.com",
        "storage_charts": "no_access",
        "logged_out_at": null,
        "idle_timeout": 1000,
        "role_name": "Qualified Prospect",
        "support_docs": "read",
        "support_downloads": "read",
        "nickname": "Foo",
        "verified_on": "2015-04-09 23:10:00.491264",
        "mgmt_app_ip": null,
        "storage_l2_config": "no_access",
        "admin_center": "no_access",
        "two_factor_type": "email",
        "two_factor": true,
        "billing_invoices": "no_access",
        "storage_config": "no_access",
        "session_id": 12,
        "access_id": 5,
        "mgmt_app_port": null,
        "sms_carrier_id": null,
        "sfdc_info": "no_access"
    }



## Session Logout

    PUT /session-<session-id>/logout

Logout a session. The `logged_out_at` is set to the time of the logout and the session state is set to `logged_out`.

    {
        "session_state": "logged_out",
        "logged_out_at": "2015-04-10 12:51:41.221414",
        "times_out_at": "2015-04-10 12:55:41.945946",
        "session_user_id": 1,
        "session_id": 5,
        "last_activity": "2015-04-10 12:35:07.945946",
        "verify_code": 0,
        "access_id": 4
    }

## Single Sign On

    PUT /session-<session-id>/sso

Get an sso token to use for single sign on. The token is passed back to the authentication server doing a single sign-on. The session must be in the `active` state.

The url-encoded token is sent back to the caller. It can be large. It is the result of signed digest that is private to the authentication server. This is a "one-shot" token; Upon it's first use it is invalidated.

    {
        "sso_token": "u0Ja4PJ2DuzLUBTgxqRw171A3MXhe8O9LNav1Q7u73hu4L%2B5zdnuseAKCHgbgW5A02utt9n23OGOlZQ8LNb92XV1jFdAikcq4Vbuy%2Byk%2BIsV0RUTenNhke440gsSHnCpBSmkXRS4MdPM8wEFi16ARo9vzGhLSKEOlk017uH4KfHpiAlAw%2BNZrbabMzdbZE2XTlxPwhzHwZ1XcVt7GIiRVNkB0zWUSlmkVgoSYHYp17Lfqj8Nl0i5HvSZV0yp70J6uowY2U3ecd919gIcJB0S70Q6Mv7PGrosanXK2kELwpxQqXnWkngsBLwnaDmg17kY9HMufe4jBq8At%2BB4EDTalA%3D%3D"
    }


## List mutliple sessions

This method can retrieve all sessions from the database. Specifying search parameters can narrow the list returned. If you specify
the customer_id parameter then the output contains the role, customer and user information. All other queries return a stripped
down session record

    GET /sessions

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`customer_id` | `integer` | *Optional*. Limit the return to sessions that match customer (more expansive output)
`user_id` | `integer` | *Optional*.  Limit the return to sessions that match user
`session_state` |  One of: `need_2nd_factor`,`choose_domain`, `active`, `expired`, `logged_out` | *Optional*. The state of the session


Returns a list of matching sessions - this used a customer_id argument of 2.

    [
        {
            "times_out_at": "2015-04-10 12:05:26.093008",
            "session_user_id": 3,
            "last_activity": "2015-04-10 11:48:46.093008",
            "session_id": 4,
            "customer_name": "testing.com",
            "session_state": "expired",
            "user_id": 3,
            "user_state": "verified",
            "role_id": 6,
            "password_expires_at": null,
            "customer_id": 917504,
            "email": "foo@xyz.com",
            "logged_out_at": null,
            "idle_timeout": 1000,
            "nickname": "Foo",
            "verified_on": "2015-04-09 23:10:00.491264",
            "mgmt_app_ip": null,
            "two_factor_type": "email",
            "two_factor": true,
            "sms_number": null,
            "access_id": 5,
            "mgmt_app_port": null,
            "sms_carrier_id": null
        }
    ]

## Get a session

Get a session (by `session_id`)

    GET /session-<session_id>

Parameter  | Type    | Description
-------------|---------|-----------------------------------------------
`session_id` | `integer`| **Required**. The session id to get

A session object. For example:

    {
        "times_out_at": "2015-04-09 23:38:57.510796",
        "session_user_id": 1,
        "last_activity": "2015-04-09 22:38:57.510796",
        "customer_name": "clearskydata.com",
        "session_state": "expired",
        "user_id": 1,
        "user_state": "unverified",
        "support_cases": "modify",
        "sms_number": null,
        "role_id": 5,
        "password_expires_at": null,
        "billing_usage": "read",
        "storage_alerts": "modify",
        "customer_id": 917504,
        "email": "tuser@clearskydata.com",
        "storage_charts": "read",
        "logged_out_at": null,
        "idle_timeout": 3600,
        "role_name": "CSD Level 2",
        "support_docs": "read",
        "support_downloads": "read",
        "nickname": null,
        "verified_on": null,
        "mgmt_app_ip": null,
        "storage_l2_config": "modify",
        "admin_center": "modify",
        "two_factor_type": "email",
        "two_factor": false,
        "billing_invoices": "read",
        "storage_config": "modify",
        "session_id": 1,
        "access_id": 1,
        "mgmt_app_port": null,
        "sms_carrier_id": null,
        "sfdc_info": "read"
    }

## Delete a session

The default session (a session who's `domain_id` is the same as the user record's `domain_id`) may not be deleted

    DELETE /session-<session_id>

Parameter | Type    | Description
-------------|---------|-----------------------------------------------
`session_id` | `integer`| **Required**. The session id to delete

Returns the session object that was deleted.
    {
        "session_state": "logged_out",
        "logged_out_at": "2015-04-10 12:51:41.221414",
        "times_out_at": "2015-04-10 12:55:41.945946",
        "session_user_id": 1,
        "session_id": 5,
        "last_activity": "2015-04-10 12:35:07.945946",
        "verify_code": 0,
        "access_id": 4
    }

# Sms Carrier Management

Sms_Carriers have the following attributes.

Name | Type | Description
-----|------|-----------------------------------------------
`sms_carrier_id` | `integer` | The id assigned to the sms_carrier
`carrier_name` | `string` | The name (to display) for the carrier
`email_gateway` | `string` | The sms_carrier's email gateway

The following carriers are created by default within the database. Use [Get a SMS carrier](#get-a-sms-carrier) to find the sms\_carrier\_id.

carrier\_name | email\_gateway
----|---
AllTel | text.wireless.alltel.com
AT&T | txt.att.net
Boost Mobile | myboostmobile.com
Cricket | sms.mycricket.com
Sprint | messaging.sprintpcs.com
T-Mobile | tmomail.net
US Cellular | email.uscc.net
Verizon | vtext.com
Virgin Mobile | vmobl.com

## Create a SMS carrier

    POST /sms_carrier-new

This method creates a new SMS carrier.

Parameter         | Type    | Description
-------------|---------|---------------------------------------
`carrier_name` | `string` | **Required**. The name of the SMS carrier
`email_gateway` | `string` | **Required**. The SMS carrier's email address

This method returns a new sms_carrier object. For example:

    {
        "carrier_name": "Sleepy the Man",
        "sms_carrier_id": 11,
        "email_gateway": "zzz.com"
    }

## List mutliple SMS carriers

    GET /sms_carriers

This method can view mutliple SMS carriers from the database. The search can be controlled by search parameters

Parameter         | Type    | Description
-------------|---------|-----------------------------------------------
`carrier_name_match`| `string` | *Optional*. Posix regular expression to match against carrier name
`email_gateway_match`| `string` | *Optional*. Posix regular expression to match against email gateways

Returns a list of matching sms_carriers:

    [
        {
            "carrier_name": "AllTel",
            "sms_carrier_id": 1,
            "email_gateway": "text.wireless.alltel.com"
        },
        {
            "carrier_name": "Sprint",
            "sms_carrier_id": 5,
            "email_gateway": "messaging.sprintpcs.com"
        },
        {
            "carrier_name": "Verizon",
            "sms_carrier_id": 8,
            "email_gateway": "vtext.com"
        },
        {
            "carrier_name": "Virgin Mobile",
            "sms_carrier_id": 9,
            "email_gateway": "vmobl.com"
        },
        {
            "carrier_name": "Adam",
            "sms_carrier_id": 10,
            "email_gateway": "xyz.com"
        },
        {
            "carrier_name": "Sleepy the Man",
            "sms_carrier_id": 11,
            "email_gateway": "zzz.com"
        }
    ]

## Get a SMS carrier

    GET /sms_carrier-<sms_carrier_id>

Get a sms_carrier (by `sms_carrier_id`)

Parameter  | Type    | Description
-------------|---------|-----------------------------------------------
`sms_carrier_id` | `integer`| **Required**. The sms_carrier id to get

A sms_carrier object. For example:

    {
        "carrier_name": "Verizon",
        "sms_carrier_id": 8,
        "email_gateway": "vtext.com"
    }

## Update a sms_carrier

    PUT /sms_carrier-<sms_carrier-id>

Allows changes to made to a sms_carrier.

Parameter    | Type    | Description
-------------|---------|-----------------------------------------------
`carrier_name`| `string` | *Optional*. The carrier name
`email_gateway`| `string` | *Optional*. The carrier email gateway

Returns a sms\_carrier object. For an example sms\_carrier return object see [Get a SMS carrier](#get-a-sms_carrier)

## Delete a sms_carrier

    DELETE /sms_carrier-<sms_carrier_id>

Delete a sms_carrier (by id)

Parameter | Type    | Description
-------------|---------|-----------------------------------------------
`sms_carrier_id` | `integer`| **Required**. The sms_carrier id to delete

Returns the sms\_carrier object that was deleted. For an example sms\_carrier return object see [Get a SMS carrier](#get-a-sms_carrier)


