# Rulealize.RuleSet.Signatures

A decision that needs several signatures before it counts — a
[Rulealize](https://github.com/reny-develop/Rulealize) rule set written to be **held**.

| | |
| --- | --- |
| Rule set id | `Rulealize.RuleSet.Signatures` |
| Package | [`Rulealize.RuleSet.Signatures`](https://www.nuget.org/packages/Rulealize.RuleSet.Signatures) |
| Inputs | `sign` `decline` |
| Holds | nothing |
| Draws on | TypeSchema, State, Comparison, Logic, Binding, Branch, Definition, Arithmetic, Sequence |

```json
"uses": [
  { "ruleSet": "Rulealize.RuleSet.Signatures", "version": "^1.0", "as": "sigs" }
]
```

**`as` is not optional.** An alias defaults to the identifier and may not contain a `.`, so an
entry that leaves it out is refused, with a message about a key you did not write.

## Four eyes, and everything above it

`required` is a number the instance supplies, so two signatures and five are two state
documents and this one rule set — and one is the degenerate case, worth being able to write
without reaching for a different document.

```json
"sigs": {
  "ruleSet": "Rulealize.RuleSet.Signatures@1.0.0",
  "data": {
    "signatories": ["ann", "bo", "cy"],
    "required": 2,
    "signed": [],
    "outcome": "open"
  }
}
```

**Nobody signs twice.** The domain is the signatories who have not signed yet, so a second
signature from one person is not a move that gets refused — it is one that is never offered.

**`required` is not checked against how many people there are.** A case asking for three
signatures from two signatories never settles, which is a true thing about that case; refusing
to represent it here would be this document deciding something its holder can see.

## What is being signed is not in this document

Deliberately. This says how many said yes and who still can; what they are saying yes *to* is
the holder's.

Held beside [`Rulealize.RuleSet.Request`](https://www.nuget.org/packages/Rulealize.RuleSet.Request)
the subject is in one, the counting is in the other, and the guard that joins them is in
neither — which is the guard composition exists for:

```json
"uses": [
  { "ruleSet": "Rulealize.RuleSet.Request",    "version": "^1.0", "as": "req"  },
  { "ruleSet": "Rulealize.RuleSet.Signatures", "version": "^1.0", "as": "sigs" }
],

"held": {
  "req": {
    "grant": {
      "when": {
        "op": "cmp.eq", "right": "signed",
        "left": { "op": "rec.at", "record": "$sigs", "key": "outcome" }
      }
    }
  }
}
```

*A request may not be granted until it has been signed.* Neither document could say that; the
request half has never heard of a signatory and the signing half has no idea what it is for.

## What a holder gets

| | |
| --- | --- |
| `rec.at($sigs, "outcome")` | `open`, `signed` or `refused` |
| `rec.at($sigs, "signed")` | who has, in the order they did |
| `rec.at($sigs, "required")` | how many it takes |

`held` sees the candidate as `@who`, which is where "the person who raised it may not sign it"
is written — a rule about two documents, and one this document could not have.

## One refusal ends it

`decline` refuses on behalf of everybody rather than dropping one signatory and letting the
rest carry on. That is the stricter reading and the one worth publishing: a process where a
named approver said no and the thing happened anyway is not an approval process.

A holder that wants the softer rule hides `decline` with `"when": false` and drives its own.

## Where it fits

| The signatures are | and the holder |
| --- | --- |
| two officers on a payment | releases it |
| a reviewer and a maintainer | merges |
| both parents on a consent form | enrols |
| a quorum of a board | records the resolution |

## Trying it

Needs [Rulealize.Cli](https://github.com/reny-develop/Rulealize.Cli):
`dotnet tool install -g Rulealize.Cli`.

```sh
rulealize restore src/Rulealize.RuleSet.Signatures/ruleset/signatures.json
rulealize play    src/Rulealize.RuleSet.Signatures/ruleset/signatures.json --state state/example.json
```

Two of the three sign and it ends `signed`; any one declines and it ends `refused`. Whoever
signs stops being offered.

## Building the package

```sh
dotnet pack src/Rulealize.RuleSet.Signatures -c Release
```

A package with no `lib` folder, holding this one document under `ruleset/`.
[What each property in the project file is for](https://github.com/reny-develop/Rulealize.Registry/blob/main/doc/publish.md#a-rule-set).

## License

Apache-2.0, and it covers the document as much as everything else here.
