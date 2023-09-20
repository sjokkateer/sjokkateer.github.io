---
topic: "Json Serializer Basics"
date: "2023-09-23"
---

## [System.Text.Json.JsonSerializer](https://learn.microsoft.com/en-us/dotnet/api/system.text.json.jsonserializer?view=net-7.0)

### Context
As I was trying to develop a small CLI spinner I found several different patterns online. Since the logic for the spinner is the same regardless of the pattern it seemed nice to extract the spinner patterns to an external file that could easily be updated without having to update the code.

The spinner is a simple class with a name to identify it, and a sequence of characters that represent the pattern of the spinner.

### Experiment

The JSON file content.

```json
[
  {
    "name": "mountain",
    "characters": [ "▁", "▂", "▃", "▄", "▅", "▆", "▇", "█", "▇", "▆", "▅", "▄", "▃", "▁" ]
  },
  {
    "name": "arrow",
    "characters": [ "←", "↖", "↑", "↗", "→", "↘", "↓", "↙" ]
  },
  {
    "Name": "dash",
    "characters": [ "|", "/", "-", "\\" ]
  },
  {
    "name": "clock",
    "characters": [ "◴", "◷", "◶", "◵" ]
  },
  {
    "name": "moon",
    "characters": [ "◐", "◓", "◑", "◒" ]
  }
]
```

The corresponding class representation of such a JSON object.

```csharp
using System.Text.Json.Serialization;

public class Spinner
{
    public string Name { get; }
    public char[] Characters { get; }

    [JsonConstructor]
    public Spinner(string name, char[] characters)
    {
        Name = name;
        Characters = characters;
    }
}
```

The `Spinner.class` definition is referred to as an [immutable type](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/immutability?pivots=dotnet-7-0#immutable-types-and-records).

> NOTE: The property `name` *AND* `type` must exactly match the constructor parameter `name` *AND* `type`.

> NOTE: The JsonConstructorAttribute is not required when there is only one parameterized constructor defined.

Personally I think it is good practice to include the attribute from the start, since if an additional paramterized constructor would be added, it could break the system. The following changes highlight such a breaking change.

```csharp
public class Spinner
{
    public string Name { get; }
    public char[] Characters { get; }

    public Spinner(string name, char[] characters) : this(name)
    {
        Characters = characters;
    }

    public Spinner(string name)
    {
        Name = name;
    }
}
```

The corresponding changes.

```diff
-   [JsonConstructor]
-   public Spinner(string name, char[] characters)
+   public Spinner(string name, char[] characters) : this(name)
    {
-       Name = name;
        Characters = characters;
    }
+
+   public Spinner(string name)
+   {
+       Name = name;
+   }
```

As a consequence the following following exception would be thrown.

```shell
System.NotSupportedException: "Deserialization of types without a parameterless constructor, a singular parameterized constructor, or a parameterized constructor annotated with 'JsonConstructorAttribute' is not supported.
```

Deserializing the JSON file content is as simple as.

```csharp
    using var fs = File.Open(pathToSpinnersFile, FileMode.Open);
    var spinners = JsonSerializer.Deserialize<IEnumerable<Spinner>> (
        fs,
        new JsonSerializerOptions() { PropertyNameCaseInsensitive = true }
    );
```

Deserializing without the `PropertyNameCaseInsensitive = true` would result in unmapped values since the serializer seems to use `nameof(Spinner.Name)` and `nameof(Spinner.Sequence)` to map to the property names in the JSON document.

Another way to map between the class' property names and the JSON properties is to add the  [JsonPropertyNameAttribute](https://learn.microsoft.com/en-us/dotnet/api/system.text.json.serialization.jsonpropertynameattribute?view=net-7.0). If you work with many/large JSON documents, specifying this for most if not all properties becomes tedious quick and it might be better to create your own [JsonNamingPolicy](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/customize-properties?pivots=dotnet-7-0#use-a-custom-json-property-naming-policy).
