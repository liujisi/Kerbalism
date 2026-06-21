# zzz_LocalPatches

User-owned ModuleManager compatibility fixes belong in `Patches/`. Prefer one clearly named `.cfg` per issue or affected mod and include dependency guards.

Suggested shape:

```cfg
// Work around <specific observed symptom> when ExampleMod is installed.
@PART[exactPartName]:NEEDS[ExampleMod]:AFTER[ExampleMod]
{
  @MODULE[ExactModuleName]
  {
    @fieldName = correctedValue
  }
}
```

Replace every placeholder and verify names against installed configuration. Avoid `:FINAL` unless a targeted pass cannot provide stable ordering.

