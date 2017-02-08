
# initialize-essay [![CircleCI](https://circleci.com/gh/dtinth/initialize-essay/tree/master.svg?style=svg)](https://circleci.com/gh/dtinth/initialize-essay/tree/master) [![codecov](https://codecov.io/gh/dtinth/initialize-essay/branch/master/graph/badge.svg)](https://codecov.io/gh/dtinth/initialize-essay)

The easiest way to initialize a new essay.

Installation:

1. Install Yarn

2. `yarn global add initialize-essay`

Usage:

1. `mkdir my-essay && cd my-essay`

2. `yarn init`

3. `initialize-essay`


```js
// showMeHowToUseIt.js
// (This file is used in the self-test)
export default function showMeHowToUseIt (run) {
  // Create your project directory
  run('mkdir /tmp/test-initialize-essay')

  // cd into it
  process.chdir('/tmp/test-initialize-essay')

  // Initialize an npm project
  run('yarn init -y')

  // Initialize an essay
  run('initialize-essay')

  // Test the essay
  run('yarn test')

  // Build the essay
  run('yarn run prepublish')
  assert(
    require('fs')
      .existsSync('/tmp/test-initialize-essay/lib/index.js')
  )

  // How about initialize it again?
  run('initialize-essay')
}
```

## Initialization steps

```js
// index.js
import fs from 'fs'
import chalk from 'chalk'
import log from './log'
import addScripts from './addScripts'
import addFiles from './addFiles'
import modifyPackageJson from './modifyPackageJson'

export function initialize () {
  log('Initializing an essay project...')

  // README.md
  if (!fs.existsSync('README.md')) {
    log('Writing an example README.md file')
    fs.writeFileSync('README.md', fs.readFileSync(require.resolve('../example.md')))
  }

  // .gitignore
  if (!fs.existsSync('.gitignore')) {
    log('Writing a .gitignore file')
    fs.writeFileSync('.gitignore', fs.readFileSync(require.resolve('../.gitignore')))
  }

  // Update package.json
  const pkg = modifyPackageJson(pkg => {
    addScripts(pkg)
    addFiles(pkg)
  })

  // Install essay
  if (!pkg.devDependencies || !pkg.devDependencies.essay) {
    const command = 'yarn add --dev essay'
    log('Installing essay using "%s"...', command)
    require('child_process').execSync(command, { stdio: 'inherit' })
  }

  // Show success message
  console.log('')
  console.log('   Your essay has been initialized.')
  console.log('')
  console.log('   To run tests: ', chalk.cyan('yarn test'))
  console.log('   To lint code: ', chalk.cyan('yarn run lint'))
  console.log('   To build:     ', chalk.cyan('yarn run prepublish'))
  console.log('')
  console.log('   Enjoy!! :)')
  console.log('')
}
```

### Adding scripts to `package.json` file

```js
// addScripts.js
import log from './log'

export default function addScripts (pkg) {
  if (!pkg.scripts) {
    log('Adding "scripts": { } to package.json')
    pkg.scripts = { }
  }
  const scripts = {
    prepublish: 'essay build',
    test: 'essay test',
    lint: 'essay lint'
  }
  for (const scriptName of Object.keys(scripts)) {
    if (!pkg.scripts[scriptName]) {
      const script = scripts[scriptName]
      log(`Adding script "${scriptName}": "${script}" to package.json`)
      pkg.scripts[scriptName] = script
    }
  }
}
```

### Adding files to `package.json` file

```js
// addFiles.js
import log from './log'

export default function addFiles (pkg) {
  if (!pkg.files) {
    log('Adding "files": [ ] to package.json')
    pkg.files = [ ]
  }
  if (!pkg.files.includes('lib')) {
    log('Adding "lib" to "files" in package.json')
    pkg.files.push('lib')
  }
  if (!pkg.files.includes('src')) {
    log('Adding "src" to "files" in package.json')
    pkg.files.push('src')
  }
}
```

## Utils

### modifying `package.json`

```js
// modifyPackageJson.js
import readPkg from 'read-pkg'
import writePkg from 'write-pkg'
import log from './log'

export default function modifyPackageJson (f) {
  const originalPkg = readPkg.sync({ normalize: false })
  const pkg = JSON.parse(JSON.stringify(originalPkg))
  f(pkg)
  if (JSON.stringify(pkg) !== JSON.stringify(originalPkg)) {
    log('Writing package.json file...')
    writePkg.sync(pkg)
  } else {
    log('package.json unchanged')
  }
  return pkg
}
```

### logging

```js
// log.js
export default function log (...stuff) {
  console.log(' *', require('util').format(...stuff))
}
```

## Self-test

```js
// self.test.js
import fs from 'fs'
import { execSync } from 'child_process'
import chalk from 'chalk'
import { initialize } from './'
import showMeHowToUseIt from './showMeHowToUseIt'

const run = (command) => {
  console.log('=========>', chalk.bold(command))
  if (command === 'initialize-essay') {
    initialize()
  } else {
    execSync(command, { stdio: 'inherit' })
  }
}

it('Works', function () {
  this.timeout(0)

  // Build itself
  run('yarn run prepublish')

  // Cleanup
  run('rm -rf /tmp/test-initialize-essay')

  // Show me!
  const cwd = process.cwd()
  try {
    showMeHowToUseIt(run)
  } finally {
    process.chdir(cwd)
  }
})
```
