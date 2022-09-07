# Solidity AST Types

[![Docs](https://img.shields.io/badge/docs-%F0%9F%93%84-blue)][docs]
[![NPM Package](https://img.shields.io/npm/v/solidity-ast.svg)](https://www.npmjs.org/package/solidity-ast)

**TypeScript types and a JSON Schema for the Solidity AST.**

```
npm install solidity-ast
```


```typescript
import type { SourceUnit, ContractDefinition } from 'solidity-ast';
```

The types included in the NPM package are automatically generated from the JSON
Schema, so you will not find them in the repository. You can see what they look
like on [unpkg] or the [documentation][docs].

[unpkg]: https://unpkg.com/solidity-ast@latest/types.d.ts
[docs]: https://solidity-ast.netlify.app/

## Solidity Versioning

The types are currently accurate and tested for Solidity >=0.6.6, but you can
very likely use them safely for any version since 0.6.0. For simple traversals
they will probably work well for 0.5.0 and up as well.

The versioning story will be gradually improved upon and the ultimate goal is
to be able to manipulate and traverse the AST in a uniform way that is as
agnostic to the Solidity version as possible.

## Utilities

Included in the package is a set of utility function for type-safe interactions
with nodes based on the node type.

### `isNodeType(nodeType, node)`

A type predicate that can be used for narrowing the type of an
unknown node, or combined with higher order functions like `filter`.

An array of node types can be used as well to check if the node matches one of them.

```typescript
import { isNodeType } from 'solidity-ast/utils';

if (isNodeType('ContractDefinition', node)) {
  // node: ContractDefinition
}

const contractDefs = sourceUnit.nodes.filter(isNodeType('ContractDefinition'));
  // contractDefs: ContractDefinition[]
```

### `findAll(nodeType, node[, prune])`

`findAll` is a generator function that will recursively enumerate all
descendent nodes of a given node type. It does this in an efficient way by
visiting only the nodes that are necessary for the searched node type.

```typescript
import { findAll } from 'solidity-ast/utils';

for (const functionDef of findAll('FunctionDefinition', sourceUnit)) {
  // functionDef: FunctionDefinition
}
```

If the optional `prune: (node: Node) => boolean` argument is specified,
`findAll` will apply the function to each node, if the return value is truthy
the node will be ignored, neither yielding the node nor recursing into it. Note
that `prune` is not available when curried.

To enumerate multiple node types at the same time, `nodeType` can be an array
of node types such as `['EnumDefinition', 'StructDefinition']`.

```typescript
for (const typeDef of findAll(['EnumDefinition', 'StructDefinition'], sourceUnit)) {
  // typeDef: EnumDefinition | StructDefinition
}
```

### `astDereferencer(solcOutput) => (nodeType, id) => Node`

`astDereferencer` looks up AST nodes based on their id. Notably, it works
across multiple source files, which is why it needs the entire solc JSON output
with the ASTs for all source files in a compilation.

> On Hardhat, the solc JSON output can be found in [build info files].

[build info files]: https://hardhat.org/guides/compile-contracts.html#build-info-files

```typescript
const deref = astDereferencer(solcOutput);

deref('ContractDefinition', 4);

for (const contractDef of findAll('ContractDefinition', sourceUnit)) {
  const baseContracts = contractDef.linearizedBaseContracts.map(deref('ContractDefinition'));
  ...
}
```
