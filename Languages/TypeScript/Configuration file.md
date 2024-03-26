In this article, we will explain the base configuration file to use TypeScript in our projects.

First we must create a file in the root of our project with the name ```tsconfig.json```

To create a base configuration, we will create an object with the following shape:

```javascript
{
	"compilerOptions": {
		"allowJs": true,
		"checkJs": true,
		"moduleResolution": "nodenext",
		"noEmit": true,
		"outDir": "build",
		"target": "ESNext"
	},
	"include": ["src/**/*"]
}
```

#### [compilerOptions](https://www.typescriptlang.org/tsconfig#compilerOptions)

**[allowJs](https://www.typescriptlang.org/tsconfig#allowJs)**
Allow JavaScript files to be imported inside our project, instead of just .ts and .tsx files

**[checkJs](https://www.typescriptlang.org/tsconfig#checkJs)**
Works in tandem with allowJs. When checkJs is enabled then errors are reported in JavaScript files. This is the equivalent of including ```// @ts-check``` at the top of all JavaScript files which are included in your project.

**[moduleResolution](https://www.typescriptlang.org/tsconfig#moduleResolution)**
Specify the module resolution strategy.

- ```'bundler'``` for use with bundlers. Like ```node16``` and ```nodenext```, this mode supports package.json ```"imports"``` and ```"exports"```, but unlike the Node.js resolution modes, ```bundler``` never requires file extensions on relative paths in imports.
bundler does not support resolution of ```require``` calls. In TypeScript files, this means the ```import mod = require("foo")``` syntax is forbidden; in JavaScript files, ```require``` calls are not errors but only ever return the type ```any``` (or whatever an ambient declaration of a global require function is declared to ```return```).
- ```'classic'``` was used in TypeScript before the release of 1.6. ```classic``` should not be used.
- ```'node10'``` (previously called ```'node'```) for Node.js versions older than v10, which only support CommonJS ```require```. You probably won’t need to use ```node10``` in modern code.
- ```'node16'``` or ```'nodenext'``` for modern versions of Node.js. Node.js v12 and later supports both ECMAScript imports and CommonJS ```require```, which resolve using different algorithms. These moduleResolution values, when combined with the corresponding module values, picks the right algorithm for each resolution based on whether Node.js will see an ```import``` or ```require``` in the output JavaScript code.

**[noEmit](https://www.typescriptlang.org/tsconfig#noEmit)**
Do not emit compiler output files like JavaScript source code, source-maps or declarations.

**[outDir](https://www.typescriptlang.org/tsconfig#outDir)**
If specified, ```.js``` (as well as ```.d.ts```, ```.js.map```, etc.) files will be emitted into this directory. The directory structure of the original source files is preserved.


**[target](https://www.typescriptlang.org/tsconfig#target)**
Modern browsers support all ES6 features, so ```ES6``` is a good choice. You might choose to set a lower target if your code is deployed to older environments, or a higher target if your code is guaranteed to run in newer environments.

The ```target``` setting changes which JS features are downleveled and which are left intact. For example, an arrow function ```() => this``` will be turned into an equivalent ```function``` expression if ```target``` is ES5 or lower.

The special ```ESNext``` value refers to the highest version your version of TypeScript supports. This setting should be used with caution, since it doesn’t mean the same thing between different TypeScript versions and can make upgrades less predictable.

#### [include](https://www.typescriptlang.org/tsconfig#include)
Specifies an array of filenames or patterns to include in the program. These filenames are resolved relative to the directory containing the ```tsconfig.json``` file.