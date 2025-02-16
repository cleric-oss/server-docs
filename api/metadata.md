# Metadata

The server metadata is used to provide human readable non-unique identification of a server and provide compatiability information.

## `/metadata.json` | `/metadata`

Authenticated: No account required

Returns the server metadata.

```ts
interface output {
    name: string;
    desc: string;
    icon_url: string; 
    version: string;
    // ex: ["colour-scheme", "widget"]
    features: string[];
    compatibility: {
        // a list of known working clients
        // hard coded or generated from clientCompatible checks
        knownClients: string[];
        /*
        informs the client as to if it is supported by the server
            true: the client is fully supported by the server
            false: the client is not supported in any way and should not attempt to connect 
            "partial": the client is partially supported by the server but does not have the full range of functionality
            undefined: the server cannot or will not determine if the client is compatible

        if a client attempts to connect to a server with any compatibility other than true, the user should be warned that they may encounter issues.

        to determine compatibility:
        - look at a known list of working and broken clients
        - use the features on the server and client
            if a server feature is required to function and is not provided on the client, the client should not be supported
            if a server feature is an enhancement (ie: colour schemes) and is missing from the client, it has partial support
        */
        clientCompatible: boolean | "partial" | undefined;
    }
    /*
    custom metadata per feature
    ie: feature version, default values
    example:
    a feature which allows stylee information to be stored on a character sheet (like accent or background colour)
    that could store the default colour scheme for the server in the custom metadata

    {
        "colour-palettes": {
            "default": {
                "accent": "#008282",
                "background": "#ffffff"
            }
        }
    }
    */
    custom: Record<string, Record<string, any>>
}
```
