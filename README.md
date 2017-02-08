
# initialize-essay

The easiest way to initialize a new essay.

Installation:

1. Install Yarn

2. `yarn global add initialize-essay`

Usage:

1. `mkdir my-essay && cd my-essay`

2. `yarn init`

3. `initialize-essay`


## Initialization steps

```js
// index.js
import readPkg from 'read-pkg'
import writePkg from 'write-pkg'
import fs from 'fs'
import chalk from 'chalk'

import log from './log'
import addScripts from './addScripts'
import addFiles from './addFiles'

export function initialize () {
  log('Initializing an essay project...')
  if (!fs.existsSync('README.md')) {
    log('Writing an example README.md file')
    fs.writeFileSync('README.md', fs.readFileSync(require.resolve('../example.md')))
  }
  if (!fs.existsSync('.gitignore')) {
    log('Writing a .gitignore file')
    fs.writeFileSync('.gitignore', fs.readFileSync(require.resolve('../.gitignore')))
  }
  const originalPkg = readPkg.sync({ normalize: false })
  const pkg = JSON.parse(JSON.stringify(originalPkg))
  addScripts(pkg)
  addFiles(pkg)
  if (JSON.stringify(pkg) !== JSON.stringify(originalPkg)) {
    log('Writing package.json file...')
    writePkg.sync(pkg)
  }
  if (!pkg.devDependencies || !pkg.devDependencies.essay) {
    const command = 'yarn add --dev essay'
    log('Installing essay using "%s"...', command)
    require('child_process').execSync(command, { stdio: 'inherit' })
  }
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

const run = (command) => {
  console.log('=========>', chalk.bold(command))
  execSync(command, { stdio: 'inherit' })
}

it('Works', function () {
  this.timeout(0)

  // Build itself
  run('yarn run prepublish')

  // Create a test directory
  run('rm -rf /tmp/test-initialize-essay')
  run('mkdir /tmp/test-initialize-essay')

  // Initialize an npm project
  run('cd /tmp/test-initialize-essay && yarn init -y')

  // Initialize an essay
  run('cd /tmp/test-initialize-essay && ' + require.resolve('../cli'))

  // Test the essay
  run('cd /tmp/test-initialize-essay && yarn test')

  // Build the essay
  run('cd /tmp/test-initialize-essay && yarn run prepublish')
  assert(fs.existsSync('/tmp/test-initialize-essay/lib/index.js'))
})
```
