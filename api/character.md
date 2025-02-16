# Character

## Generic Types

```ts
type note = {
  icon: string;
  brief: string;
  full: string;
};
```

## Get `/c/`

Authenticated: None

Returns a list of all characters.

```ts
interface NotFound {
  id: string;
  name: string;
}[]
```

## Get `/c/:character-id/`

Authenticated: None

Returns data about a character.

```ts
interface NotFound {
  success: false;
  error: string;
}

interface Output {
  generic: {
    name: string;
    species: {
      base: string; // must match to the id of a species
      lineage?: string; // must match to a lineage of the species unless the species has no lineages, then set it to undefined
    };
    background: {
      id: string; // must match to the id of a background
      // options for the background
      // for example: chartalan scam type
      options: Record<string, any>;
    };
    levels: {
      id: string; // must match the id of a class
      sub?: string; // must match the id of a subclass. if the character does not have enough levels to unlock subclasses, set to undefined
      levels: number; // between 1 and number of level entries in the class
    }[];
    /*
    settings which may be per client or server software
    for example: if a client with the ID "bard" wanted to store a setting "colour" with a value of "#008282"
        "bard-client": { "colour": "#008282" }
    */
    settings: Record<`${"client" | "server"}-${string}`, Record<string, any>>;
    inspiration: boolean;
    // values generated based on character data
    // this is convenience for clients and not designed to be stored by the server
    generated: {
      initiative: {
        value: number;
        note: note
      };
      ac: {
        value: number;
        note: note
      };
      pb: {
        value: number;
        note: note
      };
      speeds: {
        walking?: number;
        flying?: number;
        hovering?: number;
        burrowing?: number;
        climbing?: number;
        swimming?: number;
        ["walking" | "flying" | "hovering" | "burrowing" | "climbing" | "swimming"]: number;
        note: note;
      };
    };
  };

  // ability scores and related info (such as skills)
  abilities: {
    abilities: Record<string, {
        score: number; // use to generate modifier
        proficientSave: boolean;
    }>;
    skills: Record<string, {
        base: string; // must match to the id of an ability
        proficient: boolean;
    }>
  };
}
```
