# Data Adapter API

## read/write files to/from JS Objects

### Load data

```ts
abstract class Data {
  constructor(id: string) {}
  abstract get id(): string;

  abstract async read(): {
    state: "clean" | "dirty" | undefined;
    data: Object;
  };
  abstract async readRaw(): Blob;

  abstract write(data: Object):
    | {
        success: boolean;
      }
    | {
        success: false;
        error: unknown;
      };
  abstract writeRaw(data: string | Buffer):
    | {
        success: boolean;
      }
    | {
        success: false;
        error: unknown;
      };
}
```

Load data at the id. The ID is unique to the data and often structured as a file path. This is used to access the data.  
You can access the ID with `Data.id`

> [!NOTE]  
> For the purpose of checking if an ID is unique, the following should be performed:
>
> 1. remove all chars except from `a-z`, `A-Z`, `0-9`, `.`, `\`, & `/`
> 2. turn all slashes to `/`
> 3. replace sequences of `/` with a single `/`
> 4. discard `/`s at the start and end

See the definitions of the methods further in the document.

#### List IDs

```ts
function list(asFile: false): string[];
function list(asFile: true): File[];
function list(asFile: boolean): (string | File)[];
```

Get a list of all IDs

### Read

#### Read Object

```ts
async function Data.read(): {
  state: "clean" | "dirty" | undefined;
  data: Object;
};
```

Loads data using the `id` passed to the function. This is unique to the data and is created by a `write` call. (This will often be structured as a file path)  
If `state` is `clean` that means the data was last written using `write`. If `state` is `dirty` data was last written using `writeRaw`. If `state` is `undefined`, then it is not known what the data was last written with

#### Read Raw

```ts
async function Data.readRaw(): Blob;
```

Read the data at the location as is. The data will typically be a string or raw binary, however this could be different if stored in raw memory.

> [!WARNING]  
> It is not recommended to use this data unless debugging issues or if the data was written using `writeRaw`

### Write

#### Write Object

```ts
function Data.write(data: Object):
  | {
      success: boolean;
    }
  | {
      success: false;
      error: unknown;
    };
```

Writes data to a location determined by ID. Data should always be retrievable using the same ID and not another ID. Data is passed as a JS object to ease use of multiple formats. T if the adapter supports that.

#### Write Raw

```ts
function Data.writeRaw(data: string | Buffer):
  | {
      success: boolean;
    }
  | {
      success: false;
      error: unknown;
    };
```

Write the data to the location as is. The data must be a string or raw binary. This should mark data as `dirty`.

> [!CAUTION]  
> This can cause bugs for data loaded with `read`. Be careful using this function.

## Account Management

### Create account

```ts
function createAccount(
  username: string,
  password: string, // SHA256 not plaintext
  level: "visitor" | "player" | "master" | "admin"
):
  | {
      success: boolean;
    }
  | {
      success: false;
      error: unknown;
    };
```

Generate and store a user account. No email etc required. This will be created by admins using a CLI or portal.

### Edit account

```ts
function editAccount(
  username: string,
  edit: {
    password?: string; // SHA256 not plaintext
    level?: "guest" | "player" | "master" | "admin";
  }
);
```

Edit the password or level of a user account. Used by admins via a CLI or portal.  
To change username create a new account and manually transfer ownership.

> [!CAUTION]  
> The user will be signed out across all devices if the password is changed

### Delete account

```ts
function deleteAccount(username: string):
  | {
      success: true;
    }
  | {
      success: false;
      error: unknown;
    };
```

Remove an account and invalidate all auth keys for the username. Data is not deleted and must be moved or deleted by an admin.

> [!WARNING]  
> There is **_no_** confirmation for this function

### Create auth key

```ts
function createKey(
  username: string,
  password: string, // SHA256 not plaintext
  client: {
    id: string;
    version: string;
  },
  features: [] | string[],
  // timeout after n days, 30 by default
  // if the auth expires a user needs to log in again
  timeout: number = 30
):
  | {
      success: true;
      key: string;
    }
  | {
      success: false;
      error: unknown;
    };
```

Auth keys can be used to access an account without username or password and should be used to store access or generate auth tokens (for bots).

1. Check if the password matches the username and is valid.
2. Generate a new unused ID (don't use an ID which is now invalid either)
3. Store the username, client info, features, default timeout, last login, and key
4. Return the key

### Check auth key

```ts
function checkKey(key: string):
  | {
      valid: true;
      username: string;
      client: {
        id: string;
        version: string;
      };
      features: [] | string[];
      level: "visitor" | "player" | "master" | "admin";
    }
  | {
      valid: false;
    };
```

Get data associated with the key and its account. Return false if the key is not in use. If the key was last accessed longer ago than the default timeout allows, remove the entry and return false, otherwise update the last access date.
