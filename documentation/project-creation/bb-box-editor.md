# BB Box Editor Project Creation

## Build decisions

For this project, I have decided to go with Vite as my build/compilation tool.

This is mainly due to the its fast development experience, which will be necessary for manually A/B-ing the small tweaks needed when creation an interactive UI.

Ref: https://medium.com/@harishanker/webpack-vs-babel-vs-vite-understanding-modern-javascript-build-tools-4556517cecc0

## Compile Time 

Compile time refers to the phase when the TypeScript compiler processes your code, checking for type correctness, resolving dependencies, and transforming it into JavaScript.

Many decisions from the following article have been adhered to improve compile time: https://levelup.gitconnected.com/how-to-improve-typescript-compilation-and-build-times-with-tsconfig-json-optimizations-f2fa0694089b

### Compilation Improvements Made

I will list below the compilation improvements I have made, in line with the numbered suggestions in the article above:

#### 1. Enable Incremental Compilation 
`incremental: true` allows incremental data to be stored in the `tsBuildInfoFile`. This drastically reduces build times by avoiding unnecessary reprocessing of unchanged files, resulting in a more efficient development workflow.

#### 4. Exclude Unnecessary Files
`exclude: []` can exclude files/folders that aren't necessary to check with TS. I have included a few, with `"node_modules"`, `"dist"`, `"build"` and `"coverage"` being my starting examples. 

### Compilation Improvements for later

#### 2. Use `composite` for Project References.
`composite: true` is for larger projects with multiple `ts.config`'s
It generates a `.tsbuildinfo` file for each project, which gets rid of any duplicate/unnecessary re-compilation.

#### 7. Use `strict` Mode Selectively.
`strict: true` allows all type checking to take place. This is included and I personally think this is preferable. If things start to lag, you can use fine-tune what you want to be strict by making this `false` and configuring specific strict-ness. E.g: `noImplicitAny: true` or `strictNullChecks: true`.

### How to Diagnose and Measure Compile Time? 🕵️

If at any point, you feel as though compilation is taking too long and would like to diagnose certain aspects of it. It may be useful to revisit the section with the above name in the article.

For now, I have just enabled `"extendedDiagnostics": true,`, which will output information for compile debugging.